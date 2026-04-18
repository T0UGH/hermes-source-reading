---
title: 为什么 Hermes 的主循环收口后还没结束：background review 才是经验沉淀的关键转折
date: 2026-04-17
status: draft
---

# 为什么 Hermes 的主循环收口后还没结束：background review 才是经验沉淀的关键转折

## 先回答这篇最关键的问题

到前一篇为止，Hermes 这组自我进化主线其实已经拉出了四层：

- memory：记住长期事实
- session recall：按需想起过去经验
- skill：沉淀程序化做法
- compression / caching：整理越来越多的过去

但如果只看到这里，你还是很容易把 Hermes 理解成这样一种系统：

- 任务来了
- 主循环跑一遍
- 工具调用结束
- 输出结果
- 本轮结束

这当然是很多 agent 的默认节奏。

但 Hermes 真正有辨识度的地方，恰恰是它不把“给用户结果”当成经验处理的终点。

`run_agent.py` 里真正值钱的一步，是它在输出之后还要做一件事：

> **回头再看刚刚这一轮，到底有没有什么东西值得留下。**

这就是 background review。

如果把这篇先压成一句话，可以先立这个判断：

> **Hermes 的主循环在用户看到结果时还没有真正结束。真正把“这一轮完成了什么”转成“以后还能继续用什么”的关键转折，不在最终回答本身，而在响应之后的 background review。**

---

## 一、普通 agent 的“结束”，在 Hermes 这里只是第一层结束

如果把大多数 tool-calling agent 压成最低分辨率，它们的一轮通常长这样：

- 用户给任务
- 模型决定要不要调工具
- 工具结果回流
- 模型继续判断
- 最后产出回答
- 这一轮结束

在这种结构里，“输出给用户”基本就等于 turn 的完成标志。

后面当然也可能有日志、持久化、analytics，但那通常不是 agent 自己经验处理主链的一部分。

Hermes 不一样。

因为前面几篇已经说明，它关心的不只是：

- 这轮任务有没有做完

还关心：

- 这轮有没有暴露新的长期事实
- 有没有形成值得复用的新做法
- 有没有什么经验值得转成下一轮能继续用的资产

这意味着它不能把“结果已经回给用户”直接等同于“这一轮的价值已经被处理完”。

也就是说：

> **用户侧的完成，不等于系统侧的沉淀完成。**

这就是 background review 之所以必要的根本原因。

---

## 二、`run_agent.py` 写得很明白：background review 是在响应之后启动的

这件事在源码里不是暗含逻辑，而是直接写出来了。

`run_agent.py` 在最终响应附近有一段很关键：

- 先判断 `_should_review_memory`
- 再判断 `_should_review_skills`
- 先做 `sync_all()` 和 `queue_prefetch_all()`
- 然后才进入 background review

更直接的是旁边那句注释：

- Background memory/skill review — runs AFTER the response is delivered
- so it never competes with the user's task for model attention

这两句其实已经把 Hermes 的思路说透了：

> **沉淀当然重要，但不能和当前用户任务抢主模型注意力。**

也就是说，Hermes 对“任务完成”和“经验沉淀”做了一个很漂亮的分相：

### 1. 先把当前任务收口给用户

这是用户真正关心的第一优先级。

### 2. 再在后台回看这一轮值不值得留下什么

这一步不该打断用户，也不该拖慢本轮主要交付。

所以 Hermes 把它放在 after-response 这个位置。

这个位置非常关键。

因为它说明 Hermes 并不是把 memory / skill review 硬塞进当前主循环中间，而是把它当成：

> **本轮响应之后的 post-hoc reflection。**

这已经明显不像普通 agent loop，更像一个会在做完之后再反思的系统。

---

## 三、先别急着看 review，本轮结束前 Hermes 已经在做两件“为下一轮准备”的事

background review 最好不要孤立看。

在它前面，`run_agent.py` 其实已经做了两件很有意思的事：

- `self._memory_manager.sync_all(original_user_message, final_response)`
- `self._memory_manager.queue_prefetch_all(original_user_message)`

这两步非常值得注意。

### 1. `sync_all()`：刚刚这一轮已经完成，先把 turn 级结果同步出去

这里的意思不是“正式保存长期记忆”，而是：

- 当前这一轮 user / assistant 交互已经结束
- memory provider 可以先拿到这个完成态的 turn

这更像完成回合同步，而不是精选沉淀。

### 2. `queue_prefetch_all()`：下一轮甚至已经开始被预热

这一步更有意思。

它说明 Hermes 的眼光已经不是停在“本轮结束”，而是提前把下一轮 recall / prefetch 的准备也排进去了。

也就是说，在 background review 还没开始之前，系统已经在做两件面向未来的动作：

- 同步这轮结果
- 预热下一轮可能要用的东西

这让整个 turn 的收口方式和普通 agent 很不一样。

普通 agent 的收口更像：

- 结果发出
- 结束

Hermes 的收口更像：

- 结果发出
- 同步本轮
- 预热下一轮
- 再回头审视这一轮有没有值得长期留下的东西

所以它不是单点结束，而是一整段 post-turn processing。

---

## 四、为什么 review 要放到后台：因为“交付结果”和“提炼经验”不是同一种注意力任务

这一点其实非常重要。

很多人看到 background review，第一反应会是：

> 那为什么不直接在主循环里边做边存？

Hermes 没这么做，原因很深。

因为这两类任务需要的注意力模式本来就不同。

### 1. 主循环中的注意力，应该服务当前用户任务

在主循环里，模型应该优先关注：

- 现在要解决什么
- 下一步该调什么工具
- 当前结果够不够收口
- 用户最需要的回答是什么

这是执行型注意力。

### 2. review 需要的是另一种“事后判断”

而 background review 关心的是：

- 刚才用户有没有透露长期偏好
- 刚才有没有暴露新的环境事实
- 刚才这次方法值不值得变成 skill
- 有没有现有 skill 该被更新

这是提炼型注意力。

如果把这两种注意力硬塞在同一个前台过程里，结果通常会很差：

- 要么当前任务被打断
- 要么 review 做得草率
- 要么系统为了“别漏掉沉淀”变得很啰嗦

Hermes 的做法其实很清醒：

> **先把执行做好，再把提炼放到后台。**

这一步本质上是在分离两种认知负担。

也正因为如此，background review 才会显得像真正的 reflection，而不是主循环里的顺手副作用。

---

## 五、`_MEMORY_REVIEW_PROMPT` / `_SKILL_REVIEW_PROMPT` 说明 Hermes 在显式训练“什么值得留下”

`run_agent.py` 里最有意思的一组文本，其实就是这几个 prompt：

- `_MEMORY_REVIEW_PROMPT`
- `_SKILL_REVIEW_PROMPT`
- `_COMBINED_REVIEW_PROMPT`

它们的内容写得非常明确。

### 1. memory review 问的不是“有没有内容”，而是“有没有值得长期记住的用户信息”

prompt 里重点看的是：

- 用户有没有透露 persona / desires / preferences / personal details
- 用户有没有表达工作风格、期待、希望你如何运作

这说明 Hermes 不是在问：

- 这轮说了很多话吗？

而是在问：

- **这轮有没有暴露长期有效的用户层信息？**

### 2. skill review 问的不是“有没有完成任务”，而是“有没有形成可复用方法”

`_SKILL_REVIEW_PROMPT` 里关心的是：

- 有没有用到 non-trivial approach
- 有没有 trial and error
- 有没有因为经验发现而中途改路
- 用户有没有期待不同方法或结果
- 如果 skill 已存在，就 update
- 否则 create new one

这几句很重要。

它说明 Hermes 不是因为“做成了一个任务”就默认写 skill，而是要看：

> **这次是不是形成了一种值得以后复用的方法。**

### 3. review prompt 本身就是一套经验沉淀标准

也就是说，background review 不只是“后台再跑一次模型”。

它其实是在用一组非常明确的标准，筛选：

- 什么该进入 memory
- 什么该进入 skill
- 什么不值得留下

这已经非常像一个系统在显式定义自己的学习准则了。

---

## 六、`_spawn_background_review()` 真正做的，是 fork 出一个小型反思代理

`_spawn_background_review()` 的实现也非常有意思。

它不是在原主循环里插一段同步代码，而是：

- 创建一个新的后台线程
- fork 出一个新的 `AIAgent`
- 继承当前模型、平台、provider 等环境
- 把刚才那轮的 `messages_snapshot` 作为历史
- 再把 review prompt 追加成下一条 user_message

这一步非常像什么？

像是在说：

> **当前主循环先做完交付；然后再派一个小型反思代理，专门回看这一轮有没有值得长期留下的东西。**

这几乎就是“系统级反思”的最直接实现。

### 1. 它不是主循环残余，而是一个独立的小反思回合

这个细节非常重要。

因为 Hermes 没有把 review 写成：

- 主线程里顺手多跑几行 if/else

它而是显式 fork 出一个 review agent。

这说明它把反思当成一项独立任务，而不是附属尾巴。

### 2. 它写进的是共享 memory / skill stores，但不改主会话历史

注释里也写得很清楚：

- Writes directly to the shared memory/skill stores
- Never modifies the main conversation history
- or produces user-visible output

这意味着 Hermes 在这里非常克制：

- 能沉淀，就沉淀到共享资产里
- 但不回头污染刚刚用户看到的主会话轨迹

这让经验沉淀和用户交互保持了解耦。

也就是说：

> **经验可以留下，但当前会话的表面叙事不必因此被改写。**

这点非常成熟。

---

## 七、nudge interval 这类触发条件，说明 Hermes 连“什么时候值得反思”都在做控制

background review 还有一个容易被忽略、但很重要的细节：

- `_memory_nudge_interval`
- `_skill_nudge_interval`
- `_turns_since_memory`
- `_iters_since_skill`

这些变量说明 Hermes 并不是每轮都无脑做一次完整 review。

它会根据条件判断：

- 什么时候值得做 memory review
- 什么时候值得做 skill review

### 1. memory review 更偏 turn 级节奏

`_should_review_memory` 和 `_turns_since_memory` 连在一起，说明它更像按会话轮次节奏来触发。

也就是：

- 不是每句话都存
- 也不是长时间完全不看
- 而是到一定 turn 节奏，再回头审视一次

### 2. skill review 更偏复杂度 / 工具迭代节奏

skill 这边看的是：

- `self._iters_since_skill`
- how many tool iterations THIS turn used

这很说明问题。

因为一套方法值不值得沉淀，往往和：

- 本轮是否复杂
- 是否经历了多轮试错
- 是否真的形成了一条做法路径

更相关。

所以 Hermes 在这里不是平均触发，而是按经验价值去估计触发时机。

这就进一步说明，background review 不是“例行收尾动作”，而是：

> **一种被条件化控制的经验提炼机制。**

---

## 八、所以 background review 才是“结果”变成“积累”的关键转折点

现在可以把前面几步重新压成一个总判断了。

如果没有 background review，Hermes 当然仍然可以：

- 记住已有 memory
- 查回旧 session
- 使用已有 skill
- 压缩已有上下文

但它会缺一层真正关键的转折：

> **刚刚这一轮新发生的经验，怎样从“任务过程”变成“未来资产”？**

background review 处理的正是这个转折。

它把：

- 刚才的用户表达
- 刚才的工作方式
- 刚才的试错过程
- 刚才形成的新判断

重新审视一遍，问的不是：

- 这轮任务做完了吗？

而是：

- 这轮到底有什么值得以后继续带着走？

这就是为什么我会说：

- 输出结果 = 当前任务收口
- background review = 本轮经验沉淀

前者解决交付。

后者解决积累。

而 Hermes 的不同点，就在于它把这第二步正式做成了系统动作。

---

## 九、现在可以把整条 self-evolution 主线重新压回 `run_agent.py` 了

写到这里，其实整条链已经很清楚了：

### 1. session start 前后

- frozen memory 进入 system prompt
- recall / prefetch 为当前回合准备背景

### 2. main loop 进行中

- 工具调用
- 推理推进
- 当前任务收口

### 3. turn 完成后

- `sync_all()` 同步当前回合
- `queue_prefetch_all()` 预热下一轮
- background review 回看这一轮值不值得留下什么

### 4. 长会话继续推进时

- compression / caching 整理越来越多的过去

这时候 Hermes 的主循环已经明显不再只是“tool-calling runtime”。

它更像：

> **一套在执行、回忆、沉淀、整理之间持续循环的经验处理系统。**

而 background review 正好站在这条链最像“反思”的位置上。

---

## 十、最后收一句：Hermes 的真正收口，不是把答案发出去，而是把经验留下来

如果把这篇最后只留一句话，我会留这句：

> **Hermes 的主循环真正的收口，不是用户已经收到答案，而是系统已经在答案之后回头审视这一轮：有没有新的长期事实、有没有新的可复用做法、有没有什么值得变成未来资产。background review 的价值，不在于再跑一次模型，而在于把“刚刚完成的任务”正式转成“以后还能继续使用的经验”。**

如果后面继续写，最自然的最后一篇就应该是：

> Hermes 怎样把 memory、recall、skills、compression 和 background review 重新接成一套真正的 self-evolution loop？

---

## 系列内继续阅读

- 上一篇：`07-为什么-Hermes-不只是压缩上下文-而是在整理自己的过去.md`
- 回到阅读入口：`2026-04-16-Hermes-自我进化阅读路线图-v1.md`
- 如果你想回看这组文章为什么这样排：`2026-04-17-Hermes-特色与不同点系列规划-v1.md`
- 下一篇：`09-Hermes-怎样把-memory-recall-skills-compression-接成真正的-self-evolution-loop.md`
