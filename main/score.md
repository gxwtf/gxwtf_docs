# 评分机制

**公布排行榜分数计算方式**

## 商城积分

100倍用户总分加游戏分数

## 提交分数：

$$score_{submit,subjects} = \pi (1+\sum_{s \in subjects}{excel\_count(s)})^e \sum_{s \in subjects}{\ln({submit\_count(s)+1})}$$

## 供题分数：

$$score_{problems}=\sum_{p \in problems}{score_{problems}}$$

## 用户总分：

$$score=score_{submit}+score_{problems}$$

## 题目分数：

$$credit\_users = \{ user \in users | user.score_{submit,all} \geq 11.4 \vee user.problem\_count\}$$

$$score=\frac{\sum_{user \in  credit\_users} score_{user} \cdot point_{user}}{\sum_{user \in  credit\_users}score_{user}}$$

## 学校分数：

$$score_{school}=\ln^2{(user\_count_{school}+2)} \sum_{user \in school } score_{submit}$$

## 游戏分数

以下游戏的分数之和为游戏总分

### Hangman

为猜出单词数量

### 诗词九宫格

为答对诗词的评分之和。答对一次的评分为：诗词方格长和宽分别是 $n$ 和 $m$，得分为 

$$\frac{(n \times m)^1.2}{20}$$

### Wordle

为答对单词的评分之和。答对一次的评分为：记猜测次数为 $m$

对于`Normal`模式，得分为

$$\frac{36}{m^2}$$

对于`Extended`模式，得分为

$$5.14^{4.5-m} + 0.5\pi$$

### 溶解追击战

我们定义一个参数 $k$，在简单、中等、困难、专家模式中，$k$ 的值分别为 $0.5,1,2,4$

令 $score$ 为游戏中显示的得分（$score\in [0,100]$）

则实际的得分为

$$0.01 score\times k$$

### 溶解追击战（新版）
设你总共碰到了 $n$ 个可溶方块，则你的最终得分为 $$0.01\times n^{1.5}$$

### 俄罗斯方块

为所有游戏中累计消除的行数

### 24点

为在没有查看提示的情况下解出题目的数量

### 化学猜一猜

有一个参数 $k$，对于 “ 猜物质”、“猜反应”、“猜句子”，$k$的值分别为：$10, 20, 40$

设你的猜测次数为 $cnt$，则你的得分为：

$$\frac{k}{\frac{1}{10}(cnt-1)+1}$$