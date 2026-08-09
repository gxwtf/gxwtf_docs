# 遗忘曲线

广学英语的「遗忘曲线」并非简单的时间到期提醒，而是把 **SM-2 间隔重复算法** 与 **错误次数加权** 结合，形成一个二维权重，在每次 AI 出题时按权重抽样决定哪些词优先出现。

## 设计目标

1. **优先复习快忘的词**（基于 SM-2 的记忆稳定性）
2. **优先抽常错的词**（基于错误次数平方权重）
3. **尊重用户手动选词**（候选池 = 用户选中的词，不扫全词库）

## 核心公式

```
totalWeight(w) = f(t) × g(e)
```

| 因子 | 公式 | 含义 |
| --- | --- | --- |
| `f(t)` | `1 - e^(-elapsed/interval)` | 遗忘曲线权重：回忆概率越低，权重越高 |
| `g(e)` | `e²` | 错误权重：错误次数平方增长（`e` 初值 1） |

### f(t) —— 遗忘曲线权重

基于 SM-2 算出的 `interval`（记忆稳定性 S）和 `lastReviewedAt` 计算：

- 刚复习完（elapsed=0）：`f=0` → 不再出，避免连刷
- elapsed = 0.693 × interval：`f=0.5` → 视为「到期」阈值
- elapsed = interval：`f≈0.63`
- elapsed = 2 × interval：`f≈0.86` → 严重超期
- 从未复习：`f=1.0`（基础权重）

### g(e) —— 错误权重

| 错误次数 e | g(e) |
| --- | --- |
| 1（新词） | 1 |
| 2 | 4 |
| 6 | 36 |
| 11 | 121 |
| 20（上限） | 400 |

### 总权重示例

| 词 | elapsed/interval | errorCount | f(t) | g(e) | total |
| --- | --- | --- | --- | --- | --- |
| 新词（从未复习） | — | 1 | 1.0 | 1 | 1.0 |
| 刚批改完 | 0/6 | 1 | 0 | 1 | 0（不出） |
| 到期日当天 | 6/6 | 1 | 0.63 | 1 | 0.63 |
| 严重超期+多次错 | 30/6 | 6 | 0.99 | 36 | 35.7 |

## SM-2 状态更新

### quality 映射（0–5，<3 视为遗忘）

| 题型 | 数据 | quality |
| --- | --- | --- |
| fill-blank / definition-fill-blank | isCorrect | correct→5，wrong→2 |
| meaning-select / meaning-select-en | isCorrect | correct→5，wrong→2 |
| translate / word-select-translate | score 0–10 | ≥8→5, ≥6→4, ≥4→3, <4→2 |
| 空答案（放弃） | — | 1（视为错误，更新遗忘曲线） |
| word-card | — | null（查看型，不更新） |

### SM-2 核心公式

```
答对 (quality >= 3):
  repetitions=0 → interval=1
  repetitions=1 → interval=6
  repetitions≥2 → interval=round(interval × ef)
  repetitions++
答错 (quality < 3):
  repetitions=0, interval=1

ef = clamp(ef + (0.1 - (5 - quality) × (0.08 + (5 - quality) × 0.02)), [1.3, 3.0])
```

### errorCount 更新

```
答对 (quality >= 3):
  errorCount = clamp(max(1, ceil(errorCount / 2)), [1, 20])   // 快速衰减
  correctReviews++
答错 (quality < 3):
  errorCount = clamp(errorCount + 1, [1, 20])
```

衰减序列（从 e=11 起）：11 → 6 → 3 → 2 → 1，**4 步清零**，难词权重快速回落，避免霸占队列。

## 数值溢出防护

| 参数 | 上下限 | 说明 |
| --- | --- | --- |
| `errorCount` | [1, 20] | 防止 g(e)=e² 爆炸，上限 400 |
| `interval` | [1, 365] 天 | 与 Anki 默认一致 |
| `ef` | [1.3, 3.0] | 难度因子范围 |
| 加权抽样 | weight > 0 才参与 | 全为 0 时回退等概率抽样 |
| `f(t)` | stability ≥ 1 | 防止除零 |

## 加权抽样算法

采用 **Efraimidis-Spirakis 加权无重复抽样**（`src/lib/spaced-repetition/selector.ts`）：

1. 过滤掉 `weight = 0` 的词（不参与抽样）。
2. 若全部为 0，回退 Fisher-Yates 等概率洗牌。
3. 给每个词生成 `key = u^(1/w)`（`u` 为 [0,1] 均匀随机数）。
4. 取 `key` 最大的 k 个词。

复杂度 O(n log k)，千词级别毫秒级完成。

## 复习状态如何被更新

题目批改完成后，`recordReviewFromQuestion`（`src/actions/review.ts`）执行：

1. 加载 `QuestionQueue`，取出 `wordIds`、`questionContent`、`lastAnswer`、`gradingResult`。
2. 对每道小题，反推它对应的目标 `wordId`：
   - fill-blank / definition-fill-blank：`originalWord || answer`
   - translate / word-select-translate：`keyWords[0]`
   - meaning-select / meaning-select-en：`word`
3. 由批改结果映射出 `quality`（见上表）。
4. 对每个 `wordId`：`sm2Update` 更新 `repetitions/ef/interval`，按对错更新 `errorCount`，`totalReviews++`，`lastReviewedAt = now`。
5. 事务内批量 `upsert` 到 `WordReviewState`。

> 仅 `ANSWERED` 状态才记录复习；批改失败不更新；重置重做算一次新复习；关联词与干扰词不参与。

## 模块组织

```
src/lib/spaced-repetition/
├── sm2.ts          // SM-2 状态更新（纯函数）
├── weights.ts      // f(t)、g(e)、totalWeight、clamp 工具（纯函数）
├── quality.ts      // gradingResult → quality 映射
└── selector.ts     // 加权抽样（Efraimidis-Spirakis）

src/actions/review.ts          // Server Actions
src/lib/word-selection.ts      // selectWordsForQuestion（调用 weightedSample）
```

## UI 体现

- 单词本每张卡片底部展示三个权重值（总 / 遗忘 / 错误），鼠标悬浮可见公式说明。
- AI 出题对话框最上方有「开启遗忘曲线 `NEW`」复选框，默认开启，存于 `localStorage`（key: `ai-question-use-spaced-repetition`）。
- 关闭后：候选词多于所需数量时走等概率随机，与遗忘曲线无关。

## 边界情况

| 场景 | 处理 |
| --- | --- |
| 所有候选词权重都为 0（刚做完） | 回退等概率抽样 |
| 候选池为空（没选词） | 前端禁用出题按钮，后端返回空数组 |
| 候选池词数 < 所需数量 | 返回全部 + 优先取关联词补足 |
| 题目批改失败 | 不更新复习状态，可重试 |
| 单词被删除 | Prisma `onDelete: Cascade` 自动级联删除 `WordReviewState` |
| 空答案（放弃小题） | `quality=1`（视为错误），更新遗忘曲线：errorCount+1、interval 重置 |
| 新用户首次使用 | 全部词 weight=1，等价纯随机，无感过渡 |
| 时区 | 全部用 UTC `Date`，`getTime()` 绝对时间戳比较 |
