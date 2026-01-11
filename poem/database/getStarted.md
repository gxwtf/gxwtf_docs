# 数据库开发文档

## 概述

本项目使用 PostgreSQL 数据库和 Prisma ORM 进行数据管理，包含古诗、作者和读书课文章三个主要数据模型。

## 环境配置

先安装 ``prostgreSQL``，然后拷贝一份 ``.env.example`` 到 ``.env``。

### .env 文件配置

```bash
# 数据库连接
DATABASE_URL="postgresql://username:password@localhost:5432/gxwtf_poem"
```

## 部署和启动

### 1. 环境准备
```bash
# 安装依赖
npm install

# 生成 Prisma 客户端
npx prisma generate
```

### 2. 数据库初始化
```bash
# 创建数据库gxwtf_poem（如果不存在）
# 需要先确保 PostgreSQL 服务运行

# 执行数据库迁移
npx prisma migrate dev
# 或者使用 db push（开发环境）
npx prisma db push

# 导入种子数据
npx prisma db seed
```

### 3. 启动项目
```bash
# 开发模式
npm run dev

# 生产构建
npm run build
npm start
```

## 数据操作

### 数据生成脚本
项目包含多个数据生成脚本：
- `scripts/generate-poems.tsx`: 生成古诗数据
- `scripts/generate-authors.tsx`: 生成作者数据  
- `scripts/generate-articles.tsx`: 生成文章数据
- `scripts/generate-overview.tsx`: 生成概览数据
- `scripts/generate-preview.tsx`: 生成预览数据

运行所有生成脚本：

```bash
npm run generate:all
```

### Prisma Studio
启动 Prisma 数据库管理界面：
```bash
npx prisma studio
```

## 目录结构

```
prisma/
├── migrations/          # 数据库迁移文件
│   ├── 20250829145515_add_poem_model/
│   ├── 20250831110500_add_author_model/
│   ├── 20250831111215_add_article_model/
│   ├── 20250831113834_add_article_content/
│   └── migration_lock.toml
├── schema.prisma       # Prisma 数据模型定义
└── seed.ts            # 种子数据脚本

src/generated/         # Prisma 生成的客户端代码
```

## 调试和故障排除

### 常见问题

1. **数据库连接失败**
   - 检查 PostgreSQL 服务是否运行
   - 验证 DATABASE_URL 配置是否正确
   - 检查数据库用户权限

2. **迁移失败**
   - 检查迁移文件冲突
   - 使用 `npx prisma migrate reset` 重置

3. **种子数据问题**
   - 检查 seed.ts 文件语法
   - 确认数据模型匹配

### 数据备份和恢复
```bash
# 备份数据库
pg_dump gxwtf_poem > backup.sql

# 恢复数据库
psql gxwtf_poem < backup.sql
```

## 开发注意事项

1. **模式变更**：修改 schema.prisma 后必须执行迁移
2. **环境变量**：确保 .env 文件正确配置
3. **数据类型**：注意数组字段使用 String[] 类型
4. **唯一约束**：title 和 name 字段有唯一约束
5. **可选字段**：使用 ? 标记的字段可以为空

## 性能优化

- 为常用查询字段添加数据库索引
- 使用 Prisma 的 include 和 select 优化查询
- 合理使用分页限制查询结果数量
- 定期清理无用数据