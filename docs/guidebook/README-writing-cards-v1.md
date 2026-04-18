---
title: Hermes 自我进化机制 README-writing-cards v1
date: 2026-04-16
status: draft
---

# Hermes 自我进化机制 README-writing-cards v1

> 本组不是泛化介绍 Hermes 的全部架构，而是只抓它最值得学习、也最能和一般 agent 拉开差距的部分：**Hermes 怎样把记忆、回忆、技能与上下文自优化组织成持续增强自己的系统能力。**

> 2026-04-17 更新：这一组当前已经从最初的写作卡片阶段推进成 01-09 连续文章草稿。下面这份文件继续保留，主要作为原始写作意图与问题链说明；实际阅读顺序应以现有正文文件为准。

## 这几层文件现在怎么分工

- **仓库层总入口**：`README.md`
- **阅读入口**：`2026-04-16-Hermes-自我进化阅读路线图-v1.md`
- **系列规划**：`2026-04-17-Hermes-特色与不同点系列规划-v1.md`
- **写作底稿 / 问题链**：当前这份 `README-writing-cards-v1.md`

也就是说，如果你现在是第一次进入这一组，建议先看仓库层总入口 `README.md`，再进入阅读路线图；如果你要看这一组为什么这样排、每篇各自负责什么，再看系列规划；如果你要追原始写作意图与问题链，再回到这份 README-writing-cards。

---

## 本组总问题

如果把 Hermes 只看成一个“带工具的 agent”，你会先看到：

- `run_agent.py` 的 conversation loop
- `model_tools.py` 的工具发现与调度
- `cli.py` 的交互壳
- `gateway/` 的多平台接入

这些都重要，但不是本组的主问题。

本组真正要回答的是：

> **Hermes 怎样把一次次对话里的经验，沉淀成下一次运行可以直接使用的长期能力？**

也就是说，Hermes 的“变强”主要不是 model-level learning，而是 system-level growth：

- 把环境事实和用户偏好沉淀成持久记忆
- 把过往会话变成按需召回的经验
- 把成功方法沉淀成可复用 skill
- 把长历史压缩成还能继续工作的上下文

---

## 当前正文顺序（2026-04-17）

当前建议直接按下面顺序读正文：

1. `01-为什么-Hermes-不是-有记忆的-agent-而是-能持续积累自己的-agent.md`
2. `02-从主循环看-Hermes-与-coding-agent-有什么不同.md`
3. `03-Hermes-到底把什么存下来了-从静态文件层看它怎样为自我进化准备长期材料.md`
4. `04-为什么-Hermes-会把持久记忆直接接进主循环.md`
5. `05-session-recall-为什么不是历史回放-而是按需重构过去经验.md`
6. `06-为什么-skill-system-才是-Hermes-最像自我进化的地方.md`
7. `07-为什么-Hermes-不只是压缩上下文-而是在整理自己的过去.md`
8. `08-为什么-Hermes-的主循环收口后还没结束-background-review-才是经验沉淀的关键转折.md`
9. `09-Hermes-怎样把-memory-recall-skills-compression-接成真正的-self-evolution-loop.md`

也就是说，这份 README-writing-cards 里的“文章 01 / 02 / 03 ...”更适合继续被理解为**早期策划卡片编号**，而不是当前仓库里的最终正文编号。

---

## 文章 01

### 暂定标题
**为什么 Hermes 不是“有记忆的 agent”，而是“能持续积累自己的 agent”**

### 核心问题
Hermes 和一般 agent 真正的差别，到底只是“多了记忆功能”，还是它已经把记忆、会话回忆、技能沉淀与上下文优化组织成一套会持续增强自己的系统？

### 一句话主张
> **Hermes 的独特之处不在于“它也有记忆”，而在于它把记忆、回忆、技能与压缩接成了一个持续积累自己的系统闭环。**

### 最重要任务
1. 先把“自我进化”这个词去神秘化，明确它不是模型权重在线学习。
2. 给出 Hermes 自我进化的 4 条主线：持久记忆、会话回忆、技能沉淀、上下文自优化。
3. 建立全组阅读视角：后面几篇不是并列 feature，而是在拆这套闭环。

### 建议源码锚点
- `run_agent.py`
- `tools/memory_tool.py`
- `tools/session_search_tool.py`
- `tools/skill_manager_tool.py`
- `agent/context_compressor.py`

### 建议写作主线
1. 先纠正误解：Hermes 不是“多记一点”的 agent。
2. 再说明它的成长不是 model-level，而是 system-level。
3. 再把四条闭环拉平，给出低分辨率总图。
4. 最后说明本组后续每篇分别拆哪一段闭环。

### 一定要讲清的点
- “自我进化”在 Hermes 里到底指什么
- 为什么 memory / session recall / skill / compression 不能压成一层
- 为什么这一组比 CLI / gateway / tool calling 更值得先读

### 不要讲什么
- 不要在这篇里深讲 `run_agent.py` 的 while loop 细节
- 不要展开具体数据库 schema 或 tool schema
- 不要把这篇写成“全仓库导览”

### 和前后篇关系
- 本篇是整组入口，负责建立阅读视角
- 下一篇进入最基础、也是最稳的一层：持久记忆设计

---

## 文章 02

### 暂定标题
**Hermes 到底把什么存下来了：从静态文件层看它怎样为自我进化准备长期材料**

### 核心问题
我们已经从动态主循环看到了 Hermes 会处理经验。但如果这些经验最后没有稳定落到磁盘上，那“自我进化”就只是运行时幻觉。Hermes 到底把什么真的存下来了？

### 一句话主张
> **Hermes 的自我进化不是抽象口号，而是由一套明确的静态材料层支撑的：`MEMORY.md`、`USER.md`、`state.db`、skills 目录、trajectory 文件和 `config.yaml` 一起构成了它的经验落地面。**

### 最重要任务
1. 先把 Hermes 的静态文件层整体列清，不让读者只盯主循环。
2. 解释不同静态材料分别在保存什么：稳定事实、会话历史、程序化做法、运行配置、经验轨迹。
3. 压清这些静态文件为什么不是附属存档，而是后续 recall / skill / compression 能成立的物质基础。

### 建议源码锚点
- `tools/memory_tool.py`
- `hermes_state.py`
- `tools/skill_manager_tool.py`
- `agent/trajectory.py`
- `gateway/run.py`
- `hermes_cli/config.py`

### 建议写作主线
1. 从“动态主循环之外，Hermes 到底留下了什么”切入。
2. 分层讲：`MEMORY.md / USER.md`、`state.db`、skills 目录、trajectory 文件、`config.yaml`。
3. 说明这些静态材料分别服务哪一类经验：事实、会话、方法、轨迹、配置。
4. 最后把它们重新压回一句：Hermes 的自我进化有真实落点，不是纯运行时行为。

### 一定要讲清的点
- `MEMORY.md` / `USER.md` 保存的是 curated long-term facts
- `state.db` 保存的是 session transcript / metadata / FTS5 recall 基础
- `~/.hermes/skills/` 保存的是 procedural memory
- trajectory 文件与 `state.db` 不是一回事
- `config.yaml` 保存的是运行策略，不是经验正文，但它决定经验系统如何被使用

### 不要讲什么
- 不要把这篇写成“列目录”文
- 不要提前把所有 memory/recall/skill 内部机制展开完
- 不要把 `config.yaml` 误写成 memory 系统本体

### 和前后篇关系
- 本篇承接第二篇：既然 Hermes 的主循环在处理经验，那这些经验最后存到哪里
- 下一篇再进入持久记忆系统本身：为什么 memory 被直接接进主循环

---

## 文章 03

### 暂定标题
**MEMORY.md / USER.md / frozen snapshot 为什么是 Hermes 记忆系统的核心设计**

### 核心问题
Hermes 为什么要把记忆拆成 `MEMORY.md` 与 `USER.md` 两套？为什么中途写入 memory 会立即落盘，却不刷新当前 system prompt？

### 一句话主张
> **Hermes 要的是“持续积累”，但不愿意为了实时刷新记忆而破坏当前会话稳定性；`MEMORY.md / USER.md + frozen snapshot` 正是在解决这个矛盾。**

### 最重要任务
1. 解释 `MEMORY.md` 与 `USER.md` 分层各自负责什么。
2. 解释 memory tool 的 durable write 与 frozen snapshot 为什么要并存。
3. 解释 builtin memory provider 与 memory manager 的边界。

### 建议源码锚点
- `tools/memory_tool.py`
- `agent/builtin_memory_provider.py`
- `agent/memory_manager.py`

### 建议写作主线
1. 从 `tools/memory_tool.py` 开始，先说明两类持久记忆和写盘逻辑。
2. 重点解释注释里最值钱的一句：mid-session writes update files, but do NOT change system prompt。
3. 再看 `builtin_memory_provider.py`，说明它是薄适配层，不负责记忆逻辑本体。
4. 最后看 `memory_manager.py`，说明 Hermes 怎样把 builtin memory 与 external provider 编排在一起。

### 一定要讲清的点
- `MEMORY.md` 更偏环境事实 / 项目约定 / tool quirks
- `USER.md` 更偏用户偏好 / 沟通风格 / 习惯
- frozen snapshot 为什么能保住 prefix cache 与会话稳定性
- 为什么 Hermes 只允许一个 external provider

### 不要讲什么
- 不要在这篇里讲 session transcript 检索
- 不要提前展开 skills 或 context compression
- 不要把所有 memory plugin 都扫一遍

### 和前后篇关系
- 本篇讲“长期记住什么、怎样稳定注入”
- 下一篇转向另一层：不是固定记忆，而是过去会话怎样在需要时被召回

---

## 文章 04

### 暂定标题
**session recall 为什么不是历史回放，而是按需重构过去经验**

### 核心问题
Hermes 既然已经有 MEMORY.md / USER.md 了，为什么还要把完整 session transcript 存进 SQLite，并在 `session_search_tool.py` 里走“FTS5 检索 + LLM 总结”的两步流程？

### 一句话主张
> **Hermes 不只会“记住事实”，它还会在需要时把过去会话重构成当前真正需要的经验；session recall 解决的是情境回忆，不是固定记忆。**

### 最重要任务
1. 解释 `hermes_state.py` 里 session store 的职责。
2. 解释为什么底层是 SQLite + FTS5，而不是简单 JSONL。
3. 解释 `session_search_tool.py` 为什么不回放原文，而是 focused summary。
4. 压清“持久记忆”和“会话回忆”的边界。

### 建议源码锚点
- `hermes_state.py`
- `tools/session_search_tool.py`

### 建议写作主线
1. 从 `hermes_state.py` 的 schema 和设计注释入手，解释 session transcript 持久化的范围。
2. 再看 FTS5 与 session grouping，说明它不是普通日志堆积。
3. 再看 `session_search_tool.py` 的 summarization flow，说明 recall 是“检索后重构”。
4. 最后压成一句：memory 负责稳定事实，session recall 负责按需调回过去经验。

### 一定要讲清的点
- `SessionDB` 为什么是 Hermes 长期运行能力的一部分
- FTS5 检索和 LLM 总结为什么要分两步
- 为什么 raw transcript 不能直接塞回主模型上下文
- `_HIDDEN_SESSION_SOURCES` 这类设计如何说明 recall 不是“什么都回忆”

### 不要讲什么
- 不要把这篇写成数据库实现细节文
- 不要提前进入 trajectory 或 compression
- 不要展开 gateway session store 的所有细节

### 和前后篇关系
- 本篇从“固定记忆”转向“按需回忆”
- 下一篇进入最值钱的一层：procedural memory / skill system

---

## 文章 05

### 暂定标题
**为什么 skill system 才是 Hermes 最像自我进化的地方**

### 核心问题
如果说 memory 让 Hermes 会记住，session recall 让 Hermes 会想起过去，那 Hermes 真正最像“自我进化”的地方，为什么是 `skill_manager_tool.py` 代表的 skill system？

### 一句话主张
> **很多 agent 只会积累事实；Hermes 更进一步，把“怎么做”也沉淀成 procedural memory，而 skill system 正是这套能力的正式入口。**

### 最重要任务
1. 解释 `skill_manager_tool.py` 里 declarative memory vs procedural memory 的区分。
2. 解释 skill 为什么不是“另一种记忆文件”。
3. 解释 create / patch / edit / delete 这些动作为什么都要做成正式工具。
4. 说明 Hermes 怎样把成功做法沉淀成之后可直接复用的程序化能力。

### 建议源码锚点
- `tools/skill_manager_tool.py`
- `agent/skill_commands.py`
- `agent/skill_utils.py`
- `tools/skills_tool.py`

### 建议写作主线
1. 从 `skill_manager_tool.py` 开头那句最值钱的定义切入：skills are procedural memory。
2. 解释 skill 的目录布局、frontmatter 校验、supporting files 这些设计为什么说明它不是随手写文档。
3. 解释 skill create / patch / edit / write_file 背后的工程意图：把经验沉淀成可演化的操作资产。
4. 最后把 memory / recall / skill 三者重新压清：记住事实、想起过去、学会做法。

### 一定要讲清的点
- procedural memory 这个概念在 Hermes 里如何落地
- 为什么 skills 是 narrow / actionable，而 memory 是 broad / declarative
- 为什么 skill 还有 security scan、size limit、path validation
- 这层为什么比“单纯 skills hub”更接近自我进化

### 不要讲什么
- 不要把这篇写成 skills hub 使用教程
- 不要在这篇里展开 website / optional-skills 生态
- 不要提早进入 context compression 细节

### 和前后篇关系
- 本篇是第一阶段的收束篇
- 写完这篇，Hermes 的“记住事实 → 想起过去 → 沉淀方法”三层就已经立住
- 后续下一组自然进入：context compression / prompt caching / trajectory 的自优化机制

---

## 本组最后要留下的一句话

> **Hermes 最有辨识度的地方，不是它也有一个 agent loop，而是它把记忆、会话回忆、技能沉淀接成了一套会持续积累自己的系统。**
