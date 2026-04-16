# Hermes 源码导读

> **这不是一份泛化的 Agent 架构八股，而是围绕 Hermes 最值得学习的部分展开：它怎样通过记忆、会话回忆、技能沉淀与上下文自压缩，让自己在长期运行中持续变强。**

## 这套导读先回答什么

Hermes 最值得读的，不是“它也有一个 agent loop”，而是下面这条更有价值的问题链：

1. **Hermes 为什么不是普通“带工具的 agent”？**
2. **它怎样把用户偏好、环境事实与稳定约定沉淀成持久记忆？**
3. **它怎样把过往会话变成可按需召回的经验，而不是无穷变长的聊天历史？**
4. **它怎样把一次次成功方法沉淀成 skill，而不是只记住零散事实？**
5. **它怎样通过 context compression / prompt caching / trajectory，让自己在长期运行里不被历史拖垮？**
6. **这些能力又是怎样重新接回 `run_agent.py` 的主体循环，形成真正的“自我进化系统”？**

## 先不按“通用 agent 结构”读

这次不打算从 CLI、tool calling、gateway 这些“大家差不多都有”的层面起手。

先聚焦 Hermes 最值钱的 4 条线：

- **持久记忆**：`tools/memory_tool.py`、`agent/builtin_memory_provider.py`、`agent/memory_manager.py`
- **会话回忆**：`hermes_state.py`、`tools/session_search_tool.py`
- **技能沉淀**：`tools/skill_manager_tool.py`、`agent/skill_commands.py`、`agent/skill_utils.py`
- **上下文自优化**：`agent/context_compressor.py`、`agent/prompt_caching.py`、`agent/trajectory.py`

## 当前仓库写作计划

### 第一阶段：Hermes 自我进化机制

1. 为什么 Hermes 不是“有记忆的 agent”，而是“能持续积累自己的 agent”
2. MEMORY.md / USER.md / frozen snapshot 为什么是 Hermes 记忆设计的核心
3. session recall 为什么不是历史回放，而是按需重构过去经验
4. 为什么 skill system 才是 Hermes 最像自我进化的地方

### 第二阶段：Hermes 的自我优化机制

5. context compression 怎样让 Hermes 不被自己的历史压垮
6. prompt caching 怎样把长期多轮成本压下来
7. trajectory 在 Hermes 里为什么像经验资产，而不只是日志

### 第三阶段：这些能力怎样接回 agent 主体

8. memory / skill / recall / compression 怎样接回 `run_agent.py`
9. 为什么 Hermes 的成长是 system-level growth，而不是 model-level growth

## 当前入口文档

- [Hermes 自我进化阅读路线图 v1](./guidebook/2026-04-16-Hermes-自我进化阅读路线图-v1.md)

## 一句话总结

> **Hermes 最值得学习的，不是它怎么“像一个 agent 一样工作”，而是它怎么把记忆、回忆、技能和上下文管理做成持续增强自己的系统能力。**
