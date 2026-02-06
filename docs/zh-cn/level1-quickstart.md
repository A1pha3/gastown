# 快速入门

> **Level 1** ⭐ | 入门教程
>
> 本教程帮助你在 5 分钟内启动并运行 Gas Town。

## 前置要求

在开始之前，确保已安装：

| 软件 | 最低版本 | 说明 |
|------|----------|------|
| **Go** | 1.23+ | go.dev/dl |
| **Git** | 2.25+ | 用于 worktree 支持 |
| **beads (bd)** | 0.44.0+ | github.com/steveyegge/beads |
| **sqlite3** | 任意 | 通常 macOS/Linux 已预装 |
| **tmux** | 3.0+ | 推荐用于完整体验 |
| **Claude Code** | 最新版 | claude.ai/code |

## 安装 Gas Town

### 方式一：Homebrew（推荐）

```bash
brew install gastown
```

### 方式二：npm

```bash
npm install -g @gastown/gt
```

### 方式三：从源码安装

```bash
go install github.com/steveyegge/gastown/cmd/gt@latest

# 添加到 PATH（添加到 ~/.zshrc 或 ~/.bashrc）
export PATH="$PATH:$HOME/go/bin"
```

## 初始化工作空间

```bash
# 创建并初始化工作空间
gt install ~/gt --git
cd ~/gt

# 验证安装
gt doctor
```

**预期输出**：

```
✓ Go version: 1.24.2
✓ Git version: 2.45.0
✓ beads (bd): 0.44.0
✓ Claude Code: installed
✓ Tmux: 3.4 installed
```

## 添加第一个项目

```bash
# 添加项目
gt rig add gastown https://github.com/steveyegge/gastown

# 验证
gt rig list
```

## 启动 Mayor

Mayor 是你的主要 AI 协调者：

```bash
gt mayor attach
```

**在 Mayor 会话中**，你可以：

```
告诉我你想要构建什么，我来帮你协调智能体完成工作。
```

## 核心命令速查

```bash
# 工作空间管理
gt rig list                   # 列出所有项目
gt crew add <name>            # 创建成员工作区

# 智能体操作
gt mayor attach               # 启动 Mayor
gt agents                     # 列出活跃智能体
gt sling <bead-id> <rig>      # 分配工作

# 工作追踪
gt convoy create <name>       # 创建工作追踪单元
gt convoy list                # 列出所有 convoy
```

## 下一步

- [第一个任务](./level1-first-task.md) - 完成第一个开发任务
- [基础命令](./level1-basic-commands.md) - 常用命令详解
- [架构概览](./level2-architecture.md) - 理解系统架构
