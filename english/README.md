# 广学英语

本文档针对广学英语项目（仓库本目录），即基于 Next.js 的英语词汇学习与 AI 出题平台。

## 项目介绍

广学英语是一个面向中学生的英语词汇学习平台，核心目标是「把生词本变成会出题的私教」。用户可以维护自己的生词本，借助内置词典自动补全释义，通过拍照一键识别纸质资料上的高亮生词，再由 AI 生成多种题型的练习题，并基于遗忘曲线与错误次数加权进行科学复习调度。

平台同时提供作文积累本（独立模块，不在本文档范围内），单词本与作文积累本共用同一套标签体系。

## 核心能力

- **生词本管理**：增删改查、批量操作、关联词、标签筛选、模糊搜索、PDF 导出
- **拍照识词**：上传纸质资料图片，自动识别被高亮标记的生词并批量入库
- **AI 出题**：7 种题型（选词填空、翻译句子、英译中、英英释义、词义填空、选词翻译句子、单词卡片）
- **遗忘曲线**：SM-2 间隔重复算法 + 错误次数平方加权抽词
- **题目队列**：题目生成 / 作答 / 批改全流程，支持重试与 PDF 导出
- **广学账号单点登录**：接入广学账号 SSO，一次登录全站通行

## 项目架构

- 前端：React 19 + Next.js（App Router）+ TypeScript + Tailwind CSS + shadcn/ui + lucide-react
- 后端：Next.js Server Actions（不使用传统 REST API）+ Prisma ORM
- 数据库：PostgreSQL
- AI：OpenAI 兼容接口（自带请求队列与重试）
- OCR：PaddleOCR（Python 微服务，本地端口 39821）
- 词典：内置 CSV 词典（`dict/ecdict.csv`），内存查询
- 认证：广学账号 SSO + iron-session（HTTP-only Cookie，7 天有效期）

> 更详细的技术架构、目录结构、登录流程、数据库表结构请见左侧导航的对应章节。

## 快速开始

1. 安装依赖：`pnpm install`
2. 配置环境变量：参考 `.env.example`，至少需要 `DATABASE_URL`、`GXACCOUNT_URL`、`SECRET_COOKIE_PASSWORD`、`LLM_CONFIGS`
3. 初始化数据库：`pnpm exec prisma migrate dev`
4. 启动开发服务器：`pnpm dev`（会同时启动 PaddleOCR 服务和 Next.js，默认端口 3003）
5. 访问 `http://localhost:3003`

> 生产部署使用 `pnpm build` + `pnpm start`，服务器 systemd 服务名为 `gxenglish.service`。

## 相关链接

- 广学账号 SSO：[account.gxwtf.cn](https://account.gxwtf.cn)
- 广学古诗文文档：[广学古诗文](/poem/)
- 广学账号文档：[广学账号](/account/)
