# Hermes 源码导读手册

> 一套围绕 **Hermes 怎样把经验沉淀成长期能力** 重新组织的中文源码导读手册。

这个仓库现在已经不再是零散笔记入口。当前正式阅读入口已经切到 **Hermes 自我进化 guidebook**，目标不是泛泛讲一遍通用 agent 架构，而是把 Hermes 最值得学的那条主线讲清楚：它怎样把记忆、回忆、技能和上下文治理接成一套会持续增强自己的系统。

## 从哪里开始

- **在线阅读**：<https://t0ugh.github.io/hermes-source-reading/>
- **总入口**：[`guidebook/README.md`](./guidebook/README.md)
- **站点首页说明**：[`docs/index.md`](./docs/index.md)

如果你是第一次进入，建议按这个顺序：

1. [Hermes 自我进化 guidebook 总览](./guidebook/README.md)
2. [Hermes 自我进化阅读路线图](./guidebook/2026-04-16-Hermes-自我进化阅读路线图-v1.md)
3. [01 为什么 Hermes 不是“有记忆的 agent”，而是“能持续积累自己的 agent”](./guidebook/01-为什么-Hermes-不是-有记忆的-agent-而是-能持续积累自己的-agent.md)
4. [02 从主循环看，Hermes 与 coding agent 有什么不同](./guidebook/02-从主循环看-Hermes-与-coding-agent-有什么不同.md)
5. [09 Hermes 怎样把 memory、recall、skills、compression 接成真正的 self-evolution loop](./guidebook/09-Hermes-怎样把-memory-recall-skills-compression-接成真正的-self-evolution-loop.md)

如果你想直接顺着正文读，可以按 01–09 连续进入。

## 这套手册在回答什么

这不是一份“按源码目录散逛”的索引，而是在回答一串连续问题：

- Hermes 为什么不是普通“带工具的 agent”
- 它怎样把用户偏好、环境事实与稳定约定沉淀成持久记忆
- 它怎样把过往会话变成可按需召回的经验，而不是无限变长的聊天历史
- 它怎样把一次次成功方法沉淀成 skill，而不是只留下零散事实
- compression / caching / background review 为什么不是附属优化，而是长期运行还能继续变强的必要条件
- 这些能力最后又怎样重新接回 `run_agent.py` 的主体循环，形成真正的 self-evolution loop

一句话说：

> **这 9 篇不是材料分桶，而是一条连续问题链：先看清 Hermes 的不同，再看记忆、回忆、技能、上下文治理，最后收回系统级自我进化。**

## 当前仓库结构

### 正式阅读入口
- `guidebook/`：当前正式主线入口

### 站点发布层
- `docs/`：GitHub Pages / MkDocs 站点入口与导航页
- `mkdocs.yml`：站点配置与导航定义

### 已发布静态站点
- `site/`：当前构建出的静态站点产物

## 当前状态

当前状态可以直接理解为：

- `guidebook/README.md` 已经是默认入口
- 01–09 正文已经形成完整阅读主线
- `2026-04-16-Hermes-自我进化阅读路线图-v1.md` 承担阅读坡度说明
- `2026-04-17-Hermes-特色与不同点系列规划-v1.md` 与 `README-writing-cards-v1.md` 更偏控制层 / 底稿层，不作为主入口
- 后续工作重点更偏向：正文精修、图示补强、导航整理与站点一致性维护

如果你想看更细的入口与站点说明，优先读：

- [`guidebook/README.md`](./guidebook/README.md)
- [`docs/index.md`](./docs/index.md)
- [`mkdocs.yml`](./mkdocs.yml)

## 本地运行

```bash
pip install mkdocs-material pymdown-extensions
mkdocs serve
```

默认站点地址：<http://127.0.0.1:8000>

## GitHub Pages

公开站点地址：

<https://t0ugh.github.io/hermes-source-reading/>
