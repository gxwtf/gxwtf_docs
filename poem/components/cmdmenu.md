# CommandMenu 组件文档

## 概述

`CommandMenu` 是一个全局搜索命令菜单组件，提供对古诗文、作者和读书课文章的快速搜索功能。

## 功能特性

- **全局快捷键支持**: ⌘K (Mac) 或 Ctrl+K (Windows/Linux)
- **多类型搜索**: 同时搜索古诗文、作者和读书课
- **实时搜索**: 500ms 防抖优化
- **智能高亮**: 搜索结果中关键词自动高亮显示
- **分类展示**: 搜索结果按类型分组显示

## 接口定义

### SearchResult 接口
```typescript
interface SearchResult {
    title: string
    version: string
    author: string
    snippet: string
    matchType: 'title' | 'author' | 'content' | 'tags'
}
```

### AuthorSearchResult 接口
```typescript
interface AuthorSearchResult {
    name: string
    dynasty: string
    epithet: string
    tags: string[]
    matchType: 'name' | 'dynasty' | 'epithet' | 'tags'
    avatar?: string
}
```

### ArticleSearchResult 接口
```typescript
interface ArticleSearchResult {
    title: string
    author: string
    snippet: string
    matchType: 'title' | 'author' | 'abstract' | 'content'
}
```

## 状态管理

- `open`: 控制命令菜单的显示/隐藏状态
- `searchValue`: 当前搜索输入值
- `searchQuery`: 防抖处理后的搜索查询（500ms延迟）
- `searchResults`: 古诗文搜索结果
- `authorResults`: 作者搜索结果
- `articleResults`: 读书课搜索结果
- `isSearching`: 搜索进行中状态

## 搜索逻辑

1. **快捷键监听**: 监听 ⌘K/Ctrl+K 快捷键打开命令菜单
2. **防抖处理**: 用户输入后延迟500ms才触发搜索
3. **并行搜索**: 同时向三个API端点发起搜索请求
4. **结果处理**: 分别处理古诗文、作者和读书课的搜索结果
5. **状态更新**: 更新对应的结果状态并设置搜索完成

## 显示逻辑

### 搜索结果分组
- **作者搜索结果**: 显示格式为"姓名｜朝代｜称号"
- **古诗文搜索结果**: 显示格式为"标题｜作者｜片段"
- **读书课搜索结果**: 显示格式为"标题｜作者｜片段"

### 头像显示
- 作者搜索结果优先显示头像（如果存在）
- 头像尺寸: h-4 w-4 rounded-full object-cover
- 无头像时显示搜索图标

### 高亮处理
使用 `HighlightedText` 组件实现关键词高亮，高亮颜色使用主题色 `text-[var(--theme-color)]`

## 路由跳转

- **作者详情**: `/author/${encodeURIComponent(result.name)}`
- **古诗文详情**: `/poem/${encodeURIComponent(result.version)}/${encodeURIComponent(result.title)}`
- **读书课详情**: `/article/${encodeURIComponent(result.title)}`

## 快捷键

- **打开命令菜单**: ⌘K (Mac) 或 Ctrl+K
- **个人资料**: ⌘P
- **账单**: ⌘B
- **设置**: ⌘S

## 依赖组件

- `@/components/ui/command`: 命令对话框组件
- `lucide-react`: 图标组件
- `@/components/icon`: 自定义图标组件
- `next/link`: Next.js 路由链接
- `@react-hook/debounce`: 防抖钩子
- `@/hooks/use-is-mac`: 平台检测钩子

## 使用示例

```tsx
import { CommandMenu } from "@/components/command-menu"

function App() {
    return (
        <div>
            {/* 其他组件 */}
            <CommandMenu />
        </div>
    )
}
```

## 性能优化

- **防抖处理**: 减少不必要的API调用
- **并行请求**: 同时发起多个搜索请求提高响应速度
- **结果缓存**: 组件内部状态管理避免重复渲染
- **懒加载**: 搜索结果按需加载和显示

## 错误处理

- API请求失败时清空搜索结果
- 控制台输出错误信息便于调试
- 用户界面显示友好的错误状态

## 扩展性

组件设计易于扩展新的搜索类型和功能，接口定义清晰便于维护。