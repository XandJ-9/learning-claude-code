# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

本目录是一个关于claude code的学习笔记集合，可执行的脚本文件与描述性的文档需要分类存放。

## 文件组织规范

- **描述性文档**：存放于当前目录，使用数字前缀编号（如 `01-xxx.md`、`02-xxx.md`）
- **可执行脚本**：统一存放于 `scripts/` 子目录下

## Python 脚本执行

执行 Python 脚本时必须使用 `uv` 命令，避免污染本地 Python 环境：

```bash
# 执行 scripts 目录下的 Python 脚本
uv run scripts/script-name.py [args]
```


