---
title: 为什么 skill system 才是 Hermes 最像自我进化的地方
date: 2026-04-17
status: draft
---

# 为什么 skill system 才是 Hermes 最像自我进化的地方

## 先回答这篇最关键的问题

前面几篇其实已经把 Hermes 的前两层积累能力压出来了：

- memory 让它能稳定记住事实
- session recall 让它能在需要时想起过去

但如果只到这里，Hermes 仍然更像一个“会记得比较多”的系统，而不一定像一个真正会持续变强的系统。

因为还有一个更关键的问题没有解决：

> **记住事实、想起过去，和真正学会做法，其实不是一回事。**

这正是 skill system 要回答的问题。

如果说 memory 回答的是：

- 什么值得长期记住

session recall 回答的是：

- 什么值得在当前场景下重新想起

那么 skill system 回答的就是：

- **哪些成功做法，值得沉淀成以后可以直接复用的程序化能力**

这也是为什么我会把 skill system 看成 Hermes 最像“自我进化”的地方。

因为很多 agent 到前两层就停住了：

- 会积累事实
- 会搜索历史
- 但不会把“怎么做”也正式沉淀下来

而 Hermes 在 `tools/skill_manager_tool.py` 里把这件事说得非常直接：

> **Skills are the agent's procedural memory.**

这句其实几乎可以看成整篇文章的总开关。

如果把这篇先压成一句话，可以先立这个判断：

> **很多 agent 只会积累事实；Hermes 更进一步，把“怎么做”也沉淀成 procedural memory，而 skill system 正是这套能力的正式入口。**

---

## 一、Hermes 在这里区分的，不是“有没有技能”，而是“事实”和“做法”是不是两种不同资产

很多系统也会提 skills。

但它们经常把 skill 理解成下面几类东西之一：

- 一份提示词模板
- 一篇操作文档
- 一套命令别名
- 一个插件市场入口

这些都不能算错，但都还不够。

Hermes 在 `skill_manager_tool.py` 开头先做的，是一刀把 declarative memory 和 procedural memory 分开。

它说得非常清楚：

- General memory（`MEMORY.md`、`USER.md`）是 broad and declarative
- Skills are narrow and actionable

这组对照很重要。

因为它在回答一个很多 agent 系统没有真正回答的问题：

> **“知道什么”和“会怎么做”，到底是不是同一种长期资产？**

Hermes 的答案是：不是。

### 1. declarative memory 回答的是“有哪些事实应该保留下来”

前面已经讲过，`MEMORY.md` / `USER.md` 更偏：

- 用户偏好
- 环境事实
- 项目约定
- 工具 quirks
- 长期有效的经验判断

这类东西的共同点是：

- 它们是事实
- 它们是背景
- 它们是系统以后该继续带着的认知材料

### 2. procedural memory 回答的是“以后再遇到这类事，该怎么做”

skill system 处理的不是事实，而是做法。

它更偏：

- 一类任务的稳定操作路径
- 成功做法的整理版
- 以后可直接调用的处理流程
- 可以继续修补、继续细化的程序化经验

这类东西和 memory 最大的不同在于，它不是“知道有这回事”，而是：

> **已经把做法组织成可重复执行的形式。**

也就是说：

- memory 让 Hermes 不会忘
- recall 让 Hermes 不会白做
- skill 让 Hermes 不必每次从头重新悟

这一层一出来，Hermes 才真的开始像“持续进化”。

---

## 二、所以 skill 不是“另一种记忆文件”，而是一种可演化的操作资产

如果只把 skill 理解成另一种 markdown 文件，你会低估这层设计。

`skill_manager_tool.py` 从一开头就把它定义成 agent-managed skill creation & editing。

也就是说，Hermes 眼里的 skill 不是静态说明书，而是：

- 可以创建
- 可以更新
- 可以修补
- 可以删除
- 可以继续写 supporting files

这和 memory 的气质完全不一样。

### 1. memory 更像事实仓库

memory 的主要动作是：

- add
- replace
- remove

它解决的是：

- 某条长期事实该不该进来
- 某条事实要不要改写
- 某条事实是不是已经过期

这层逻辑虽然重要，但它本质上仍然是在维护“认知内容”。

### 2. skill 更像方法资产库

而 skill system 的动作集明显更重：

- `create`
- `edit`
- `patch`
- `delete`
- `write_file`
- `remove_file`

这说明 Hermes 对 skill 的预期不是“存一下”，而是：

> **把一类任务的成功做法保存成以后还能继续加工的资产。**

它甚至不是普通意义上的“会用一个 skill”。

它更像：

- 这次做成了
- 不能只留在聊天里
- 要把方法沉淀下来
- 以后还要能继续修、继续扩、继续拆 supporting files

这就是为什么我说，它比普通的 memory 更像“自我进化”。

因为这里沉淀下来的已经不是知识点，而是行动模式。

---

## 三、skill 的目录结构本身，就说明它不是随手写篇文档

这一点在 `skill_manager_tool.py` 和 `tools/skills_tool.py` 里都很清楚。

Hermes 不是把 skill 当作一个平面文本条目，而是给了它完整目录结构：

```text
~/.hermes/skills/
├── my-skill/
│   ├── SKILL.md
│   ├── references/
│   ├── templates/
│   ├── scripts/
│   └── assets/
```

这层结构很重要，因为它直接说明：

> **一个 skill 可以是一个小型操作知识包，而不是一段孤零零的提示词。**

### 1. `SKILL.md` 只是主入口，不是全部内容

`tools/skills_tool.py` 文件头把 skills 的 progressive disclosure 架构写得很清楚：

- metadata 先通过 `skills_list` 暴露
- full instructions 通过 `skill_view` 加载
- linked files 再按需读取

这意味着 skill 不是一个一次性塞满上下文的大包，而是一个：

- 先看描述
- 再按需展开
- 再按需读取 supporting files

的结构化知识对象。

这比“写一篇长文档让模型自己消化”强很多。

### 2. supporting files 说明它承载的是操作世界，不只是文字世界

`references/`、`templates/`、`scripts/`、`assets/` 这些目录的存在，本身就是证据。

因为一套成熟做法往往不只是一段说明：

- 可能有参考材料
- 可能有模板
- 可能有辅助脚本
- 可能有图、样例、素材

也就是说，Hermes 并不是在说：

> 我把会的东西写成一段提示词就行了。

它在说：

> **真正可复用的方法，应该有自己的操作结构。**

这已经非常接近“把经验资产化”了。

---

## 四、为什么 create / patch / edit / write_file 这些动作必须做成正式工具

这一点尤其重要。

如果 Hermes 只是支持“新增一个 skill”，那还可以理解成它有一套知识沉淀机制。

但它不只支持 create。

它还明确支持：

- `patch`
- `edit`
- `write_file`
- `remove_file`

这组动作放在一起，意思就完全不一样了。

### 1. `create` 说明经验可以首次沉淀

这个最容易理解。

一次复杂任务做通之后，系统可以把成功方法沉淀成一个 skill。

这一步对应的是：

- 从“做成了一次”
- 变成“以后有了一个可复用起点”

### 2. `patch` 说明经验不是一锤子买卖，而是可以渐进修正

这一步非常像真正的学习过程。

现实里最有价值的方法，很少第一次就完美。

更常见的是：

- 跑过一次
- 发现有坑
- 补一条注意事项
- 修一个错误命令
- 添一个缺失步骤

`patch` 的存在意味着 Hermes 允许技能像代码一样被增量修补。

这和“存一篇教程”完全不是一个层次。

### 3. `edit` 说明经验可以整体重写

有些时候不是补一行就够，而是：

- 原 skill 结构已经旧了
- 场景边界变了
- 一整套流程都该改写

这时候 `edit` 就不是修修补补，而是正式重构。

这说明 skill 在 Hermes 里不是死档案，而是活资产。

### 4. `write_file` 说明操作经验不只是一段文本

这一步特别能说明 Hermes 的工程感。

因为很多真正可复用的方法最后都会长成：

- 主说明文档
- 参考说明
- 配置模板
- 验证脚本
- 样例资产

`write_file` 的存在，就是在承认这件事：

> **经验沉淀到一定程度，最终会超出“正文”本身。**

也正因为如此，我会说 Hermes 的 skill system 不是“有 skills 功能”，而是：

> **它允许成功做法长成一套可以维护的操作资产。**

---

## 五、validation / security scan / size limit 这些细节，说明 Hermes 把 skill 当成正式生产资产

如果 skill 只是聊天时顺手生成的一段文字，其实没必要这么认真。

但 `skill_manager_tool.py` 里明显不是这个态度。

它有一整套很工程化的约束：

- name / category 校验
- frontmatter 校验
- size limit
- path validation
- allowed subdirs 限制
- security scan

这些细节放在一起，其实只说明一件事：

> **Hermes 认为 skill 不是随便存点经验，而是在正式写一种以后还会被加载、被执行、被复用的生产资产。**

### 1. frontmatter 校验说明它不是随手写文

它要求 skill 要有：

- `name`
- `description`
- 正确 frontmatter 结构
- frontmatter 后面必须有正文

这说明 Hermes 不接受“一坨自由散文式经验”。

它要求这份做法必须长成一个可识别、可管理、可扫描的对象。

### 2. path validation 和 allowed subdirs 说明它不是任意文件写入器

`ALLOWED_SUBDIRS = {"references", "templates", "scripts", "assets"}`

再加上 file path 校验，说明 Hermes 很明确地在控制 skill 资产的边界。

它允许你写 supporting files，但不允许你借 skill system 在目录树里随便乱长。

### 3. security scan 说明 skill 会真正进入后续执行世界

这点尤其重要。

`skill_manager_tool.py` 会对 agent-created skills 跑 security scan，并复用和社区 hub 安装技能同一套审查思路。

这说明 skill 在 Hermes 里不是“写着玩”的东西。

它未来是可能真的影响执行行为的。

也正因为如此，它必须被当作带风险、带价值的正式资产看待。

换句话说：

- 如果 memory 更像长期认知材料
- 那么 skill 更像长期操作材料

后者显然更接近“学会做事”。

---

## 六、`skills_list` / `skill_view` / `skill_commands` 这套结构，说明 skill 会重新回流主循环

skill system 真正重要的地方，还不只是“能存下来”。

更重要的是：

> **存下来的 skill 不是归档结束，而是以后还会重新回到系统运行里。**

这一点在 `tools/skills_tool.py` 和 `agent/skill_commands.py` 里看得很清楚。

### 1. `skills_list` / `skill_view` 说明 skill 是可被再次加载的

`tools/skills_tool.py` 一开始就把两层工具写清了：

- `skills_list`：列 metadata
- `skill_view`：按需加载完整 skill 或 linked file

这意味着 skill 不是“写完就埋起来”，而是：

- 以后还能发现
- 以后还能展开
- 以后还能按需取正文和 supporting files

也就是说，skill 已经不只是沉淀结果，而是未来推理和执行的可用材料。

### 2. `agent/skill_commands.py` 说明它甚至可以长成 slash-command 式入口

`skill_commands.py` 会扫描 `~/.hermes/skills/`，把 skill 变成 `/command -> skill info` 的映射。

这说明 Hermes 不是把 skill 当成背景仓库，而是允许它继续长成一种运行入口。

更有意思的是，它不是把 skill 粗暴塞进 system prompt，而是以按需触发、按需注入的方式参与后续会话。

这说明 Hermes 很清楚 skill 的位置：

- 它不是长期固定背景
- 它也不是一次性结果
- 它是随场景重新被调入的程序化能力

这一步一成立，skill 就不再只是“积累经验”，而是：

> **把过去学会的做法重新转回当前能力。**

这已经非常接近“系统级学习”。

---

## 七、现在可以把 memory / recall / skill 三层重新压清了

写到这里，其实 Hermes 的前三层积累结构已经很清楚了。

### 1. memory：记住事实

它解决的是：

- 用户是谁
- 环境是什么
- 哪些长期事实要一直带着

### 2. session recall：想起过去

它解决的是：

- 以前有没有处理过类似问题
- 哪些旧会话和当前任务最相关
- 该把哪些过去经验调回来

### 3. skill：学会做法

它解决的是：

- 哪些成功方法值得正式沉淀
- 以后再遇到同类任务时，系统能不能不从头试
- 这些做法能不能继续修、继续扩、继续复用

如果把这三层连起来，Hermes 的“自我进化”才真正开始站稳：

- 不是只会多记一点
- 不是只会多搜一点
- 而是会把成功路径沉淀成以后能直接调用的方法资产

这一步一出来，Hermes 和很多“有记忆能力的 agent”就已经明显不在一个层次上了。

因为后者往往停在：

- 知道更多
- 找回更多

而 Hermes 进一步走到：

- **以后可以更会做**

---

## 八、最后收一句：Hermes 最像自我进化的地方，不是会记，而是会把做法留下来

如果把这篇最后只留一句话，我会留这句：

> **Hermes 最像“自我进化”的地方，不在于它会记住多少事实，也不只在于它能回忆多少旧会话，而在于它把成功做法正式沉淀成 procedural memory，并允许这些做法继续被创建、修补、重写、扩展和重新加载。到了这一层，系统积累下来的已经不是认知材料，而是未来还能继续使用的行动能力。**

如果后面继续往下写，最自然进入的就不是 memory / recall / skill 这一组本身了，而是另一组问题：

> 既然 Hermes 已经能记住事实、想起过去、沉淀做法，那它又是怎么在长会话里继续整理这些经验，而不是被历史反过来拖垮的？

---

## 系列内继续阅读

- 上一篇：`05-session-recall-为什么不是历史回放-而是按需重构过去经验.md`
- 回到阅读入口：`2026-04-16-Hermes-自我进化阅读路线图-v1.md`
- 如果你想回看这组文章为什么这样排：`2026-04-17-Hermes-特色与不同点系列规划-v1.md`
- 下一篇：`07-为什么-Hermes-不只是压缩上下文-而是在整理自己的过去.md`
