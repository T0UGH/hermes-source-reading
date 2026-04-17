---
title: 为什么 Hermes 不是“有记忆的 agent”，而是“能持续积累自己的 agent”
date: 2026-04-16
status: draft
---

# 为什么 Hermes 不是“有记忆的 agent”，而是“能持续积累自己的 agent”

## 先回答读者最容易问错的那个问题

第一次看 Hermes，很容易先把它归进一个熟悉的类别里：

- 一个能调很多工具的 agent
- 一个有 CLI、gateway、cron、delegate 的 agent 框架
- 或者再进一步，觉得它的特别之处只是“多了 memory 和 skills”

这些理解都不算全错，但都还没有抓到 Hermes 最值得学的地方。

这篇真正要先立住的判断只有一句：

> **Hermes 最独特的地方，不在于“它也有记忆”，而在于它把记忆、会话回忆、技能沉淀与上下文自优化接成了一套会持续积累自己的系统闭环。**

也就是说，Hermes 的“变强”主要不是模型参数自己长出来，而是系统越来越会：

- 把环境和用户事实沉淀下来
- 把过去会话在需要时重新找回来
- 把成功方法写成以后可复用的 skill
- 把越来越长的历史压成还能继续工作的上下文

如果这个判断先没立住，后面你再看 `run_agent.py`、`model_tools.py`、`cli.py`、`gateway/`，就会一直像在看一个“功能很多的 agent 项目”；而不是在看一套**会不断整理并复用自己经验的系统**。

---

## 先把“自我进化”这个词去神秘化

这里最容易犯的错，是把“自我进化”听成一种比实际更玄的东西。

如果不先压清边界，读者会很自然代入下面几种想象：

- Hermes 会在运行时修改模型权重
- Hermes 会自动微调自己
- Hermes 会像在线 RL 一样让底层模型越来越聪明

但从源码看，Hermes 这里说的“进化”并不是这个层面。

它更接近一种**system-level growth**：

- 模型还是那个模型
- tool loop 还是那个 tool loop
- 但系统会越来越会整理、保留、召回和复用自己在运行中积累出来的经验

换句话说：

> **Hermes 不是靠“模型自己学会了更多”来变强，而是靠“系统越来越会处理自己的经验”来变强。**

这点非常重要。因为它决定了 Hermes 最值得读的，不是“它怎么做一个 agent”，而是：

> **它怎么把一次次对话、任务、方法和上下文变成之后还能继续使用的长期能力。**

---

## 一、先看最小总图：Hermes 的变强，不是一条线，而是四条闭环

如果把 Hermes 里最像“持续变强”的部分压成最低分辨率模型，可以先看这四条线：

1. **持久记忆闭环**
2. **会话回忆闭环**
3. **技能沉淀闭环**
4. **上下文自优化闭环**

这四条线分别回答的是不同问题。

### 1. 持久记忆闭环：系统怎样稳定记住事实
这条线的核心文件是：

- `tools/memory_tool.py`
- `agent/builtin_memory_provider.py`
- `agent/memory_manager.py`

它解决的问题不是“当前回复里临时记一句话”，而是：

- 用户有哪些稳定偏好
- 当前环境有哪些重要事实
- 项目有哪些长期约定
- 哪些工具 quirks 应该在以后继续记住

也就是说，这一层处理的是：

> **未来多次会话都还值得继续携带的稳定事实。**

### 2. 会话回忆闭环：系统怎样在需要时把过去经验调回来
这条线的核心文件是：

- `hermes_state.py`
- `tools/session_search_tool.py`

它解决的不是“固定记忆”，而是另一类问题：

- 之前我们是不是处理过类似问题
- 上次怎么做的
- 哪些文件、命令、错误信息值得重新调回来
- 当下这次任务最相关的那段过去经验是什么

这一层其实更像：

> **情境回忆，而不是长期事实记忆。**

### 3. 技能沉淀闭环：系统怎样把“知道”变成“会做”
这条线的核心文件是：

- `tools/skill_manager_tool.py`
- `agent/skill_commands.py`
- `agent/skill_utils.py`
- `tools/skills_tool.py`

这里最值钱的一句定义，其实源码已经直接写出来了：

> **Skills are the agent's procedural memory.**

也就是说，Hermes 不满足于“记住一个事实”，它还要把成功方法写成之后可复用的做法。

这是第三层闭环：

- 这次会做了
- 不是只留下印象
- 而是沉淀成正式技能
- 下次可以直接拿出来复用

### 4. 上下文自优化闭环：系统怎样不被自己的历史压垮
这条线的核心文件是：

- `agent/context_compressor.py`
- `agent/prompt_caching.py`
- `agent/trajectory.py`

这一层不是“记住更多”，而是：

- 怎么压缩长对话
- 怎么保留最该保留的头尾
- 怎么把中段历史总结成还能继续工作的形式
- 怎么让多轮成本变低
- 怎么把经验资产存下来

也就是说，Hermes 不只是积累经验，它还会：

> **主动把经验整理成更低成本、更可持续运行的形式。**

---

## 二、为什么这四条线加在一起，才构成 Hermes 的真正差异

如果只看其中一条，你很容易觉得这没什么特别：

- 有记忆的 agent 很常见
- 有历史搜索的 agent 也不少
- 有 skills 的系统也见过
- 有 context compression 的实现也不稀奇

Hermes 真正有意思的地方，不是“它有这些功能”，而是：

> **它把这些东西接成了一个互相补位的闭环。**

更具体地说：

### 1. Memory 负责沉淀稳定事实
`tools/memory_tool.py` 开头就把这个边界说得很清楚：

- `MEMORY.md` 记录环境事实、项目约定、工具 quirks、学到的东西
- `USER.md` 记录用户偏好、沟通风格、工作习惯

这已经说明，Hermes 的 memory 不是“再多存一点聊天内容”，而是在做一种**稳定事实层**。

### 2. Session recall 负责把过去会话重构成当前可用经验
`tools/session_search_tool.py` 的 flow 也很直接：

1. FTS5 搜匹配消息
2. 按 session 分组
3. 抽出相关 transcript
4. 再让便宜模型做 focused summary
5. 返回的是回忆摘要，而不是原始 transcript

这说明 Hermes 不只是“能检索历史”，而是：

> **会先检索，再重构，再把过去经验以当前真正能用的形式拿回来。**

### 3. Skill 负责把成功做法固定成 procedural memory
`tools/skill_manager_tool.py` 直接区分了两类东西：

- memory = broad / declarative
- skills = narrow / actionable

这意味着 Hermes 在明确回答另一个问题：

- 事实要怎么记
- 做法要怎么学

很多 agent 会停在前者。Hermes 进一步做了后者。

### 4. Compression / caching 负责让“越积越多”不会变成“越积越重”
`agent/context_compressor.py` 的算法说明也很直白：

1. 先裁剪旧 tool output
2. 保护 head
3. 保护 tail
4. 总结 middle turns
5. 多次 compaction 时做迭代式 summary update

这套设计说明 Hermes 的目标不是“把所有历史都塞进 prompt”，而是：

> **把历史压成还够继续工作的样子。**

所以 Hermes 的差异，不是“四个 feature 并列摆着”，而是这四层在回答同一个更大的问题：

> **系统怎样在长期运行中越来越会利用自己的过去。**

---

## 三、`run_agent.py` 之所以要放到最后看，就是因为它不是入口问题，而是汇合问题

如果第一次读 Hermes 就先扑进 `run_agent.py`，你当然会看到很多关键结构：

- tool calling loop
- iteration budget
- prompt building
- context compression 接入点
- memory manager 接入点
- trajectory 保存

但问题在于，这样读很容易把 Hermes 理解成：

> “一个功能很多的主循环”

而错过真正重要的问题：

> **这些 memory / recall / skill / compression，到底为什么会一起出现在这里？**

`run_agent.py` 顶部的导入本身就已经说明了这件事：

- `build_memory_context_block`
- `ContextCompressor`
- `apply_anthropic_cache_control`
- `save_trajectory`

这些并不是一些零散增强，而是整套“经验处理系统”重新接回主体 loop 的入口。

也就是说，`run_agent.py` 在这一组文章里更适合被理解成：

> **四条自我进化闭环的汇合点**

而不是入口篇本身。

这也是为什么这组导读不应该从 `run_agent.py` 起手。

---

## 四、这一组为什么比 CLI、gateway、tool calling 更值得先读

这不是说 CLI、gateway、tool calling 不重要。

而是说，如果你想从 Hermes 里学最值钱、最不像通用 agent 模板的部分，那优先级应该先放在这里：

- 记住什么
- 想起什么
- 学会什么
- 怎样不被自己的历史拖死

原因很简单。

### 1. CLI / gateway 是暴露面
它们回答的是：

- Hermes 怎样被人和平台使用
- 怎样被 CLI、Telegram、Feishu、Discord 之类接住

### 2. Tool calling 是执行面
它回答的是：

- 模型怎样发起调用
- 系统怎样把调用落成真实动作

### 3. 但 memory / recall / skill / compression 回答的是“成长面”
它们回答的是更上层的问题：

- 为什么这次做过的事，下次能更顺
- 为什么用户不用一遍遍重复偏好
- 为什么过去解决过的问题，系统还能想起来
- 为什么成功方法不会随着会话结束就蒸发

如果你只学执行面，你学到的是“它怎么工作”。
如果你先学成长面，你学到的是：

> **它为什么会越跑越像一个会积累自己的系统。**

这正是 Hermes 最值得读的地方。

---

## 五、这篇最后要先替整组立住什么

这篇作为入口，不应该急着把每条机制展开到细节。

它真正要先留下来的，是下面这几个判断：

### 判断 1：Hermes 的“自我进化”不是模型在线学习，而是系统级成长
也就是：

- 记忆系统
- 会话回忆
- 技能沉淀
- 上下文自优化

这些能力怎样让下一次运行直接受益。

### 判断 2：这四条线不是并列 feature，而是一套闭环
它们分别回答：

- 记住稳定事实
- 调回过去经验
- 沉淀成功做法
- 控制长期运行成本

### 判断 3：Hermes 真正有辨识度的地方，不是“它也有一个 agent loop”
而是：

> **它怎样把自己的过去，变成未来还能继续使用的能力。**

---

## 收口：为什么这不是“又一个 agent 项目”

如果把 Hermes 只看成一个 agent 框架，你当然也能读到很多东西：

- conversation loop
- tools
- CLI
- gateway
- scheduler
- multi-platform support

但那样读，Hermes 最值钱的部分反而会被淹掉。

因为 Hermes 真正厉害的地方，不只是“能跑很多任务”，而是它已经在明确处理一个更高层的问题：

> **一套 agent 系统怎样在长期运行中不断沉淀、回收、压缩并复用自己的经验。**

这才是 Hermes 和很多“能跑起来的 agent”真正拉开差距的地方。

所以如果这篇只留一句话，最值得留下的是：

> **Hermes 最独特的地方，不在于它有记忆，而在于它把记忆、回忆、技能和上下文优化接成了一套会持续积累自己的系统。**

---

## 卷内导航

- 这是本组起点，建议先顺着往下读。
- 回到本组入口：[本组写作卡片](./README-writing-cards-v1.md)
- 下一篇建议进入：[《MEMORY.md / USER.md / frozen snapshot 为什么是 Hermes 记忆系统的核心设计》](./README-writing-cards-v1.md)
