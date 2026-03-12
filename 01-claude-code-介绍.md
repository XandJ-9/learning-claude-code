# Claude Code 介绍

## 什么是 Claude Code

Claude Code 是 Anthropic 推出的 AI 编程助手，可以直接在终端、IDE、桌面应用和浏览器中使用。它能够理解整个代码库，跨多个文件和工具协作，帮助开发者构建功能、修复 Bug 和自动化开发任务。

## 核心特性

- **智能代码理解**：深度分析整个代码库结构
- **多文件编辑**：同时处理多个文件的修改
- **命令执行**：直接运行 shell 命令和测试
- **工具集成**：与 Git、MCP 服务器、CI/CD 等深度集成

## 使用环境

| 环境 | 特点 |
|------|------|
| **Terminal CLI** | 完整功能的命令行工具，适合日常开发 |
| **VS Code** | 内联差异视图、@提及、计划审查 |
| **JetBrains** | 支持 IntelliJ IDEA、PyCharm、WebStorm 等 |
| **Desktop App** | 可视化差异查看、多会话并行 |
| **Web** | 无需本地安装，可在浏览器和 Claude iOS 应用中使用 |

## 安装方式

### macOS / Linux / WSL

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

### Windows (PowerShell)

```powershell
irm https://claude.ai/install.ps1 | iex
```

### Homebrew

```bash
brew install --cask claude-code
```

### WinGet

```powershell
winget install Anthropic.ClaudeCode
```

## 常用命令

```bash
# 启动交互式会话
claude

# 使用初始提示启动
claude "explain this project"

# 通过 SDK 查询后退出
claude -p "explain this function"

# 继续最近的对话
claude -c

# 恢复指定会话
claude -r "session-name"

# 登录
claude auth login

# 查看认证状态
claude auth status

# 更新版本
claude update
```

## 常用选项

| 选项 | 说明 |
|------|------|
| `-p, --print` | 打印响应而不进入交互模式 |
| `-c, --continue` | 继续当前目录最近的对话 |
| `-r, --resume` | 按 ID 或名称恢复会话 |
| `--model` | 指定模型（sonnet/opus/haiku） |
| `--permission-mode` | 设置权限模式（plan/默认） |
| `--append-system-prompt` | 附加自定义系统提示 |

## 工作流程集成

| 需求                    | 最佳方案                       |
| --------------------- | -------------------------- |
| 从手机继续本地会话             | Remote Control             |
| 自动化 PR 审查             | GitHub Actions / GitLab CI |
| 获取每条 PR 的自动代码审查       | GitHub Code Review         |
| 从 Slack 路由 Bug 报告到 PR | Slack 集成                   |
| 调试实时 Web 应用           | Chrome 集成                  |
| 构建自定义工作流代理            | Agent SDK                  |

## 官方资源

- [官方文档](https://code.claude.com/docs/en/overview)
- [CLI 参考](https://code.claude.com/docs/zh-CN/cli-reference)
- [产品页面](https://claude.com/product/claude-code)
