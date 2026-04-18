---
title: Hermes 源码导读
date: 2026-04-18
---

# Hermes 源码导读

> 一套围绕 **Hermes 怎样把经验沉淀成长期能力** 重新组织的中文源码导读手册。

这里是公开站点首页。当前这套导读不先按“通用 agent 分层”展开，而是直接从 Hermes 最值得学的主线切入：

- 持久记忆怎样被稳定注入
- 会话回忆怎样把过去重构成当前可用经验
- skill system 怎样把做法沉淀成 procedural memory
- compression / caching / background review 怎样让系统在长期运行里继续变轻、变稳、变可复用

## 从哪里开始

### 第一次进入这个项目
建议直接从这里开始：

- [开始阅读｜Hermes 自我进化 guidebook](./guidebook/README.md)
- [Hermes 自我进化阅读路线图](./guidebook/2026-04-16-Hermes-自我进化阅读路线图-v1.md)
- [01 为什么 Hermes 不是“有记忆的 agent”，而是“能持续积累自己的 agent”](./guidebook/01-为什么-Hermes-不是-有记忆的-agent-而是-能持续积累自己的-agent.md)

### 如果你想直接顺着正文读
可以直接按 01–09 的顺序进入：

1. [01 为什么 Hermes 不是“有记忆的 agent”，而是“能持续积累自己的 agent”](./guidebook/01-为什么-Hermes-不是-有记忆的-agent-而是-能持续积累自己的-agent.md)
2. [02 从主循环看，Hermes 与 coding agent 有什么不同](./guidebook/02-从主循环看-Hermes-与-coding-agent-有什么不同.md)
3. [03 Hermes 到底把什么存下来了：从静态文件层看它怎样为自我进化准备长期材料](./guidebook/03-Hermes-到底把什么存下来了-从静态文件层看它怎样为自我进化准备长期材料.md)
4. [04 为什么 Hermes 会把持久记忆直接接进主循环](./guidebook/04-为什么-Hermes-会把持久记忆直接接进主循环.md)
5. [05 session recall 为什么不是历史回放，而是按需重构过去经验](./guidebook/05-session-recall-为什么不是历史回放-而是按需重构过去经验.md)
6. [06 为什么 skill system 才是 Hermes 最像自我进化的地方](./guidebook/06-为什么-skill-system-才是-Hermes-最像自我进化的地方.md)
7. [07 为什么 Hermes 不只是压缩上下文，而是在整理自己的过去](./guidebook/07-为什么-Hermes-不只是压缩上下文-而是在整理自己的过去.md)
8. [08 为什么 Hermes 的主循环收口后还没结束：background review 才是经验沉淀的关键转折](./guidebook/08-为什么-Hermes-的主循环收口后还没结束-background-review-才是经验沉淀的关键转折.md)
9. [09 Hermes 怎样把 memory、recall、skills、compression 接成真正的 self-evolution loop](./guidebook/09-Hermes-怎样把-memory-recall-skills-compression-接成真正的-self-evolution-loop.md)

## 这套导读在回答什么

这套导读不是带你“逛 Hermes 代码目录”，而是在回答一串更关键的问题：

- Hermes 为什么不是普通“带工具的 agent”
- `MEMORY.md / USER.md` 和 frozen snapshot 为什么是长期记忆设计的核心
- `state.db + session_search` 为什么不是聊天记录搜索，而是过去经验的按需重构层
- skill system 为什么比“多一个记忆文件”更接近真正的自我进化
- compression / caching / background review 为什么不是附属优化，而是经验系统继续可工作的条件

## 当前入口说明

- `guidebook/README.md` 是当前正式阅读入口
- `guidebook/2026-04-16-Hermes-自我进化阅读路线图-v1.md` 是主阅读坡度说明
- `guidebook/2026-04-17-Hermes-特色与不同点系列规划-v1.md` 是结构解释层，不进入主导航
- `guidebook/README-writing-cards-v1.md` 是写作底稿层，不进入主导航

## 一句话理解这本书

> 这不是一份泛化的 Agent 架构八股，而是围绕 Hermes 最值得学习的部分展开：它怎样通过记忆、会话回忆、技能沉淀与上下文自压缩，让自己在长期运行中持续变强。