## 数据库结构

这里是数据库存储结构，存放在 ``/prisma/schema.prisma`` 内。

## 1. Poem 模型（古诗文）
```js
model Poem {
  id        String   @id @default(cuid())
  title     String   // 古诗标题
  version   String?  // 版本信息
  tags      String[] // 标签数组
  author    String?  // 作者
  dynasty   String?  // 朝代
  mode      String?  // 模式
  content   String?  // 内容（使用#分隔段落，/分隔句子）
}
```

## 2. Author 模型（作者）
```js
model Author {
  id      String   @id @default(cuid())
  name    String   @unique // 作者姓名（唯一）
  dynasty String?  // 朝代
  epithet String?  // 称号
  quote   String?  // 名言
  avatar  String?  // 头像URL（可选）
  intro   String?  // 简介
  tags    String[] // 标签数组
}
```

## 3. Article 模型（读书课）
```js
model Article {
  id       String   @id @default(cuid())
  title    String   @unique // 文章标题（唯一）
  author   String?  // 作者
  dynasty  String?  // 朝代
  views    Int      @default(0) // 浏览量
  abstract String?  // 摘要
  content  String?  // 内容
  img      String?  // 图片URL
  tags     String[] // 标签数组
}
```