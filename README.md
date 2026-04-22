# Claude Code 学习笔记

这是一个关于 Claude Code 的学习笔记集合，记录使用经验、技巧和最佳实践。

## 文档索引

| 编号 | 文档 | 描述 |
|------|------|------|
| 01 | [Claude Code 介绍](./01-claude-code-介绍.md) | Claude Code 的简介、核心特性、安装方式及使用环境 |
| 02 | [Claude Code 命令大全](./02-claude-code-命令大全.md) | 完整的 CLI 命令参考，涵盖启动、会话管理、模型选择等 |
| 03 | [Claude Code 架构与工程实践](./03-claude-code-架构与工程实践.md) | 氪金换来的实战经验：六层架构、上下文治理、Skills 设计 |
| 04 | [Claude Code 核心概念全解析](./04-claude-code-核心概念全解析.md) | Memory、Skills、MCP、Hooks 等核心概念的大白话解析 |
| 05 | [Claude Code 记忆机制深度解析](./05-claude-code-记忆机制深度解析.md) | CLAUDE.md 与 Auto Memory 详解，双记忆系统全指南 |
| 06 | [Claude Code 规则机制深度解析](./06-claude-code-规则机制深度解析.md) | 模块化规则系统详解，路径特定规则、层级继承与最佳实践 |
| 07 | [Claude Agent 深度解析](./07-claude-agent-深度解析.md) | Claude Agent / Subagent 详解，涵盖 `/agents`、作用域、配置与使用策略 |

## 快速开始

```bash
# 安装 Claude Code (macOS/Linux)
curl -fsSL https://claude.ai/install.sh | bash

# 启动交互式会话
claude

# 使用初始提示启动
claude "explain this project"
```

## 项目结构

```
.
├── CLAUDE.md                          # Claude Code 工作指南
├── README.md                          # 本文档
├── 01-claude-code-介绍.md              # Claude Code 介绍
├── 02-claude-code-命令大全.md          # 命令参考
├── 03-claude-code-架构与工程实践.md     # 架构与工程实践
├── 04-claude-code-核心概念全解析.md     # Memory、Skills、MCP、Hooks 等核心概念
├── 05-claude-code-记忆机制深度解析.md     # CLAUDE.md 与 Auto Memory 详解
├── 06-claude-code-规则机制深度解析.md     # 模块化规则系统详解
├── 07-claude-agent-深度解析.md          # Claude Agent / Subagent 详解
└── scripts/                           # 可执行脚本目录
```

## 贡献

欢迎补充更多学习笔记和实用脚本！
