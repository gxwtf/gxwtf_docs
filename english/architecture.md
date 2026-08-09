# 技术架构

## 技术栈

| 层 | 技术 |
| --- | --- |
| 框架 | Next.js（App Router）+ React 19 |
| 语言 | TypeScript |
| 样式 | Tailwind CSS + shadcn/ui + lucide-react |
| 数据层 | Prisma ORM + PostgreSQL |
| 业务逻辑 | Next.js Server Actions（不使用传统 REST API） |
| AI | OpenAI 兼容接口（自定义封装 + 请求队列） |
| OCR | PaddleOCR（Python 微服务，端口 39821） |
| 词典 | 内置 CSV 词典（`dict/ecdict.csv`），内存查询 |
| 认证 | 广学账号 SSO + iron-session |
| PDF | jsPDF + html2canvas |
| 包管理 | pnpm |

## 目录结构

```
gxwtf_english/
├── src/
│   ├── app/                  # Next.js App Router 页面与路由
│   │   ├── page.tsx          # 单词本首页 (/)
│   │   ├── practice/         # 题目列表 + 答题页 (/practice, /practice/[questionId])
│   │   ├── writing/          # 作文积累 (/writing)
│   │   ├── settings/         # 设置 (/settings)
│   │   ├── api/              # 少量 API 路由（auth/login、writing-entries）
│   │   └── sso/callback/     # SSO 回调路由
│   ├── actions/              # Server Actions（业务逻辑）
│   │   ├── ai-question/      # AI 出题（7 题型 + 批改 + 重试 + PDF）
│   │   ├── auth.ts           # 会话校验 / 登出 / 用户同步
│   │   ├── words.ts          # 单词 CRUD / 标签配置
│   │   ├── writing-entries.ts# 作文积累 CRUD / AI 抽词
│   │   ├── review.ts         # 遗忘曲线复习状态
│   │   ├── query.ts          # 词典查询
│   │   └── image-recognition.ts # 拍照识词
│   ├── components/           # React UI 组件
│   │   ├── ui/               # shadcn/ui 基础组件
│   │   ├── AuthenticatedPage.tsx     # 单词本主页
│   │   ├── AIQuestionTypeSelector.tsx# AI 出题对话框
│   │   ├── *Answer.tsx       # 各题型作答组件
│   │   └── ...
│   ├── lib/                  # 核心库
│   │   ├── openai.ts         # AI 接口封装（重试 / 降级）
│   │   ├── ai-queue.ts       # AI 请求并发队列（默认 3）
│   │   ├── ocr.ts            # PaddleOCR 通信 + 高亮检测
│   │   ├── db.ts             # Prisma 客户端
│   │   ├── iron.ts           # iron-session 配置
│   │   ├── word-selection.ts # 选词逻辑（加权抽样）
│   │   ├── spaced-repetition/# 遗忘曲线（sm2/weights/quality/selector）
│   │   ├── dict/             # 词典查询（ecdict.csv）
│   │   ├── pdf-generator.ts  # PDF 导出
│   │   └── word-search.ts    # 模糊搜索
│   ├── hooks/                # React Hooks（useAuth 等）
│   └── types/                # TypeScript 类型定义
├── prisma/                   # schema.prisma + migrations
├── scripts/                  # dev.mjs（同时启动 OCR + Next.js）等
├── paddleocr-service/        # PaddleOCR Python 微服务
├── dict/ecdict.csv           # 词典数据
└── docs/                     # 文档站（docsify）
```

## 关键模块

### AI 集成

- `src/lib/openai.ts`：OpenAI 兼容接口封装，支持多模型配置、自动重试、超时、降级模型。模型配置由环境变量 `LLM_CONFIGS`（JSON 数组）注入。
- `src/lib/ai-queue.ts`：`AIRequestQueue` 限制并发（默认 3），防止触发模型限流。
- `src/actions/ai-question/`：7 种题型的生成、入队、批改、重试逻辑。

### 词典系统

- 数据源：`dict/ecdict.csv`，由 `src/lib/dict/ecdict.ts` 的 `DictCsv` 类加载到内存 Map。
- 手动解析 CSV（处理引号与转义字符），按换行符拆分释义并匹配词性正则（`n.`、`adj.` 等）。
- `src/actions/query.ts` 提供查词入口，返回结构化的 `Meaning[]`。

### OCR（拍照识词）

- `src/lib/ocr.ts` 与本地 PaddleOCR 微服务通信（`http://127.0.0.1:39821`），发送 base64 图片，接收识别单词、边界框、置信度。
- 流程：透视变换 → 光照归一化 → OCR 识别 → HSV 颜色检测高亮词。
- `scripts/dev.mjs` 在 `pnpm dev` 时自动启动该服务（首次会下载模型，最长等待 180 秒）。

### 单词选择

`src/lib/word-selection.ts` 的 `selectWordsForQuestion` 从用户选中的词池抽取真正进入题目的单词，支持加权抽样（详见[《遗忘曲线》](/english/forgetting-curve.md)）与关联词依赖闭包。

### 认证

广学账号 SSO + iron-session，HTTP-only Cookie（`gxwtf_auth`），有效期 7 天。详见[《登录与单点登录》](/english/auth.md)。

## 启动与部署

### 本地开发

```bash
pnpm install
pnpm dev      # 同时启动 PaddleOCR（39821）+ Next.js（3003）
```

`scripts/dev.mjs` 会先启动 PaddleOCR 并等待健康检查通过，再启动 Next.js。任一进程退出都会一起终止。

### 生产部署

```bash
pnpm build
pnpm start    # next start -p 3003
```

服务器 systemd 服务名为 `gxenglish.service`。

## 环境变量

见 `.env.example`，关键项：

| 变量 | 用途 |
| --- | --- |
| `DATABASE_URL` | PostgreSQL 连接串 |
| `GXACCOUNT_URL` | 广学账号 SSO 地址 |
| `SECRET_COOKIE_PASSWORD` | iron-session 加密密码（≥32 字符） |
| `SECRET_API_KEY` | 作文积累查询 API 认证密钥 |
| `LLM_CONFIGS` | AI 模型配置（JSON 数组） |

> 生产环境必须更换所有密钥，使用 HTTPS，且不要把 `.env` 上传到公开仓库。
