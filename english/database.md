# 数据库结构

数据库为 PostgreSQL，通过 Prisma ORM 访问，Schema 定义在 `prisma/schema.prisma`。

## 数据表概览

| 表 | 说明 |
| --- | --- |
| `User` | 用户信息（从广学账号 SSO 同步） |
| `Word` | 用户的生词及其释义、关联词 |
| `QuestionQueue` | AI 生成的题目队列 |
| `Tag` | 标签（单词本与作文积累本共用） |
| `WordTag` | 单词与标签的多对多关联 |
| `RelatedWord` | 单词间关联关系（不同形 / 易混） |
| `TagConfig` | 用户级标签配置（名称、颜色、描述） |
| `WritingEntry` | 作文积累条目 |
| `WritingEntryTag` | 作文条目与标签的多对多关联 |
| `WordReviewState` | 单词复习状态（SM-2 + 错误统计） |

## User

从广学账号 SSO 同步而来的用户信息，`userId` 与广学账号一致。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | Int (PK, 自增) | 本地主键 |
| `userId` | Int (unique) | 广学账号用户 ID，业务主键 |
| `userName` | String | 用户名 |
| `admin` | Int (默认 0) | 0 普通 / 1 管理员 |
| `email` | String? | 邮箱 |
| `realName` | String? | 真实姓名 |
| `createdAt` / `updatedAt` | DateTime | 时间戳 |

关联：`words`、`questions`、`relatedWords`、`tagConfigs`、`writingEntries`、`wordReviewStates`，均 `onDelete: Cascade`。

## Word

用户的生词。`(userId, text)` 唯一约束。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | Int (PK) | 主键 |
| `userId` | Int | 所属用户（关联 `User.userId`） |
| `text` | String | 单词文本 |
| `meanings` | Json[] | 用户不熟悉的释义列表（结构化） |
| `relatedWords` | Json? | 关联词（`{ text, type }`） |
| `createdAt` / `updatedAt` | DateTime | 时间戳 |

> `meanings` 由词典查询回填，用户可只勾选自己不熟悉的释义。

## QuestionQueue

AI 生成的题目。`id` 为 cuid。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | String (PK, cuid) | 题目 ID |
| `userId` | Int | 所属用户 |
| `questionType` | String | 题型（见下） |
| `status` | QuestionStatus | 状态枚举 |
| `questionContent` | Json? | 题面（各题型结构不同） |
| `lastAnswer` | Json? | 用户最近一次作答 |
| `wordIds` | Int[] | 涉及的单词 ID |
| `gradingResult` | Json? | 批改结果 |
| `relatedWordEntries` | Json? | 关联词信息 |
| `createdAt` / `updatedAt` | DateTime | 时间戳 |

### QuestionStatus 枚举

```
GENERATING       // 生成中
GENERATED        // 已生成，待作答
ANSWERED         // 已作答并批改完成
FAILED           // 生成失败
GRADING          // 批改中
GRADING_FAILED   // 批改失败
```

### 题型（questionType）

`fill-blank` / `translate` / `meaning-select` / `meaning-select-en` / `definition-fill-blank` / `word-select-translate` / `word-card`

## Tag / WordTag / WritingEntryTag

- `Tag`：全局标签，`name` 唯一，含 `colorId`（默认 `blue`）与 `description`。
- `WordTag`：单词 ↔ 标签，`(wordId, tagId)` 唯一。
- `WritingEntryTag`：作文条目 ↔ 标签，`(writingEntryId, tagId)` 唯一。
- 删除标签时，关联表通过 `onDelete: Cascade` 自动清理。

## RelatedWord

记录单词间的关联关系。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `userId` | Int | 所属用户 |
| `wordText` | String | 源单词文本 |
| `relatedText` | String | 关联单词文本 |
| `type` | String | `different_form`（不同形）/ `easily_confused`（易混） |

索引：`(userId, wordText)`。

## TagConfig

用户级标签配置（颜色、描述），`(userId, name)` 唯一。与全局 `Tag` 配合使用，允许每个用户对同一标签名有不同的颜色 / 描述偏好。

## WritingEntry

作文积累条目（独立模块，本文档仅作数据结构说明）。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | Int (PK) | 主键 |
| `userId` | Int | 所属用户 |
| `content` | String | 作文素材内容 |
| `note` | String? | 备注 |
| `createdAt` / `updatedAt` | DateTime | 时间戳 |

索引：`userId`。

## WordReviewState

单词复习状态，每个「用户 × 单词」一行，服务于[遗忘曲线](/english/forgetting-curve.md)。`(userId, wordId)` 唯一。

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `userId` | Int | 所属用户 |
| `wordId` | Int | 关联 `Word.id` |
| `repetitions` | Int (默认 0) | 连续正确次数 |
| `ef` | Float (默认 2.5) | 难度因子，范围 [1.3, 3.0] |
| `interval` | Int (默认 1) | 当前间隔（天），范围 [1, 365] |
| `errorCount` | Int (默认 1) | 错误次数，范围 [1, 20] |
| `totalReviews` | Int (默认 0) | 总复习次数 |
| `correctReviews` | Int (默认 0) | 答对次数 |
| `lastReviewedAt` | DateTime? | 最近一次批改完成时间（f(t) 基准） |

设计要点：

- **不存 `dueDate`**，到期由 `lastReviewedAt + interval` 运行时计算。
- **不建 ReviewLog**，保持极简。
- `errorCount` 默认 1（非 0），保证新词有基础权重。
- 删除单词时通过 `onDelete: Cascade` 自动级联删除该词的复习状态。

## Migration 策略

- 仅新增表 + 索引，不修改 / 删除现有表数据。
- 新增 `WordReviewState` 时，现有单词默认视为「从未复习」（无对应行，运行时按 `f=1, g=1` 处理），无感过渡。
- 迁移文件位于 `prisma/migrations/`，按日期命名（如 `20260806_add_word_review_state`）。

> 数据库高危操作（删表、清空、`prisma migrate reset` 等）必须经人工确认后才可执行，详见项目 `CLAUDE.md`。
