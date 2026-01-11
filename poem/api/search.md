# 搜索API文档

## 概述

本项目提供三个搜索API端点，分别用于搜索古诗、作者和读书课文章。所有API都使用GET方法，接受查询参数`q`，返回JSON格式的搜索结果。

## API端点

### 1. 古诗搜索 API

**端点：** `/api/search/poem`

**功能：** 在古诗的标题、作者、内容和标签中搜索匹配项

**请求参数：**
- `q` (必需): 搜索关键词

**响应格式：**
```typescript
{
  results: Array<{
    title: string;       // 古诗标题
    version: string;     // 版本信息
    author: string;      // 作者
    snippet: string;     // 匹配的句子片段
    matchType: 'title' | 'author' | 'content' | 'tags'; // 匹配类型
  }>
}
```

**搜索优先级：** ``标题`` > ``作者`` > ``内容`` > ``标签``

**特殊处理：** 对于内容匹配，会提取包含搜索词的完整句子

---

### 2. 作者搜索 API

**端点：** `/api/search/author`

**功能：** 在作者的姓名、朝代、称号和标签中搜索匹配项

**请求参数：**
- `q` (必需): 搜索关键词

**响应格式：**
```typescript
{
  results: Array<{
    name: string;        // 作者姓名
    dynasty: string;     // 朝代
    epithet: string;     // 称号
    tags: string[];      // 标签数组
    avatar?: string;     // 头像URL（可选）
    matchType: 'name' | 'dynasty' | 'epithet' | 'tags'; // 匹配类型
  }>
}
```

**搜索优先级：** ``姓名`` > ``朝代`` > ``称号`` > ``标签``

---

### 3. 读书课文章搜索 API

**端点：** `/api/search/article`

**功能：** 在文章的标题、作者、摘要和内容中搜索匹配项

**请求参数：**
- `q` (必需): 搜索关键词

**响应格式：**
```typescript
{
  results: Array<{
    title: string;       // 文章标题
    author: string;      // 作者
    matchType: 'title' | 'author' | 'abstract' | 'content'; // 匹配类型
    snippet?: string;   // 内容匹配时的智能片段（可以匹配abstract和content）
  }>
}
```

**搜索优先级：** ``标题`` > ``作者`` > ``内容`` > ``摘要``

**特殊处理：** 对于内容匹配，使用智能片段提取算法：
- 找到关键词的第一个出现位置
- 向前查找最近的标点符号（。，；！…？）或文章开头
- 从标点符号后开始截取 30 个字符
- 确保片段包含完整的搜索关键词

## 错误处理

所有API在发生错误时返回标准错误响应：
```typescript
{
  error: string  // 错误信息
}
```

HTTP状态码：
- 200: 成功
- 500: 服务器内部错误

## 使用示例

### 搜索古诗
```bash
curl "/api/search/poem?q=明月"
```

### 搜索作者
```bash
curl "/api/search/author?q=李白"
```

### 搜索文章
```bash
curl "/api/search/article?q=诗词"
```

## 技术实现

- **数据库：** 使用Prisma ORM连接PostgreSQL数据库
- **搜索逻辑：** 使用PostgreSQL的 `contains` 操作符进行模糊搜索（不区分大小写）
- **结果去重：** 使用Map数据结构基于ID去重
- **排序：** 按匹配类型优先级排序，确保最相关的结果排在前面

## 性能考虑

- 每个搜索类型限制返回10-20条结果
- 使用数据库索引优化搜索性能
- 智能片段提取在服务端完成，减少前端处理负担

## 扩展性

API设计易于扩展，可以轻松添加新的搜索字段或修改搜索算法。当前架构支持快速添加新的搜索类型。
        