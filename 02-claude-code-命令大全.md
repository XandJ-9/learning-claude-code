# Claude Code 命令大全

## 启动命令

### 基础启动

```bash
# 启动交互式会话
claude

# 使用初始提示启动
claude "explain this project"

# 通过 SDK 查询后退出（不进入交互模式）
claude -p "explain this function"
```

### 管道输入

```bash
# 处理管道内容
cat file.txt | claude -p "summarize this"
cat logs.txt | claude -p "analyze errors"
git diff | claude -p "review this diff"
```

## 会话管理

### 继续会话

```bash
# 继续当前目录中最近的对话
claude -c
claude --continue

# 继续并通过 SDK 发送消息
claude -c -p "Check for type errors"
```

### 恢复会话

```bash
# 交互式选择会话恢复
claude -r
claude --resume

# 按名称恢复会话
claude -r "session-name"
claude --resume "auth-refactor"

# 按 ID 恢复会话
claude -r "abc123-def456"
```

### 会话分支

```bash
# 恢复时创建新会话（不修改原会话）
claude --resume abc123 --fork-session
claude -c --fork-session
```

## 认证命令

```bash
# 登录 Anthropic 账户
claude auth login
claude auth login --email user@example.com
claude auth login --sso

# 登出
claude auth logout

# 查看认证状态（JSON 格式）
claude auth status

# 查看认证状态（人类可读）
claude auth status --text
```

## 更新与版本

```bash
# 更新到最新版本
claude update

# 查看当前版本
claude -v
claude --version
```

## 模型选择

```bash
# 使用 Opus 模型
claude --model opus

# 使用 Sonnet 模型
claude --model sonnet

# 使用 Haiku 模型
claude --model haiku

# 使用完整模型名称
claude --model claude-sonnet-4-6
claude --model claude-opus-4-6
```

## 权限模式

```bash
# 进入计划模式（适合复杂任务）
claude --permission-mode plan

# 跳过所有权限提示（谨慎使用）
claude --dangerously-skip-permissions

# 允许特定工具无需权限确认
claude --allowedTools "Bash(git log *)" "Bash(git diff *)" "Read"

# 禁用特定工具
claude --disallowedTools "Bash(rm *)" "Edit"
```

## 系统提示定制

```bash
# 替换整个系统提示
claude --system-prompt "You are a Python expert"

# 从文件加载系统提示
claude -p --system-prompt-file ./custom-prompt.txt "query"

# 附加到默认提示（推荐）
claude --append-system-prompt "Always use TypeScript"

# 从文件附加提示
claude -p --append-system-prompt-file ./style-rules.txt "query"
```

## 目录与工作区

```bash
# 添加额外的工作目录
claude --add-dir ../apps ../lib

# 在 git worktree 中启动
claude -w feature-branch
claude --worktree feature-auth

# 使用特定会话 ID
claude --session-id "550e8400-e29b-41d4-a716-446655440000"
```

## 集成相关

```bash
# 启用 Chrome 浏览器集成
claude --chrome

# 禁用 Chrome 集成
claude --no-chrome

# 自动连接到 IDE
claude --ide

# 启动 Remote Control 会话
claude remote-control
```

## Agent 与 MCP

```bash
# 指定自定义 agent
claude --agent my-custom-agent

# 动态定义 subagents（JSON 格式）
claude --agents '{
  "reviewer": {
    "description": "Reviews code",
    "prompt": "You are a code reviewer"
  }
}'

# 列出所有已配置的 subagents
claude agents

# 配置 MCP 服务器
claude mcp

# 从文件加载 MCP 配置
claude --mcp-config ./mcp.json

# 仅使用指定 MCP 配置
claude --strict-mcp-config --mcp-config ./mcp.json
```

## 输出格式

```bash
# 打印模式（文本输出）
claude -p "query"

# JSON 输出
claude -p --output-format json "query"

# 流式 JSON 输出
claude -p --output-format stream-json "query"

# 结构化 JSON 输出（符合 Schema）
claude -p --json-schema '{"type":"object","properties":{...}}' "query"
```

## 调试与日志

```bash
# 启用详细日志
claude --verbose

# 启用调试模式（所有类别）
claude --debug

# 启用特定类别调试
claude --debug "api,mcp"

# 排除特定类别
claude --debug "!statsig,!file"
```

## 网络会话

```bash
# 创建网络会话
claude --remote "Fix the login bug"

# 在本地恢复网络会话
claude --teleport

# 恢复特定 PR 关联的会话
claude --from-pr 123
claude --from-pr https://github.com/user/repo/pull/123
```

## 其他选项

```bash
# 限制最大轮数
claude -p --max-turns 3 "query"

# 设置最大预算（美元）
claude -p --max-budget-usd 5.00 "query"

# 禁用会话持久化
claude -p --no-session-persistence "query"

# 设置回退模型
claude -p --fallback-model sonnet "query"

# 禁用所有 skills 和命令
claude --disable-slash-commands

# 运行初始化 hooks
claude --init

# 仅运行初始化 hooks（无交互）
claude --init-only

# 运行维护 hooks
claude --maintenance

# 加载设置文件
claude --settings ./settings.json

# 指定设置源
claude --setting-sources user,project,local

# 加载插件目录
claude --plugin-dir ./my-plugins

# 设置 agent teammate 模式
claude --teammate-mode in-process
claude --teammate-mode tmux

# 指定可用工具
claude --tools "Bash,Edit,Read"
claude --tools ""  # 禁用所有工具
claude --tools "default"  # 启用所有工具
```

## 常用组合示例

```bash
# 快速代码审查（打印模式，使用 Sonnet）
claude -p --model sonnet "Review this file for issues"

# 继续上次会话，添加 TypeScript 要求
claude -c --append-system-prompt "Use strict TypeScript"

# 计划模式下重构代码
claude --permission-mode plan "Refactor this module"

# 安全分析（禁用 Bash 工具）
claude --tools "Read,Grep,Glob" "Analyze security issues"

# 远程调试（Chrome 集成）
claude --chrome "Debug the login form"

# 多目录项目工作
claude --add-dir ../frontend ../backend "Analyze project structure"
```
