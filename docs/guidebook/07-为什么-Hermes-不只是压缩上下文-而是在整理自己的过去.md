---
title: 为什么 Hermes 不只是压缩上下文，而是在整理自己的过去
date: 2026-04-17
status: draft
---

# 为什么 Hermes 不只是压缩上下文，而是在整理自己的过去

## 先回答这篇最关键的问题

到前一篇为止，Hermes 的前三层积累结构已经站得比较稳了：

- memory 让它能记住事实
- session recall 让它能想起过去
- skill 让它能把做法沉淀成 procedural memory

但问题并没有结束。

因为一个系统只要真的开始长期积累自己，很快就会撞上另一类难题：

> **过去越积越多之后，系统还能不能继续把这些过去带着往前跑？**

如果答案只是“把历史全塞进上下文”，那这套自我进化机制很快就会反噬自己。

历史会越来越长，成本会越来越高，注意力会越来越散，真正重要的经验反而会被海量旧内容埋掉。

这正是为什么 Hermes 的 `context_compressor.py` 不能只被理解成一个 token 节约器。

它当然在压缩上下文，但它真正做的事更接近：

> **把系统自己的过去重新整理成还能继续工作的形式。**

如果把这篇先压成一句话，可以先立这个判断：

> **Hermes 不只是压缩上下文，而是在整理自己的过去：保住该保住的头尾，压缩中段历史，持续更新总结，并配合 prompt caching，把长会话中的经验重新整理成还能继续运行的形状。**

---

## 一、Hermes 在这里要解决的，不是“消息太多”，而是“过去会不会拖垮未来”

讨论 context compression 时，最容易把问题看窄。

表面看，它像是在解决：

- token 快超了
- prompt 太长了
- 该删点历史了

但如果只是这么理解，你会低估 Hermes 在这里的判断。

因为在前面几篇里，我们已经看到：

- Hermes 想留下长期事实
- 想保留可召回的过去会话
- 想沉淀可复用的 skill

也就是说，它不是一个“尽量少留历史”的系统，恰恰相反，它是一个**有意积累历史**的系统。

问题就来了：

> **一个会主动积累自己的系统，怎样才能不被自己的过去压死？**

这才是 `context_compressor.py` 真正要回答的问题。

所以 Hermes 在这里处理的不是单纯的“上下文变长”，而是：

- 什么该留下
- 什么该压缩
- 什么该被删成占位符
- 什么必须以更低成本的形式继续存在

换句话说，compression 在 Hermes 这里不是删减，而是整理。

---

## 二、`context_compressor.py` 一开头就说明了：它不是粗暴截断，而是有明确整理策略

`agent/context_compressor.py` 文件头写得非常清楚：

- uses auxiliary model (cheap/fast) to summarize middle turns
- protecting head and tail context
- structured summary template
- iterative summary updates
- tool output pruning before LLM summarization
- scaled summary budget

这几句已经把 Hermes 的思路全暴露出来了。

它不是这样：

- 超了就砍掉前半段
- 或者简单留最近 N 条消息

它做的是另一种更有组织的事：

> **先区分不同位置、不同价值的历史，再决定哪些保留原样、哪些压缩、哪些删成低成本表示。**

也就是说，Hermes 对历史的态度不是“尽量多留”或“尽量多删”，而是：

- 头部有头部的价值
- 尾部有尾部的价值
- 中段历史要换一种存在方式

这已经不是日志清理思维，而是经验整理思维。

---

## 三、为什么 Hermes 要保护 head 和 tail：因为过去不是平均重要的

`context_compressor.py` 的核心流程写得很清楚：

1. 先 prune old tool results
2. protect head messages
3. protect tail messages by token budget
4. summarize middle turns
5. subsequent compactions 时迭代更新 previous summary

这组步骤里最重要的判断是：

> **历史的不同位置，价值并不相同。**

### 1. head 不是“最老的部分”，而是当前会话的起点语义

文件里直接写着：

- protect head messages（system prompt + first exchange）

这说明 Hermes 眼里的 head 不是“老消息”，而是：

- 系统身份
- 本轮会话的起始语境
- 初始目标与问题设定

这些内容一旦丢掉，后面再长的历史都容易失去坐标。

所以它必须保。

### 2. tail 不是“最新消息”，而是当前工作面的活跃边界

文件里又写着：

- protect tail messages by token budget
- token-budget tail protection instead of fixed message count

这说明 Hermes 不是机械地“保最后 N 条”，而是在保：

> **距离当前工作面最近、最可能继续影响下一步推理的那一段活跃上下文。**

也就是说，tail 之所以重要，不是因为它新，而是因为它还活着。

### 3. middle 才是最适合被整理的部分

这一步非常关键。

Hermes 不会把整个过去都压扁。

它真正要动的是中段历史。

因为那一段通常最容易出现两种问题：

- 有价值，但不值得继续逐字保留
- 信息很多，但已经不该再占原始体积

这时，summary 才有意义。

所以 Hermes 不是在“压掉过去”，而是在：

> **把过去从原始形态转换成更适合继续携带的形态。**

---

## 四、tool output pruning 说明 Hermes 连“怎么删”都在做经验判断

`context_compressor.py` 里有一条很容易被忽略，但非常值钱：

- Tool output pruning before LLM summarization
- `_PRUNED_TOOL_PLACEHOLDER = "[Old tool output cleared to save context space]"`

这说明 Hermes 不是把所有旧内容都送去做 LLM summary。

它会先做一层更便宜的预处理：

- 先清旧 tool output
- 用占位符替代
- 再把真正值得总结的部分交给 summarizer

这件事看起来像优化，但本质上仍然是判断。

因为 Hermes 已经默认接受一个事实：

> **不是所有历史都值得进入“经验总结”这一步。**

有些旧 tool output 的信息密度太低、冗余太高、对当前未来帮助太小。

与其让 summarizer 花昂贵注意力去吃这些内容，不如先把它们降成廉价表示。

这一步说明 Hermes 的 compression 不是“交给模型统统概括”，而是：

- 先做便宜清理
- 再做重点总结

这非常像一个成熟系统会做的事情。

---

## 五、iterative summary update 才是最能说明“整理自己的过去”的地方

如果说前面这些设计还可以被理解成“更聪明的压缩”，那么 `iterative summary updates` 这一步就已经明显超出普通压缩器了。

`context_compressor.py` 文件头直接把它列成改进点。

后面的实现里又保留了：

- `self._previous_summary`
- subsequent compactions 时 update previous summary

这意味着 Hermes 不是每压一次就从零重新总结一遍。

它会把上一次总结过的内容继续带着，再吸收新压缩进来的那段历史。

这一步很关键，因为它改变了 compression 的性质。

### 1. 普通压缩器做的是“这次压一下”

很多上下文压缩逻辑本质上更像一次性事务：

- 这次太长了
- 总结一下
- 结束

下一次再长了，就再来一次新的总结。

问题是这样做很容易丢掉跨 compaction 的连续性。

### 2. Hermes 做的是“持续维护一份被整理过的过去”

有了 `_previous_summary` 之后，summary 就不再只是一次性应急产物。

它更像：

> **系统对自己过去的一份持续更新版整理稿。**

这和普通“截断+总结”最大的不同就在这里。

因为它承认：

- 历史不是只压一次
- 长会话里会多次进入 compaction
- 系统需要的是跨多次压缩仍然能保持连续性的过去摘要

这时候，summary 就已经从“节约 token 的副产物”变成了“长期运行中的经验整理层”。

这正是 Hermes 味道很重的一步。

---

## 六、为什么 structured summary template 很重要：因为 Hermes 要的不是短，而是可继续工作

文件头里还有一条很关键：

- Structured summary template (Goal, Progress, Decisions, Files, Next Steps)

这说明 Hermes 的 summary 不是“随便概括一下说了什么”，而是：

- Goal
- Progress
- Decisions
- Files
- Next Steps

这组字段非常能说明问题。

因为它要保留的不是聊天氛围，也不是原文修辞，而是：

> **以后继续推进任务时最需要的结构性信息。**

也就是说，Hermes 压缩历史时保住的是：

- 目标是什么
- 已经做到哪
- 做过哪些决策
- 涉及哪些文件
- 接下来该往哪走

这已经非常接近“工作记忆的结构化重写”了。

所以它要的不是“更短的过去”，而是：

> **更适合继续工作的过去。**

---

## 七、prompt caching 也不是单纯省钱，而是在配合这套“稳定过去”策略

这一点很多人容易忽略。

`agent/prompt_caching.py` 文件头写得很直白：

- Reduces input token costs by ~75% on multi-turn conversations
- caching the conversation prefix
- 4 cache_control breakpoints：system prompt + last 3 non-system messages

如果只从表面看，这像一个成本优化模块。

但它和前面的 frozen snapshot、head/tail 保护、iterative summary 其实是连在一起的。

### 1. caching 要依赖稳定前缀

prompt caching 的前提是：

- 前缀别乱动

而 Hermes 前面几篇里已经在不断保这件事：

- memory 用 frozen snapshot 保 system prompt 稳定
- compression 尽量保住 head
- 历史不是随意抖动，而是整理后再续上

这意味着 caching 不是孤立技巧，而是建立在“稳定前缀”之上的。

### 2. caching 让“整理过的过去”能更便宜地继续存在

如果 compression 负责把过去整理成更短、更稳的形态，那么 caching 负责的就是：

- 让这份已经整理好的前缀，以更低成本继续留在后续回合里

也就是说：

- compression 在整理过去
- caching 在稳定复用这份整理过的过去

这两者连起来，Hermes 才不会变成：

- 一边拼命积累历史
- 一边被历史的成本拖垮

所以 prompt caching 在这套结构里，不只是账单优化，而是经验整理系统的一部分。

---

## 八、所以 Hermes 的 compression / caching 更像“经验自优化”，而不是“上下文急救”

现在可以把前面几条重新压成一条总判断了。

如果这是一个普通 agent，它的 compression 常常只是：

- 上下文超了
- 快总结一下
- 别炸

这是急救逻辑。

Hermes 不是。

它已经有：

- memory
- session recall
- skill

也就是说，它本来就在持续积累过去。

在这种前提下，compression / caching 的意义就变了。

它们不再只是“上下文不够了怎么办”，而变成：

> **一个会长期积累自己的系统，怎样把这些过去整理成还能继续携带、继续复用、继续运行的形式。**

所以我会把它叫作：

- 经验自优化
- 或者过去整理系统

而不是单纯的上下文压缩器。

---

## 九、现在可以把前四层重新连起来了

写到这里，Hermes 的自我进化主线已经开始显出完整形状：

### 1. memory：记住长期事实

回答的是：

- 以后一直要带着什么

### 2. session recall：按需想起过去经验

回答的是：

- 当前任务里该把哪些过去调回来

### 3. skill：沉淀程序化做法

回答的是：

- 哪些成功方法值得正式变成以后可复用的能力

### 4. compression / caching：把过去整理成还能继续工作的形状

回答的是：

- 当过去越来越多时，系统怎样不被自己的历史拖垮

到这一步，Hermes 的“持续积累自己”才真正闭环。

因为它不只会留下过去，它还会继续管理过去。

---

## 十、最后收一句：Hermes 不是在丢弃历史，而是在把历史整理成下一轮还能带着走的样子

如果把这篇最后只留一句话，我会留这句：

> **Hermes 的 compression / caching 不是为了在上下文快炸时临时救火，而是为了把越来越多的过去经验重新整理成还能继续携带、继续复用、继续工作的形状。它压缩的不是价值，而是原始体积；它保住的不是原文，而是未来继续运行所需要的结构。**

如果后面继续写，最自然进入的就不是“过去怎么整理”，而是最后那个真正最像反思的环节：

> 为什么 Hermes 的主循环在输出之后还没真正结束，background review 才是经验沉淀的关键转折？

---

## 系列内继续阅读

- 上一篇：`06-为什么-skill-system-才是-Hermes-最像自我进化的地方.md`
- 回到阅读入口：`2026-04-16-Hermes-自我进化阅读路线图-v1.md`
- 如果你想回看这组文章为什么这样排：`2026-04-17-Hermes-特色与不同点系列规划-v1.md`
- 下一篇：`08-为什么-Hermes-的主循环收口后还没结束-background-review-才是经验沉淀的关键转折.md`
