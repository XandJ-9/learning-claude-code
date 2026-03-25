# Claude Code 核心概念全解析

## 前言

		前面聊了架构和工程实践，今天咱们来扒一扒 Claude Code 的"四大金刚"：Memory、Skills、MCP 和 Hooks。这些概念官方文档讲得挺详细，但说实话，看完还是容易晕 🤯

所以今天用大白话给你拆解一遍，让你真正理解这些东西到底有啥用、怎么用、啥时候用。

---

## Memory：Claude 的"记事本"

> **官方文档**：[Memory - Claude Code Docs](https://code.claude.com/docs/en/memory)

Claude Code 每次会话都从一个干净的上下文窗口开始，那它怎么记住你之前告诉它的事儿呢？答案就是两套记忆系统：

```
┌─────────────────────────────────────────────────────────┐
│  Memory 系统                                            │
├─────────────────────────────────────────────────────────┤
│  CLAUDE.md    → 你写指令，Claude 遵循                   │
│  Auto Memory  → Claude 自己写笔记，积累经验             │
└─────────────────────────────────────────────────────────┘
```

### CLAUDE.md vs Auto Memory

| 特性 | CLAUDE.md | Auto Memory |
|------|-----------|-------------|
| **谁写的** | 你 | Claude 自己 |
| **存啥** | 指令和规则 | 学习到的模式 |
| **范围** | 项目/用户/组织 | 每个 worktree |
| **加载时机** | 每次会话 | 每次会话（前 200 行） |
| **适合存啥** | 编码规范、工作流 | 构建命令、调试心得 |

简单说：**CLAUDE.md 是你告诉 Claude 怎么干，Auto Memory 是 Claude 自己总结经验教训** 📝

### CLAUDE.md 放哪儿？

```
┌─────────────────────────────────────────────────────────┐
│  Managed Policy（组织级）                                │
│  macOS: /Library/Application Support/ClaudeCode/         │
│  Linux: /etc/claude-code/                                │
│  Windows: C:\Program Files\ClaudeCode\                   │
├─────────────────────────────────────────────────────────┤
│  Project（项目级）     ./CLAUDE.md 或 ./.claude/CLAUDE.md │
├─────────────────────────────────────────────────────────┤
│  User（用户级）       ~/.claude/CLAUDE.md                 │
└─────────────────────────────────────────────────────────┘
```

优先级：Managed > Project > User

### CLAUDE.md 写点啥好？

**✅ 应该写：**
- 怎么 build、test、run（最核心！）
- 关键目录结构和模块边界
- 代码风格和命名约定
- 那些不明显的环境坑
- 绝对不能干的事（NEVER 列表）
- 压缩时必须保留的信息

**❌ 别写：**
- 大段背景介绍
- 完整 API 文档
- 空泛原则，比如"写高质量代码"
- Claude 读读代码就能推断出来的显然信息

### .claude/rules/：按需加载的规则

当项目变大了，CLAUDE.md 写太长不好维护。这时候可以用 `.claude/rules/` 目录拆分规则：

```
.claude/
├── CLAUDE.md           # 主指令
└── rules/
    ├── code-style.md   # 代码风格
    ├── testing.md      # 测试约定
    └── security.md     # 安全要求
```

**更厉害的是路径特定规则**：

```yaml
---
paths:
  - "src/api/**/*.ts"
---

# API 开发规则
- 所有 API 端点必须包含输入验证
- 使用标准的错误响应格式
- 包含 OpenAPI 文档注释
```

这样规则只会在处理匹配的文件时才加载，省空间又精准 🎯

### Auto Memory：Claude 的学习笔记

Auto Memory 让 Claude 在每次会话中自动积累知识，不用你操心。它会记啥：

- 构建命令："哦，这个项目用 `pnpm build` 不用 `npm build`"
- 调试心得："上次遇到这个报错是因为 Redis 没启动"
- 架构笔记："认证模块在 `src/auth/`，别去 `src/user/` 找"
- 代码风格偏好："这个项目喜欢 2 空格缩进"

**存储位置**：
```
~/.claude/projects/<project>/memory/
├── MEMORY.md          # 索引，每次会话加载前 200 行
├── debugging.md       # 调试笔记
├── api-conventions.md # API 设计决策
└── ...                # 其他主题文件
```

**管理 Auto Memory**：

```bash
# 在会话里运行
/memory

# 可以：
# - 查看 CLAUDE.md 和 rules 加载情况
# - 切换 auto memory 开关
# - 打开 memory 文件夹
```

### Compact 指令：压缩时的保命符

当上下文快满了，Claude 会自动压缩。但默认算法可能会把重要决策也给删了 🙈

在 CLAUDE.md 里写上 Compact Instructions，告诉 Claude 啥不能扔：

```markdown
## Compact Instructions
When compressing, preserve in priority order:
1. Architecture decisions (NEVER summarize)
2. Modified files and their key changes
3. Current verification status (pass/fail)
4. Open TODOs and rollback notes
5. Tool outputs (can delete, keep pass/fail only)
```

---

## Skills：给 Claude 加技能包

> **官方文档**：[Skills - Claude Code Docs](https://code.claude.com/docs/en/skills)

Skills 是按需加载的知识和工作流。简单说就是：**你写一个 SKILL.md，Claude 看到相关场景就会用**。

### 一个 Skill 的样子

```
.claude/skills/
└── my-skill/
    ├── SKILL.md           # 主文件（必需）
    ├── reference.md       # 详细文档（按需加载）
    ├── examples.md        # 示例（按需加载）
    └── scripts/
        └── helper.sh      # 脚本
```

### SKILL.md 的结构

```yaml
---
name: explain-code
description: 用图表和类比解释代码如何工作
---

# 代码解释指南

解释代码时，总是包括：
1. **类比**：把代码比作日常事物
2. **画图**：用 ASCII 艺术展示流程
3. **走代码**：逐步解释发生了什么
4. **坑点**：常见错误是什么

保持对话风格。复杂概念用多个类比。
```

### 谁能调用 Skill？

```yaml
# 默认：你手动调用 + Claude 自动判断
---
name: review
description: 代码审查
---

# 只有你能手动调用
---
name: deploy
description: 部署到生产环境
disable-model-invocation: true
---

# 只有 Claude 能用（背景知识）
---
name: legacy-context
description: 老系统上下文
user-invocable: false
---
```

### Skill 的三种类型

```
┌──────────────────┬────────────────────────────────────────┐
│  检查清单型       │  质量门禁：发布前跑一遍，确保不漏项      │
├──────────────────┼────────────────────────────────────────┤
│  工作流型         │  标准化操作：显式调用 + 内置回滚步骤    │
├──────────────────┼────────────────────────────────────────┤
│  领域专家型       │  封装决策框架：按固定路径收集证据        │
└──────────────────┴────────────────────────────────────────┘
```

### 内置 Skills

Claude Code 自带几个实用的 Skills：

| Skill | 用途 |
|-------|------|
| `/simplify` | 审查代码的复用性、质量和效率，然后修复 |
| `/batch <instruction>` | 大规模并行修改代码库 |
| `/debug [description]` | 调试当前会话的日志 |
| `/loop [interval] <prompt>` | 定期运行某个任务 |
| `/claude-api` | 加载 Claude API 和 Agent SDK 文档 |

---

## MCP：连接外部工具的"万能插头"

> **官方文档**：[MCP - Claude Code Docs](https://code.claude.com/docs/en/mcp)

MCP（Model Context Protocol）是个开源标准，让 Claude 能连接各种外部工具和数据源。

### MCP 能干啥？

```
┌─────────────────────────────────────────────────────────┐
│  MCP 服务器 ← Claude Code                               │
├─────────────────────────────────────────────────────────┤
│  GitHub    → PR 管理、代码审查                           │
│  JIRA      → Issue 跟踪                                   │
│  Notion    → 文档协作                                    │
│  Slack     → 消息通知                                    │
│  Sentry    → 错误监控                                    │
│  PostgreSQL → 数据库查询                                 │
│  ...       → 任何 MCP 服务器                             │
└─────────────────────────────────────────────────────────┘
```

### 安装 MCP 服务器

**HTTP 服务器**（推荐，用于云服务）：
```bash
claude mcp add --transport http notion https://mcp.notion.com/mcp
```

**Stdio 服务器**（本地进程）：
```bash
claude mcp add --transport stdio --env API_KEY=xxx airtable \
  -- npx -y airtable-mcp-server
```

### MCP 的三种作用域

```
┌──────────────────┬────────────────────────────────────────┐
│  Local           │  仅当前项目，不共享（默认）              │
├──────────────────┼────────────────────────────────────────┤
│  Project         │  团队共享，存入 .mcp.json                │
├──────────────────┼────────────────────────────────────────┤
│  User            │  跨项目可用，个人工具                    │
└──────────────────┴────────────────────────────────────────┘
```

### MCP 工具命名

MCP 工具的命名规则：`mcp__<server>__<tool>`

```
mcp__memory__create_entities      # Memory 服务器的创建实体工具
mcp__filesystem__read_file        # Filesystem 服务器的读文件工具
mcp__github__search_repositories  # GitHub 服务器的搜索工具
```

可以用正则匹配：`mcp__memory__.*` 匹配所有 Memory 服务器的工具。

---

## Hooks：自动化的"守门员"

> **官方文档**：[Hooks - Claude Code Docs](https://code.claude.com/docs/en/hooks)

Hooks 是在特定时间点自动执行的脚本或 HTTP 请求。简单说就是：**某些事情发生时，自动跑一段代码**。

### Hook 生命周期

```
┌─────────────────────────────────────────────────────────┐
│  SessionStart → UserPromptSubmit → PreToolUse          │
│       ↓              ↓                ↓                 │
│  Tool 执行 → PostToolUse → Stop → SessionEnd          │
└─────────────────────────────────────────────────────────┘
```

### 常用 Hook 事件

| 事件 | 什么时候触发 | 能做啥 |
|------|-------------|--------|
| `SessionStart` | 会话开始或恢复 | 加载上下文、设置环境变量 |
| `PreToolUse` | 工具调用前 | 阻止危险命令 |
| `PostToolUse` | 工具调用后 | 自动格式化、运行测试 |
| `UserPromptSubmit` | 提交提示前 | 验证或添加上下文 |
| `Stop` | Claude 完成响应后 | 检查任务是否真的完成 |
| `SessionEnd` | 会话结束时 | 清理、记录日志 |

### Hook 配置示例

**阻止危险命令**：
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/block-rm.sh"
          }
        ]
      }
    ]
  }
}
```

```bash
#!/bin/bash
# block-rm.sh
COMMAND=$(jq -r '.tool_input.command')

if echo "$COMMAND" | grep -q 'rm -rf'; then
  jq -n '{
    hookSpecificOutput: {
      hookEventName: "PreToolUse",
      permissionDecision: "deny",
      permissionDecisionReason: "危险命令被 Hook 阻止"
    }
  }'
else
  exit 0  # 允许
fi
```

**编辑后自动格式化**：
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npm run format",
            "async": true
          }
        ]
      }
    ]
  }
}
```

### Hook 的三种类型

| 类型 | 说明 | 适用场景 |
|------|------|----------|
| `command` | 运行 shell 命令 | 本地脚本、格式化、测试 |
| `http` | 发送 HTTP 请求 | 远程验证、Webhook 通知 |
| `prompt` | 让 LLM 评估决策 | 复杂判断、质量检查 |
| `agent` | 启动 subagent 验证 | 需要读文件、搜索代码 |

### Hook 匹配器

`matcher` 字段用正则表达式过滤何时触发：

```
"Bash"              # 仅 Bash 工具
"Edit|Write"         # Edit 或 Write 工具
"mcp__memory__.*"    # Memory 服务器的所有工具
```

---

## 其他核心概念

### Subagents：子代理人

Subagent 是从主对话派生的独立 Claude 实例，有自己的上下文和工具限制。

**核心价值是隔离**，不是并行：

```
┌─────────────────┐         ┌─────────────────┐
│  主线程          │         │  Subagent       │
│  干净的上下文    │  派遣   │  独立上下文     │
│  只拿摘要        │ ←────── │  大量探索       │
└─────────────────┘  结果   └─────────────────┘
```

**配置要点**：
- `tools/限制工具`：别给和主线程一样宽的权限
- `model`：探索用 Haiku/Sonnet，审查用 Opus
- `maxTurns`：防止跑飞
- `isolation: worktree`：需要动文件时隔离文件系统

### Tools：内置工具

Claude Code 有一堆内置工具：

| 工具 | 用途 |
|------|------|
| `Read` | 读文件 |
| `Edit` | 替换文件中的字符串 |
| `Write` | 创建或覆盖文件 |
| `Grep` | 搜索文件内容 |
| `Glob` | 查找匹配模式的文件 |
| `Bash` | 运行 shell 命令 |
| `Agent` | 启动 subagent |
| `WebFetch` | 获取网页内容 |
| `WebSearch` | 搜索网络 |

---

## 实战建议

### 什么时候用啥？

```
┌─────────────────────────────────────────────────────────┐
│  想给 Claude 持久指令？    → CLAUDE.md                  │
│  想让 Claude 自己学习？    → Auto Memory                │
│  想打包可复用的工作流？    → Skills                     │
│  想连接外部工具或数据？    → MCP 服务器                  │
│  想自动执行某些操作？      → Hooks                      │
│  想隔离执行探索任务？      → Subagents                  │
└─────────────────────────────────────────────────────────┘
```

### 踩坑经验

1. **CLAUDE.md 写太长** → 上下文先被自己污染了，拆到 Skills 或 rules
2. **MCP 接太多** → 工具定义吃掉 10%+ 上下文，按需添加
3. **Hooks 到处写** → 难以调试，集中管理在 `.claude/hooks/`
4. **Subagents 滥用** → 状态漂移，只在需要隔离时用
5. **Auto Memory 不看** → Claude 记了些啥你都不知道，定期 `/memory` 检查

---

## 总结

Claude Code 的核心概念环环相扣：

```
Memory     → 持久化指令和经验
Skills     → 可复用的工作流和知识
MCP        → 连接外部工具和数据
Hooks      → 自动化的守门员
Subagents  → 隔离执行环境
Tools      → 内置操作能力
```

理解了这些，你就能让 Claude Code 真正为你服务，而不是被它牵着鼻子走 🎯

---

**官方文档来源**：
- [Memory - Claude Code Docs](https://code.claude.com/docs/en/memory)
- [Skills - Claude Code Docs](https://code.claude.com/docs/en/skills)
- [MCP - Claude Code Docs](https://code.claude.com/docs/en/mcp)
- [Hooks - Claude Code Docs](https://code.claude.com/docs/en/hooks)
