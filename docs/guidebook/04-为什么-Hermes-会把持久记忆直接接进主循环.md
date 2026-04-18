---
title: 为什么 Hermes 会把持久记忆直接接进主循环
date: 2026-04-17
status: draft
---

# 为什么 Hermes 会把持久记忆直接接进主循环

## 先回答这一篇最该先回答的问题

上一篇已经把 Hermes 的静态材料层拆开了。现在问题会自然推进到下一层：

> 既然 Hermes 已经把经验落成了 `MEMORY.md`、`USER.md`、`state.db`、skills 和 trajectory 文件，那为什么它要把持久记忆直接接进主循环，而不是把 memory 做成一个附属功能？

这是 Hermes 和记忆型 agent 最容易看起来“差不多”、但实际上差得很大的地方。

很多系统谈 memory，默认做法往往是这样的：

- 有一层外置记忆库
- 需要时查一下
- 查到结果再拼回当前上下文
- 没查的时候，主循环照常跑

这种做法当然能工作，但 memory 更像一个外挂检索层。它不是主循环的地基，只是主循环需要时临时调用的一项能力。

Hermes 走的不是这条路。

它的判断更激进一些：

> **有些东西不该等“需要时再查”，而应该在每次会话一开始就稳定站在主循环脚下。**

这也是为什么 `MEMORY.md`、`USER.md` 和 frozen snapshot 不是一个普通的 memory 实现细节，而是 Hermes 记忆系统最核心的设计。

如果把这篇先压成一句话，可以先立这个判断：

> **Hermes 要的是“持续积累”，但不愿意为了实时刷新记忆而破坏当前会话稳定性；`MEMORY.md / USER.md + frozen snapshot` 正是在解决这个矛盾。**

---

## 一、Hermes 先回答的不是“怎么查记忆”，而是“什么必须每轮都带着”

讨论记忆系统时，最容易先想到检索。

但 Hermes 在 `tools/memory_tool.py` 里先做的不是检索设计，而是记忆分层。

文件开头就把两套 store 说清楚了：

- `MEMORY.md`：agent 的个人笔记、环境事实、项目约定、tool quirks、学到的东西
- `USER.md`：对用户的了解，包括偏好、沟通风格、预期、工作习惯

这个拆法看起来简单，实际上已经决定了 Hermes 的记忆观。

它不是把所有“长期记忆”混成一个大桶，而是先区分两种性质完全不同的东西：

### 1. `MEMORY.md` 处理的是系统以后还要继续携带的环境事实

这一层更偏：

- 当前环境的稳定事实
- 项目里的长期约定
- 工具行为的 quirks
- 做事时已经确认过、以后不该再反复试错的经验

也就是说，它回答的是：

> **这个 agent 以后继续工作时，哪些背景事实应该一直带着。**

### 2. `USER.md` 处理的是系统以后还要继续记住的“这个用户是谁”

这一层更偏：

- 用户偏好
- 沟通风格
- 工作习惯
- 厌恶点与常见纠正

也就是说，它回答的是：

> **以后再和这个人协作时，系统应该以什么方式对待他。**

这两层之所以要分开，是因为它们虽然都叫“长期记忆”，但服务对象不同：

- `MEMORY.md` 更像系统对环境与工作世界的长期理解
- `USER.md` 更像系统对协作对象的长期理解

这也是 Hermes 不愿意把记忆做成一团混合上下文的原因。它一开始就在压边界：

> **记住世界，和记住这个用户，不是同一件事。**

---

## 二、真正值钱的设计不只是双文件，而是 frozen snapshot

如果只看到 `MEMORY.md` 和 `USER.md` 两个文件，你还不能完全看清 Hermes 的设计锋利在哪里。

真正关键的是 `tools/memory_tool.py` 开头那几句注释：

- 两套 store 会在 session start 时以 frozen snapshot 形式注入 system prompt
- mid-session writes 会立刻写盘，保证 durable
- 但不会修改当前 system prompt
- snapshot 会在下一次 session start 时刷新

这套设计非常值钱，因为它直接处理了一个很多 memory 系统都会撞到的矛盾：

> **记忆要不要实时更新到当前会话里？**

如果回答“要”，你会得到一个更实时、但更不稳定的 system prompt。

如果回答“不要”，你又会担心：那我这一轮刚写进去的记忆，岂不是白写了？

Hermes 的答案不是二选一，而是拆成两层状态：

### 1. live state：记忆实时写盘，马上 durable

`MemoryStore` 维护着 `memory_entries` 与 `user_entries` 这组 live state。

这组状态会被 tool call 改写，并且立刻持久化到磁盘。

也就是说，只要本轮写入成功：

- `MEMORY.md` / `USER.md` 已经真的更新了
- 下次会话一定能看到
- 不是“暂时放在内存里，等会儿再说”

### 2. frozen snapshot：当前 session 的 system prompt 不跟着抖动

同一个 `MemoryStore` 还维护着另一份状态：

- `_system_prompt_snapshot`

这份 snapshot 在 `load_from_disk()` 时捕获，随后被 `format_for_system_prompt()` 拿来注入 system prompt。

而 `format_for_system_prompt()` 的注释说得很明确：

- 返回的是 load time 捕获的 snapshot
- 不是 live state
- mid-session writes 不影响它
- 这样可以保持整场会话的 prefix cache 稳定

这意味着 Hermes 在这里压住的是一个非常工程化的判断：

> **记忆可以实时落盘，但当前会话的 system prompt 不能跟着实时抖。**

这不是保守，而是清醒。

因为一旦 system prompt 随每次 memory 写入变化，当前 session 的前缀就不再稳定。对长会话来说，这会直接影响：

- prefix cache
- prompt 稳定性
- 会话内行为一致性
- 成本与延迟

Hermes 宁愿接受一件事：

- 这轮新写进去的记忆，不会立刻反灌当前 system prompt

也要保住另一件更重要的事：

- **当前会话始终站在一份稳定的记忆底座上运行**

这就是 frozen snapshot 的价值。

---

## 三、所以 Hermes 的 memory 不是“实时改脑子”，而是“稳定积累，下轮带上”

理解 frozen snapshot 之后，Hermes 的 memory 逻辑就会一下子清楚很多。

它要的不是“模型在本轮中途被新记忆即时改写”，而是：

1. 会话开始时，先带上一份稳定的长期记忆底座
2. 会话进行中，如果发现新的稳定事实，立刻写盘
3. 当前轮不回改 system prompt
4. 下一次会话启动时，再把最新状态整体带上

这套机制的气质其实非常 Hermes：

- 它相信持续积累
- 但不牺牲当前回合的稳定性
- 它允许经验不断沉淀
- 但不把主循环搞成一块不断抖动的地板

所以如果一定要压一句最核心的判断，可以这样说：

> **Hermes 的 memory 不是“边跑边改脑子”，而是“边跑边落盘，下轮整体带上”。**

这比很多“实时记忆注入”的表面聪明做法更稳，也更适合长期运行。

---

## 四、这也是为什么持久记忆会被直接接进 system prompt，而不是做成 query-based recall

这一点在 `agent/builtin_memory_provider.py` 里表现得更清楚。

文件头先把边界说死了：

- `BuiltinMemoryProvider` 只是把 `MEMORY.md` / `USER.md` 包成 `MemoryProvider`
- 它永远是第一个 provider
- 不能被禁用、不能被移除
- 真正的存储逻辑不在这里，而在 `tools/memory_tool.py`

这已经说明，Hermes 对 builtin memory 的定位不是一个普通插件，而是：

> **无论外部 memory 生态怎么扩展，内建持久记忆都必须先站住。**

更关键的是它的三个方法：

### 1. `system_prompt_block()`：builtin memory 的主入口就是 system prompt

`BuiltinMemoryProvider.system_prompt_block()` 的注释非常清楚：

- 它返回 `MEMORY.md` 和 `USER.md` 内容
- 使用的是 load time 捕获的 frozen snapshot
- 目的就是让 system prompt 在整个 session 内保持稳定

这说明 builtin memory 的第一角色不是查询结果，而是：

> **每轮推理都默认携带的背景层。**

### 2. `prefetch()`：builtin memory 不做 query-based recall

`BuiltinMemoryProvider.prefetch()` 直接返回空字符串，并明确写着：

- Built-in memory doesn't do query-based recall
- it's injected via system_prompt_block

这句话其实特别关键。

它说明 Hermes 明确地区分了两种东西：

- **持久记忆**：默认注入，不等查
- **按需回忆**：看 query，再决定召回什么

前者是 memory；后者更像 recall。

Hermes 不愿意把这两者混成一套“统一检索系统”，因为它们根本解决的不是同一个问题。

### 3. `sync_turn()`：builtin memory 也不自动把每轮对话整段吸进去

`BuiltinMemoryProvider.sync_turn()` 的注释也非常直接：

- Built-in memory doesn't auto-sync turns
- writes happen via the memory tool

这又钉死了一层边界。

也就是说，Hermes 的 memory 不是“对话一结束自动吸收本轮所有内容”，而是：

> **只有真正值得长期保存的稳定事实，才会通过 memory tool 进入持久记忆。**

它不要 transcript-style accumulation，它要 curated memory。

---

## 五、`run_agent.py` 真正做的事，是把持久记忆放进主循环的地基层

如果再回头看 `run_agent.py`，这个判断会更清楚。

`_build_system_prompt()` 那段注释已经把 system prompt 的层次顺序排出来了：

1. agent identity
2. user / gateway system prompt
3. persistent memory（frozen snapshot）
4. skills guidance
5. context files
6. current date & time
7. platform-specific hint

关键不在于“memory 也在 system prompt 里”，而在于：

> **persistent memory 被放在了一个非常靠前、非常基础的位置。**

它不是某轮需要时再拼进来的补充材料，也不是 tool result 返回后才参与的后置上下文。

它在正式进入主循环之前，就已经成为循环地板的一部分。

`_build_system_prompt()` 里也确实这样做了：

- 如果启用了 memory，就拿 `self._memory_store.format_for_system_prompt("memory")`
- 如果启用了 user profile，就拿 `format_for_system_prompt("user")`
- 然后把这些 block 直接 append 到 system prompt

这相当于在架构上回答了一件事：

> **Hermes 认为长期记忆不是“需要时查一下”的边角能力，而是 agent 每轮都该带着的稳定背景。**

这也是“memory 被直接接进主循环”最实在的含义。

不是因为它调用得更频繁，而是因为它被放进了更靠前的层。

---

## 六、MemoryManager 的作用，不是替代 builtin memory，而是给它搭总线

很多人读到 `agent/memory_manager.py` 时，会误以为 Hermes 的 memory 逻辑主体已经搬到 manager 里了。

其实没有。

`memory_manager.py` 开头写得很清楚：

- 它负责编排 built-in memory provider
- 外加最多一个 external provider
- built-in provider 永远先注册
- 只允许一个 non-builtin external provider

这里的关键词不是“memory logic”，而是 **orchestrates**。

也就是说，`MemoryManager` 的职责不是重写 `MEMORY.md` / `USER.md` 这一套，而是把内建记忆与外部 provider 摆进同一条总线上。

### 1. builtin provider 永远先站住

`MemoryManager.add_provider()` 明确允许 builtin 永远存在，而第二个 external provider 会被拒绝。

这说明 Hermes 的态度非常明确：

- 外部 memory 生态可以接进来
- 但不能把内建持久记忆挤掉
- 也不能并行挂多个 external backend，让系统边界越来越乱

这是很强的控制欲，但也是很值钱的控制欲。

因为 memory 一旦同时挂多个外部后端，系统很快就会遇到：

- schema 膨胀
- 行为冲突
- “到底谁才是长期记忆真相源”这类边界问题

Hermes 在 manager 这一层直接提前把这些风险压掉了。

### 2. builtin memory 与 external provider 分工并不一样

`MemoryManager` 提供了几条统一接口：

- `build_system_prompt()`
- `prefetch_all()`
- `queue_prefetch_all()`
- `sync_all()`

但 builtin memory 和 external provider 走的并不是同一条路。

对于 builtin memory：

- 主职责是 `system_prompt_block()`
- 不做 query recall
- 不自动 sync turn

对于 external provider：

- 更可能参与 `prefetch`
- 更可能参与 turn sync
- 更像可插拔的扩展记忆通路

这说明 Hermes 并没有把 builtin memory 降格成“external provider 的一个特例”。

恰恰相反：

> **builtin memory 是地基；external provider 是加挂在地基旁边的扩展能力。**

### 3. `build_memory_context_block()` 也反向证明了这层边界

`memory_manager.py` 里还有个很重要的辅助函数：`build_memory_context_block()`。

它会把 recall 到的内容包进一个 fenced block，并加上系统注释：

- 这是 recalled memory context
- 不是新的 user input
- 只能当信息背景使用

这个函数最有意思的地方在于，它其实反向证明了 builtin memory 的位置更底层。

因为需要加 fenced block 的，是 query-based recalled context；而 `MEMORY.md` / `USER.md` 不是这样注入的。

这两类材料从进场方式上就不同：

- 持久记忆：直接进入 system prompt 基础层
- recalled context：作为额外背景，在 API call 时临时包进去

这正是 memory 与 recall 的边界。

---

## 七、所以 Hermes 真正想保住的，是“长期记忆稳定注入”，不是“长期记忆随时乱跳”

把 `memory_tool.py`、`builtin_memory_provider.py`、`memory_manager.py` 连起来看，Hermes 的判断其实很一致：

### 1. 有些记忆必须稳定存在于每轮之前

例如：

- 这个用户讨厌什么
- 这个环境有什么已知事实
- 这个项目有哪些长期约定
- 哪些工具 quirks 以后不要再踩

这类信息如果还要等“需要时再查”，就太晚了。

它们本来就应该是每轮思考的地板，而不是思考开始后才临时补上的材料。

### 2. 但这层地板不能随着会话中途写入不断重建

否则 system prompt 会不断变化。

而一旦变化，代价就不只是“实现复杂一点”，而是会影响：

- prompt cache
- 会话稳定性
- 运行成本
- 长会话的一致性

所以 Hermes 宁愿把实时性让给“落盘”，也不把实时性强加给“当前 prompt”。

### 3. 这让 memory 真正变成了长期能力，而不是回合内技巧

很多系统的 memory 看起来很聪明，但本质上更像：

- 当前轮的补充检索
- 当前回答的辅助材料
- 一种会用会不用的 sidecar 能力

Hermes 的 builtin memory 则更像：

> **系统长期自我保持的一部分。**

这就是为什么它要被直接接进主循环。

不是因为它功能最多，而是因为它负责的是最不应该缺席的背景层。

---

## 八、最后收一句：Hermes 把 memory 放进主循环，不是为了“查得快”，而是为了“站得稳”

如果把这篇最后只留一句话，我会留这句：

> **Hermes 把持久记忆直接接进主循环，不是为了把 memory 做成更频繁的检索能力，而是为了让长期事实以稳定、可持续、可缓存的方式成为每轮运行的背景地板。`MEMORY.md / USER.md + frozen snapshot` 的核心价值，不在于实时改写系统，而在于让系统能够长期、稳定地带着已经确认过的事实继续工作。**

这也是下一篇最自然要继续追的问题：

> Hermes 既然已经把长期事实和持久记忆接稳了，那为什么还需要 `state.db` 和 `session_search_tool.py` 这一整套会话回忆系统？

---

## 系列内继续阅读

- 上一篇：`03-Hermes-到底把什么存下来了-从静态文件层看它怎样为自我进化准备长期材料.md`
- 回到阅读入口：`2026-04-16-Hermes-自我进化阅读路线图-v1.md`
- 如果你想回看这组文章为什么这样排：`2026-04-17-Hermes-特色与不同点系列规划-v1.md`
- 下一篇：`05-session-recall-为什么不是历史回放-而是按需重构过去经验.md`
