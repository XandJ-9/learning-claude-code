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

**使用场景：**
- **交互式会话**：当你需要进行多轮对话、逐步探索代码库或执行复杂任务时，直接输入 `claude` 进入交互模式最合适
- **初始提示启动**：当你已经明确知道要做什么，想让 Claude 直接开始特定任务时使用，比如新项目接手时快速了解代码结构
- **打印模式**：适合脚本集成或快速获取单次答案，比如在 CI/CD 流水线中进行代码分析，或者需要将输出结果传递给其他命令

### 管道输入

```bash
# 处理管道内容
cat file.txt | claude -p "summarize this"
cat logs.txt | claude -p "analyze errors"
git diff | claude -p "review this diff"
```

**使用场景：**
- **日志分析**：当应用程序出现问题时，将错误日志通过管道传递给 Claude 进行快速诊断
- **代码审查**：在提交代码前，将 `git diff` 的输出传递给 Claude 进行自动审查
- **文档总结**：阅读长文档时，让 Claude 快速总结要点
- **配置文件解析**：理解复杂的配置文件（如 nginx.conf、docker-compose.yml）时使用

## 会话管理

### 继续会话

```bash
# 继续当前目录中最近的对话
claude -c
claude --continue

# 继续并通过 SDK 发送消息
claude -c -p "Check for type errors"
```

**使用场景：**
- **中断后恢复**：当你正在与 Claude 协作时被其他事情打断（如开会、午餐），使用 `-c` 快速恢复上次的对话上下文
- **多轮开发**：在实现复杂功能时，可能需要多次会话才能完成，每次继续时保持上下文连贯
- **批处理检查**：在开发过程中，定期让 Claude 检查代码质量、类型错误等，同时保持会话历史

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

**使用场景：**
- **多任务切换**：同时处理多个功能或 Bug 修复时，每个任务使用独立的会话，通过名称快速切换
- **会话历史回溯**：当之前的解决方案有问题，需要回到某个历史会话重新审视时使用
- **团队协作**：团队成员通过会话 ID 分享特定问题的解决过程

### 会话分支

```bash
# 恢复时创建新会话（不修改原会话）
claude --resume abc123 --fork-session
claude -c --fork-session
```

**使用场景：**
- **探索性开发**：你想在一个会话的基础上尝试不同的解决方案，但不想破坏原有的会话记录
- **A/B 对比**：实现同一个功能时，创建多个分支尝试不同的实现方案，最后选择最优的
- **安全实验**：在已有解决方案的基础上进行大胆尝试，如果失败了可以随时回到原会话

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

**使用场景：**
- **首次使用**：新机器或新用户首次使用 Claude Code 时需要登录
- **SSO 企业环境**：在公司环境下，使用 `--sso` 强制通过单点登录认证
- **脚本集成**：在自动化脚本中检查认证状态，确保已登录后再执行后续操作
- **多账户切换**：当需要在个人账户和工作账户之间切换时使用

## 更新与版本

```bash
# 更新到最新版本
claude update

# 查看当前版本
claude -v
claude --version
```

**使用场景：**
- **获取新功能**：Claude Code 持续更新，定期运行 `claude update` 获取最新功能和性能改进
- **问题排查**：遇到 Bug 时，先检查版本号，确认是否是已知问题或已在新版本中修复
- **环境一致性**：团队协作时，确保所有成员使用相同版本，避免因版本差异导致的问题

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

**使用场景：**
- **Opus（最强）**：适合需要深度推理的复杂任务，如架构设计、复杂 Bug 修复、跨文件重构
- **Sonnet（均衡）**：日常开发的首选，性能和成本的最佳平衡点
- **Haiku（最快）**：简单任务如代码格式化、快速问答、批量文件处理
- **成本控制**：在大型项目或长时间会话中，选择合适的模型控制 API 调用成本

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

**使用场景：**
- **计划模式**：进行大型重构或新功能开发时，让 Claude 先制定完整计划，你审查通过后再执行，避免意外修改
- **受信任环境**：在本地开发且信任代码库时，使用 `--dangerously-skip-permissions` 提高效率（不建议在生产环境使用）
- **只读分析**：只想让 Claude 分析代码而不做任何修改时，禁用 `Edit` 和 `Bash` 工具
- **安全审查**：禁用危险命令（如 `rm`、`git push`），防止意外操作
- **Git 集成**：允许只读 Git 命令（log、diff）查看历史，但阻止写操作

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

**使用场景：**
- **技术栈定制**：在 Python 项目中使用 `--append-system-prompt` 添加 Python 专家上下文，让回答更精准
- **团队规范**：将团队的编码规范（如命名约定、注释要求）保存在文件中，确保所有成员获得一致的代码风格
- **项目特定要求**：在遗留代码项目中，添加项目特定的架构说明和约定
- **专业角色**：临时让 Claude 扮演特定角色（如安全专家、性能优化顾问）进行专项任务
- **完整替换**：需要完全控制 Claude 行为时使用，比如构建专门的代码生成工具

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

**使用场景：**
- **Monorepo 项目**：当代码库包含多个相关目录（如 frontend、backend、shared）时，让 Claude 能够访问完整的上下文
- **微服务架构**：同时处理多个服务时，添加所有服务的目录以便跨服务分析和修改
- **Git Worktree**：在并行开发多个分支时，每个 worktree 独立运行 Claude，避免会话混乱
- **会话关联**：在外部工具（如项目管理软件）中记录会话 ID，实现会话追踪和关联

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

**使用场景：**
- **前端调试**：开发 Web 应用时，使用 `--chrome` 让 Claude 能够控制浏览器进行自动化测试、表单填写、截图等
- **E2E 测试**：让 Claude 帮助编写或调试端到端测试脚本
- **IDE 协作**：在 VS Code 或 JetBrains 中开发时，使用 `--ide` 直接在编辑器中查看差异和应用更改
- **远程工作**：离开电脑前启动 Remote Control，之后在手机或另一台设备上继续控制本地会话
- **演示准备**：Remote Control 可以让 Claude 在本地执行任务，你在远程查看结果

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

**使用场景：**
- **专业分工**：配置代码审查、测试、文档等专门的 agent，各司其职提高效率
- **MCP 服务器**：集成外部工具和服务（如数据库、API、文件系统），扩展 Claude 的能力边界
- **团队 agent 库**：在团队配置中共享经过验证的 agent 定义，确保一致的工作流
- **临时任务**：使用 `--agents` 动态创建专门的 agent 处理特定任务，完成后即弃
- **隔离环境**：使用 `--strict-mcp-config` 确保只使用指定的 MCP 服务器，提高安全性

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

**使用场景：**
- **脚本解析**：使用 JSON 输出格式，让其他程序能够解析 Claude 的响应，实现自动化工作流
- **实时处理**：流式 JSON 输出允许在 Claude 生成响应的同时进行处理，降低延迟
- **数据验证**：使用 JSON Schema 确保输出符合预期格式，便于后续处理
- **CI/CD 集成**：在持续集成流程中，让 Claude 生成结构化的报告（如测试结果、代码质量评分）
- **API 开发**：构建基于 Claude 的 API 服务时，结构化输出确保响应格式的一致性

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

**使用场景：**
- **问题排查**：当 Claude 行为异常时，启用调试模式查看详细的 API 请求和响应
- **性能分析**：查看 API 调用耗时、token 使用量等，优化使用成本
- **MCP 调试**：配置 MCP 服务器时，使用 `--debug mcp` 查看连接和通信细节
- **错误报告**：向 Claude Code 团队报告 Bug 时，附上详细日志帮助定位问题

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

**使用场景：**
- **长时间任务**：启动耗时的代码分析或重构任务，在 Web 界面查看进度，完成后回来检查结果
- **云端工作**：在没有本地代码仓库的环境中（如平板电脑），通过 Web 界面使用 Claude Code
- **PR 协作**：通过 GitHub PR 创建时自动关联会话，使用 `--from-pr` 恢复特定 PR 的上下文
- **任务交接**：在本地开始的任务可以通过 Remote Control 交接给其他设备继续

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

**使用场景：**
- **成本控制**：使用 `--max-turns` 和 `--max-budget-usd` 限制资源消耗，防止意外超支
- **临时查询**：使用 `--no-session-persistence` 进行一次性查询，不污染会话历史
- **高可用性**：设置 `--fallback-model` 确保在主模型过载时自动切换，避免服务中断
- **纯净环境**：使用 `--disable-slash-commands` 禁用自定义 skills，确保默认行为
- **自动化**：`--init-only` 和 `--maintenance` 适合在 CI/CD 或 cron 任务中使用
- **团队配置**：通过 `--settings` 和 `--setting-sources` 实现项目级和用户级的配置分层
- **插件系统**：通过 `--plugin-dir` 加载团队共享的插件和扩展

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

**使用场景：**
- **提交前审查**：快速检查即将提交的代码，使用 Sonnet 平衡速度和质量
- **持续开发**：在开发过程中保持会话连续性，同时强化技术栈要求
- **大型重构**：先让 Claude 制定详细计划，确保你对改动范围有完全掌控
- **安全审计**：只读分析代码安全风险，避免任何意外的代码修改
- **Web 调试**：结合浏览器自动化能力，定位和修复前端问题
- **全栈分析**：跨越前后端代码库，进行架构级别的分析和修改
