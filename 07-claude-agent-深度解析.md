# Claude Agent 深度解析

## 前言

很多人第一次看到 Claude Code 里的 agent，会有一种很自然的误解：

- "哦，这不就是多开几个 Claude 吗？"
- "哦，这不就是把 prompt 包一层壳吗？"
- "哦，这不就是高级版聊天分身术吗？"

说实话，这么理解不能算全错，但确实有点低估它了。

Claude Agent 真正有意思的地方，不是"多一个 AI"，而是**把不同类型的工作拆到不同上下文里处理**。你可以把它理解成：主对话像总指挥，agent 像被临时拉来的专项同事。查资料的去查资料，跑日志的去跑日志，审代码的去审代码，最后别把一大坨过程垃圾全倒回主会话里，只把结论带回来。

这件事看起来只是"分工"，实际上它同时解决了上下文污染、权限控制、复用配置、成本控制这几个老大难问题。

所以今天这篇，咱们就把 Claude Agent 这套机制拆开讲明白：**它是什么、为什么有用、怎么创建、怎么调用、什么时候该用、什么时候别硬上。**

---

## 一句话理解 Claude Agent

先看最短版本：

```
┌─────────────────────────────────────────────────────────┐
│  主会话（你当前正在聊的 Claude）                         │
│                                                         │
│  负责：理解目标、做决策、整合结果                         │
│                         │                               │
│                         │ 派活                           │
│                         ↓                               │
│  Agent（专项助手）                                      │
│  - 独立上下文                                             │
│  - 独立系统提示词                                          │
│  - 独立工具权限                                            │
│  - 可指定模型                                              │
│                         │                               │
│                         │ 只返回摘要/结果                  │
│                         ↓                               │
│  主会话继续推进                                            │
└─────────────────────────────────────────────────────────┘
```

核心点就四个字：**隔离干活**。

这意味着 agent 不是简单"帮你多说几句话"，而是：

- 在自己的上下文窗口里做事
- 按自己的规则和工具边界行动
- 把冗长过程留在自己那边
- 只把真正有用的结果交回主线程

如果你之前经常觉得 Claude 用着用着就"脑子变乱"，agent 往往就是那个很对症的解法。

---

## 为什么 Agent 很重要？

### 1. 它先解决的不是能力问题，而是上下文污染问题

很多任务不是做不了，而是**做完以后把主会话搞脏了**。

比如这些典型场景：

- 扫一整个代码库找认证逻辑
- 跑完整测试然后筛失败项
- 读一堆日志只为了找到一条报错根因
- 查官方文档然后提炼几个关键结论

如果这些内容都直接塞回主会话，上下文很快就会变成这样：

```
┌─────────────────────────────────────────────────────────┐
│  真正需要保留的内容                                       │
│  - 任务目标                                               │
│  - 架构决策                                               │
│  - 修改方案                                               │
│  - 风险和结论                                             │
├─────────────────────────────────────────────────────────┤
│  不太想永久保留但特别占地方的内容                           │
│  - 搜索结果 200 行                                         │
│  - 测试日志 1000 行                                        │
│  - 文档摘录 5000 tokens                                    │
│  - 临时探索过程                                             │
└─────────────────────────────────────────────────────────┘
```

Agent 的作用，就是把下面那一层垃圾桶单独拿走。

### 2. 它让"专项限制"终于变得可操作

有些任务你就是不想让它乱来。

比如：

- 研究代码结构时，只允许读，不允许写
- 跑数据库查询时，只允许 `SELECT`
- 做代码审查时，不要顺手改文件
- 跑浏览器自动化时，只给浏览器相关 MCP

这就是 agent 的第二个价值：**限制工具和权限**。

你可以给某个 agent 明确规定：

- 能用哪些工具
- 禁止哪些工具
- 用什么 permission mode
- 是否挂特定 MCP server

这就不是"希望它听话"，而是"结构上不让它乱来"。

### 3. 它适合复用，不适合一次次复制 prompt

如果你经常做这类事：

- 每次都写一段代码审查 prompt
- 每次都要强调“优先关注安全和正确性”
- 每次都要告诉它“先查上下文，再给结论”

那其实就该把它做成 agent 了。

因为 agent 本质上就是一个**可复用的专项角色定义**：

- 描述什么时候调用它
- 指定它的系统提示词
- 限制它的工具和权限
- 选定它的模型

写一次，后面一直用。比每次口头重新培训省心太多。

### 4. 它还能省钱

这一点非常现实。

不是所有任务都值得上大模型。

官方内置的 Explore agent 就是典型思路：**用更快更便宜的 Haiku 去做搜索和探索**。这种“查资料型”工作，本来就不需要每次都让主模型亲自下场。

所以 agent 还有个非常实用的价值：**把贵的模型留给决策，把便宜的模型留给体力活。**

---

## Claude Code 内置了哪些 Agent？

官方文档里提到，Claude Code 自带几类内置 agent。最常见的是下面这几个：

| Agent | 模型 | 工具特点 | 典型用途 |
|------|------|------|------|
| **Explore** | Haiku | 只读工具 | 搜代码、查结构、做代码库探索 |
| **Plan** | 继承主会话模型 | 只读工具 | 在 Plan Mode 里做调研，辅助产出计划 |
| **general-purpose** | 继承主会话模型 | 全工具 | 复杂多步骤任务，既要查也要改 |

再加上一些辅助型 agent，比如：

| Agent | 什么时候会被调用 |
|------|------|
| **statusline-setup** | 运行 `/statusline` 配状态栏时 |
| **Claude Code Guide** | 你询问 Claude Code 自身功能时 |

这里最关键的认知是：

> **内置 agent 不是摆设，它们会被 Claude 根据任务自动委派。**

也就是说，你不一定每次都要手动点名。只要任务描述足够像某个 agent 的职责，Claude 就可能主动把活派过去。

---

## `/agents` 命令到底是干嘛的？

这个命令可以理解成 Claude Code 的 **Agent 管理后台**。

官方推荐的创建和管理方式，就是用 `/agents`。

它大概做两件事：

### Running 标签页：看现在有哪些 agent 正在干活

- 当前活跃的 agent
- 可以打开它们
- 也可以停止它们

### Library 标签页：管理 agent 仓库

- 看所有可用 agent（内置、用户级、项目级、插件级）
- 创建新的 agent
- 编辑已有 agent 的配置
- 删除自定义 agent
- 查看重名时谁覆盖了谁

如果你不想进交互界面，命令行也能直接列出所有 agent：

```bash
claude agents
```

这个命令很适合拿来做两件事：

1. 确认当前到底有哪些 agent 能用
2. 排查“我明明定义了 agent，怎么没生效”这种覆盖问题

---

## Agent 有哪些作用域？谁会覆盖谁？

Claude Code 的 agent 不是只能写在一个地方。官方给了多层来源，优先级从高到低大致是这样：

| 位置 | 作用域 | 优先级 |
|------|------|------|
| managed settings | 组织级 | 1 |
| `--agents` CLI 参数 | 当前会话 | 2 |
| `.claude/agents/` | 当前项目 | 3 |
| `~/.claude/agents/` | 当前用户的所有项目 | 4 |
| 插件里的 `agents/` | 插件启用范围 | 5 |

这里有两个特别实用的结论：

### 项目级 agent：适合团队共享

放在：

```bash
.claude/agents/
```

适合那种和仓库强绑定的角色，比如：

- `repo-reviewer`
- `migration-runner`
- `frontend-a11y-checker`

这类 agent 可以直接提交到版本控制，团队一起用。

### 用户级 agent：适合个人工作习惯

放在：

```bash
~/.claude/agents/
```

适合那种你在所有项目里都爱用的角色，比如：

- `my-code-reviewer`
- `debug-first-agent`
- `doc-summarizer`

一句话总结：

```
项目相关 → .claude/agents/
个人偏好 → ~/.claude/agents/
临时试验 → --agents
```

---

## 一个自定义 Agent 文件长什么样？

Agent 本质上就是一个 Markdown 文件：**上面是 YAML frontmatter，下面是 system prompt**。

最小可用示例：

```markdown
---
name: code-reviewer
description: Expert code reviewer. Use proactively after code changes.
tools: Read, Glob, Grep
model: sonnet
---

You are a senior code reviewer. Focus on correctness, security, and
maintainability. Give specific, actionable feedback.
```

这个结构看起来简单，但威力其实挺大：

- `name` 决定 agent 的标识
- `description` 决定 Claude 什么时候会想起它
- frontmatter 里的配置决定它能干什么
- Markdown 正文决定它怎么思考、怎么输出

注意一个很重要的细节：

> **Subagent 拿到的是它自己的 system prompt，不会自动继承默认的 Claude Code system prompt。**

它会有一些基础环境信息，比如工作目录，但它不是把主线程整套脑回路原封不动复制一份。

*所以你写 agent prompt 时，别偷懒，职责、边界、输出格式最好都写清楚。*

---

## 最常用的 frontmatter 字段，别一上来全背，先抓重点

官方支持的字段很多，但实际最常用的，先记住下面这些就够了：

| 字段 | 干嘛的 |
|------|------|
| `name` | agent 名称，只能小写和连字符 |
| `description` | 告诉 Claude 什么时候该调它 |
| `tools` | 明确允许哪些工具 |
| `disallowedTools` | 从可用工具里剔除一部分 |
| `model` | 指定 `sonnet` / `opus` / `haiku` / `inherit` |
| `permissionMode` | 控制权限模式 |
| `skills` | 启动时预加载 skill 内容 |
| `mcpServers` | 给它挂特定 MCP server |
| `hooks` | 配生命周期 hook |
| `memory` | 开启持久化记忆 |
| `background` | 默认后台运行 |
| `isolation` | 用临时 git worktree 隔离执行 |
| `maxTurns` | 限制 agent 最多跑多少轮 |
| `initialPrompt` | 当它作为主 session agent 时的初始提示 |

如果你第一次上手，我建议记这个优先级：

```
第一层先会用：
description / tools / model

第二层进阶：
permissionMode / skills / mcpServers

第三层高手区：
hooks / memory / isolation / background
```

别一开始就把所有开关全拧上，不然很容易把自己绕晕。

---

## `description` 是灵魂，不是摆设

很多人会认真写 prompt，却把 `description` 写得特别敷衍，比如：

```yaml
description: Helps with code
```

这种描述基本等于没写。

因为 Claude 是靠这个字段来判断：

- 什么时候该委派给它
- 这活到底是不是它负责
- 需不需要主动调用它

更靠谱的写法应该像这样：

```yaml
description: Expert code reviewer. Use proactively after code changes.
```

这句话短，但信息量很足：

- 它是干什么的：code reviewer
- 什么时候触发：after code changes
- 要不要主动用：use proactively

这就是“让 Claude 知道什么时候该把这位同事叫进来”的关键提示。

---

## 工具控制：Agent 最有价值的工程能力之一

官方支持两种主要控制方式：

### 1. 用 `tools` 做白名单

```yaml
tools: Read, Grep, Glob, Bash
```

意思是：只给这些工具，别的都别碰。

这适合特别强调边界的 agent，比如只读研究员。

### 2. 用 `disallowedTools` 做黑名单

```yaml
disallowedTools: Write, Edit
```

意思是：其他都能继承，但这俩不准用。

这适合那种“总体能力不限制，但有几样绝对不许动”的场景。

比如一个代码审查 agent，通常就很适合禁止写文件。

### 一个很典型的安全研究型 agent

```markdown
---
name: safe-researcher
description: Research agent with restricted capabilities
tools: Read, Grep, Glob, Bash
---

Search the codebase and summarize findings. Do not modify files.
```

这种 agent 就非常适合：

- 查调用链
- 读日志
- 扫配置
- 做文档调研

它能干活，但基本没有搞破坏的机会。

---

## 还可以限制 MCP、权限模式和 Hook

这部分是 agent 从“好用”走向“工程化可控”的关键。

### 1. `mcpServers`：让某个 agent 独享外部能力

比如你想做个浏览器测试 agent，只给它 Playwright：

```markdown
---
name: browser-tester
description: Tests features in a real browser using Playwright
mcpServers:
  - playwright:
      type: stdio
      command: npx
      args: ["-y", "@playwright/mcp@latest"]
---

Use the Playwright tools to navigate, screenshot, and interact with pages.
```

这个设计很妙的一点在于：

> **如果 MCP 只在 agent 里内联定义，它的工具描述不会常驻主会话上下文。**

也就是说，既拿到了能力，又少吃了主线程的上下文空间。

### 2. `permissionMode`：明确它遇到权限时怎么处理

常见模式包括：

- `default`
- `acceptEdits`
- `auto`
- `dontAsk`
- `bypassPermissions`
- `plan`

最实用的理解方式是：

| 模式 | 适合什么场景 |
|------|------|
| `plan` | 只读调研，不要改文件 |
| `default` | 正常人工确认 |
| `acceptEdits` | 文件改动比较可控，想减少确认弹窗 |
| `auto` | 想让分类器代你审一部分权限 |

### 3. `hooks`：在 agent 生命周期里插“工程护栏”

比如你想让某个数据库 agent 只允许只读查询，可以在 `PreToolUse` 里检查 Bash 命令，发现有 `INSERT` / `DELETE` / `DROP` 就直接拦掉。

这个能力特别适合做：

- 命令白名单/黑名单
- 自动 lint
- 自动 cleanup
- 合规校验

一句话总结：

```
tools 决定“能不能用”
permissionMode 决定“怎么批”
hooks 决定“用之前/之后还要不要再审一遍”
```

---

## 持久化记忆：让 Agent 越用越懂你这个项目

这是官方文档里一个很容易被忽略，但非常有潜力的能力：`memory`。

你可以给 agent 开持久化记忆，作用域有三种：

| scope | 位置 | 适合场景 |
|------|------|------|
| `user` | `~/.claude/agent-memory/<agent>/` | 跨项目通用经验 |
| `project` | `.claude/agent-memory/<agent>/` | 项目级知识，适合团队共享 |
| `local` | `.claude/agent-memory-local/<agent>/` | 本地私有，不想进版本控制 |

官方推荐默认优先考虑 `project`，因为它最适合沉淀项目知识。

比如一个 code reviewer agent 可以慢慢记住：

- 这个仓库常见的目录职责
- 某些老问题总出在哪
- 哪套错误处理模式才是团队认可的
- 哪些路径特别容易踩坑

这时候 agent 就不只是“临时工”，而是开始变成“熟悉项目的老同事”了。

当然，前提是你得真的让它维护记忆，而不是开了这个功能之后就放着不管。

---

## 怎么调用 Agent？三种方式够你用

官方给了三种主要调用思路，基本覆盖绝大多数使用场景。

### 1. 自然语言点名

这是最省事的一种：

```text
Use the test-runner subagent to fix failing tests
Have the code-reviewer subagent look at my recent changes
```

你说了 agent 名字，Claude 通常就会理解并尝试委派。

特点是灵活，但最终是否委派，还是 Claude 来判断。

### 2. `@` 提及：强制这次就用它

```text
@"code-reviewer (agent)" look at the auth changes
```

这相当于明确指定：“别猜你该叫谁了，就叫这个。”

特别适合：

- 你已经知道该用哪个 agent
- 不想让 Claude 自己选择
- 做精确控制

### 3. 整个 session 直接以某个 agent 身份运行

```bash
claude --agent code-reviewer
```

或者在 `.claude/settings.json` 里设默认：

```json
{
  "agent": "code-reviewer"
}
```

这种方式的意思是：**整个主线程都切成这个 agent 的人格和约束模式。**

这就不是“叫个同事来帮忙”，而是“今天你自己就以这个专项角色开工”。

适合：

- 固定职责会话
- 长时间只做某一类任务
- 想让默认系统提示词整体被专项角色替换

---

## 前台 Agent 和后台 Agent：差别比你想的更大

Claude Code 里的 agent 可以在前台跑，也可以后台跑。

### 前台运行

- 会阻塞主会话
- 有权限提示会转给你
- 有澄清问题也能正常来问你

这适合那种：

- 任务比较关键
- 过程中可能需要你拍板
- 你愿意盯着它完成

### 后台运行

- 可以并发跑，主会话继续干别的
- 启动前会先把所需权限问清楚
- 一旦开跑，没预批准的权限会自动拒绝
- 如果它中途需要问你问题，调用会失败，但 agent 会继续跑

这就很适合：

- 长日志分析
- 批量测试
- 大范围搜索
- 多个互不依赖的研究任务并行

还有两个很实用的小点：

1. 你可以直接要求 Claude “run this in the background”
2. 正在跑的任务可以用 **Ctrl+B** 丢到后台

---

## 什么时候该用 Agent，什么时候别硬上？

这是最容易踩坑的地方。

Agent 很强，但**不是所有任务都该 agent 化**。

### 适合用 Agent 的场景

| 场景 | 为什么适合 |
|------|------|
| 高输出任务 | 测试日志、搜索结果、文档摘录会污染主上下文 |
| 自包含任务 | 做完后只需要结果摘要 |
| 需要严格权限边界 | 读写分离、只读审查、特定 MCP |
| 可并行研究 | 多个模块互不依赖，可以一起查 |
| 可复用专项角色 | 同一种 prompt 和限制反复出现 |

### 不太适合的场景

| 场景 | 为什么不太适合 |
|------|------|
| 高频来回讨论 | agent 上下文隔了一层，交流成本更高 |
| 很短的小修改 | 启动 agent 的成本可能比任务本身还高 |
| 多阶段强耦合任务 | 计划、实现、验证如果强依赖共享上下文，主线程更顺 |
| 需要持续精细调参 | 你一会儿改目标、一会儿改方向，主会话更直接 |

官方文档里也明确给了一个很重要的判断标准：

> **如果任务产生很多你不打算在主会话里反复引用的过程输出，就很适合丢给 agent。**

这个标准非常实用，建议直接记住。

---

## Agent、Skill、主会话，到底怎么选？

这三个概念经常混。

其实可以这样分：

| 机制 | 核心作用 | 是否隔离上下文 |
|------|------|------|
| **主会话** | 负责总体推进、判断、整合 | 否 |
| **Skill** | 给主会话按需加载知识/工作流 | 否 |
| **Agent** | 把某类工作丢到独立上下文里处理 | 是 |

一句大白话总结：

```
Skill = 给当前大脑装插件
Agent = 再叫一个独立同事来干活
```

所以如果你的目标是：

- “让当前会话懂某套流程” → 更像 Skill
- “把这坨脏活单独扔出去做” → 更像 Agent

这两个别混了。混了就很容易出现两种尴尬：

1. 明明该隔离，结果把技能塞进主上下文，越聊越乱
2. 明明只是加点知识，结果强行起 agent，徒增延迟

---

## 三个特别实用的 Agent 设计模式

### 模式 1：高输出隔离器

适合：

- 跑测试
- 看日志
- 查文档
- 全仓搜索

目标就一句话：**过程很吵，但结论很短。**

示例 prompt：

```text
Use a subagent to run the test suite and report only the failing tests with their error messages
```

### 模式 2：并行研究员

适合多个互不依赖的研究任务同时展开：

```text
Research the authentication, database, and API modules in parallel using separate subagents
```

这类任务里，agent 的价值不只是隔离上下文，还包括**并发提速**。

### 模式 3：链式分工

先查，再改；先审，再修。

```text
Use the code-reviewer subagent to find performance issues, then use the optimizer subagent to fix them
```

这种玩法本质上就是把原来一个大任务拆成明确阶段，然后给每一段配合适的人设。

---

## 进阶能力：worktree 隔离和恢复上下文

### 1. `isolation: worktree`

这个选项很适合做隔离修改。

```yaml
isolation: worktree
```

意思是：给 agent 一个临时 git worktree，在隔离副本里工作。

这个能力的好处特别明显：

- 并行任务互不踩文件
- 试验性修改更安全
- 没改出内容时还能自动清理

对于多分支并行、危险重构、实验性修改，这个配置相当香。

### 2. Resume：继续上次那个 agent，而不是从零再来

官方文档提到，agent 每次调用默认是新实例，但在启用 agent teams 的前提下，可以恢复之前的 agent，延续它已有的上下文。

这个能力更适合复杂长任务，比如：

- 第一轮先审 authentication
- 第二轮接着审 authorization
- 中间不想让它忘掉前面的发现

不过这块属于更进阶的多 agent 协同玩法，日常使用里先把基础的创建、限制和调用玩明白，已经能解决 80% 的问题。

---

## 一份很实用的入门心法

如果你准备真正开始自己写 agent，我建议按这个顺序来：

### 第一步：先做一个只读 agent

比如：

- code-searcher
- doc-summarizer
- repo-reviewer

因为只读 agent 风险最低，最容易看出上下文隔离的价值。

### 第二步：把 `description` 写具体

别写“helps with code”这种空气描述。

至少写清楚：

- 它是谁
- 什么时候该用
- 要不要主动调用

### 第三步：只给必要工具

能用白名单就别一股脑全给。

越小的能力面，越稳定，越不容易出现“叫它调研，结果顺手给你改了 14 个文件”的惊喜。

### 第四步：先解决一个重复问题，再考虑高级配置

先问自己：

> 我到底有没有一类重复出现、值得抽成 agent 的任务？

如果没有，就别为了“感觉高级”硬配一堆 agent。

Agent 是为了解决真实分工问题，不是为了把工具栏堆满。

---

## 最后总结：Claude Agent 的本质，是把“协作结构”做出来

如果只看表面，agent 像是“给 Claude 增加几个角色”。

但从工程视角看，它真正做的事情是：

- **把高噪音任务隔离出去**
- **把能力边界写进配置**
- **把重复角色沉淀成可复用资产**
- **把模型和成本做分层分配**

它不是单纯提高“回答质量”的玄学技巧，而是在改善 Claude Code 的**工作组织方式**。

一句最实在的话收尾：

> **当你发现某类任务总是又吵、又长、又重复，还总把主会话搞乱时，八成就该上 agent 了。**

官方文档链接：

- [Subagents / Agents 官方文档](https://code.claude.com/docs/en/sub-agents#use-the-%2Fagents-command)
