# 基础命令

> **Level 1** ⭐ | 入门教程
>
> 本文档提供 Gas Town 常用命令的快速参考。

## 命令分类

| 类别 | 说明 | 命令前缀 |
|------|------|----------|
| **工作空间** | 管理 Town 和 Rig | `gt install`, `gt rig` |
| **智能体** | 管理 AI 智能体 | `gt mayor`, `gt agents` |
| **工作追踪** | Convoy 和 Beads | `gt convoy`, `bd` |
| **通信** | 智能体间通信 | `gt mail`, `gt nudge` |
| **状态** | 查看系统状态 | `gt status`, `gt doctor` |

---

## 工作空间管理

### 初始化

```bash
# 创建并初始化工作空间
gt install ~/gt --git

# 仅初始化（不创建 Git）
gt install ~/gt
```

### Rig 管理

```bash
# 添加项目
gt rig add <name> <repository-url>

# 示例
gt rig add gastown https://github.com/steveyegge/gastown
gt rig add beads https://github.com/steveyegge/beads

# 列出所有 Rigs
gt rig list

# 查看 Rig 详情
gt rig show <name>

# 删除 Rig
gt rig remove <name>
```

### Crew 管理

```bash
# 创建成员工作区
gt crew add <name> --rig <rig>

# 示例
gt crew add alice --rig gastown

# 列出成员
gt crew list --rig <rig>

# 删除成员
gt crew remove <name> --rig <rig>
```

---

## 智能体操作

### Mayor

```bash
# 启动 Mayor 会话
gt mayor attach

# 在后台启动 Mayor
gt mayor start

# 查看状态
gt mayor status

# 停止 Mayor
gt mayor stop

# 分离 Mayor
gt mayor detach
```

### 查看智能体

```bash
# 列出所有活跃智能体
gt agents

# 按类型过滤
gt agents --type=polecat
gt agents --type=witness

# 按 Rig 过滤
gt agents --rig=gastown

# 查看详细信息
gt agent show <agent-id>
```

### Polecat 管理

```bash
# 列出 Polecats
gt polecat list --rig <rig>

# 清理已完成的 Polecats
gt polecat cleanup --rig <rig>

# 清理卡住的 Polecats
gt polecat cleanup --rig <rig> --stuck

# 手动清理特定 Polecat
gt polecat cleanup <name> --rig <rig>
```

---

## 工作分配与追踪

### Sling（投掷工作）

```bash
# 基本用法
gt sling <bead-id> <rig>

# 指定智能体名称
gt sling <bead-id> <rig> --agent <name>

# 使用特定运行时
gt sling <bead-id> <rig> --agent cursor

# 示例
gt sling gt-abc123 gastown
gt sling gt-def456 beads --agent codex
```

### Convoy 管理

```bash
# 创建 Convoy
gt convoy create "名称" [beads...]

# 带通知创建
gt convoy create "功能开发" gt-abc gt-def --notify

# 列出 Convoys
gt convoy list

# 显示详情
gt convoy show <convoy-id>

# 添加 Beads 到现有 Convoy
gt convoy add <convoy-id> <bead-ids...>

# 刷新状态
gt convoy refresh <convoy-id>
```

### Beads 操作

```bash
# 列出 Beads
bd list
bd list --type=task
bd list --type=issue
bd list --status=open

# 查看 Bead 详情
bd show <bead-id>

# 创建 Bead
bd create --title="标题" --type=task

# 更新 Bead
bd update <bead-id> --status=closed
bd update <bead-id> --title="新标题"

# 搜索 Beads
bd search "关键词"
```

---

## 通信命令

### Mail

```bash
# 查看收件箱
gt mail inbox

# 发送邮件
gt mail send <address> -s "主题" -m "内容"

# 阅读邮件
gt mail read <message-id>

# 标记为已读
gt mail ack <message-id>

# 清空收件箱
gt mail clear
```

### Nudge（轻推）

```bash
# 发送轻推
gt nudge <agent> "消息"

# 示例
gt nudge gastown/witness "检查 polecat 状态"

# 广播消息
gt nudge broadcast "系统维护通知"
```

---

## 状态与诊断

### 系统状态

```bash
# 查看整体状态
gt status

# 详细状态
gt status --verbose

# 按 Rig 查看
gt status --rig <rig>
```

### 诊断

```bash
# 全面健康检查
gt doctor

# 检查特定组件
gt doctor --component=dolt
gt doctor --component=tmux
gt doctor --component=git

# 修复问题
gt doctor --fix
```

### Hook 查看

```bash
# 查看我的 Hook
gt hook

# 查看 Molecule 进度
gt mol current

# 查看 Molecule 列表
gt mol list
```

---

## 配置命令

### 查看配置

```bash
# 显示所有配置
gt config show

# 显示特定配置
gt config get <key>

# 列出所有 Agent 配置
gt config agent list
```

### Agent 配置

```bash
# 设置自定义 Agent
gt config agent set <alias> <command>

# 示例
gt config agent set claude-glm "claude-glm --model glm-4"
gt config agent set codex-low "codex --thinking low"

# 设置默认 Agent
gt config default-agent claude-glm

# 删除 Agent 配置
gt config agent delete <alias>
```

---

## 完成工作

### Polecat 完成

```bash
# 完成当前工作（Polecat 必须执行）
gt done

# 这会：
# 1. 推送分支到 origin
# 2. 提交工作到合并队列
# 3. 清理自己的沙盒和会话
# 4. 立即退出
```

### Molecule 操作

```bash
# 完成当前步骤
gt mol step done <step-id>

# 压缩 Molecule
gt mol squash

# 烧掉 Wisp
gt mol burn
```

---

## 帮助与完成

### 获取帮助

```bash
# 总体帮助
gt help

# 命令帮助
gt <command> --help
gt <command> <subcommand> --help

# 示例
gt convoy --help
gt sling --help
```

### Shell 完成

```bash
# Bash
gt completion bash > /etc/bash_completion.d/gt
source /etc/bash_completion.d/gt

# Zsh
gt completion zsh > "${fpath[1]}/_gt"
compinit

# Fish
gt completion fish > ~/.config/fish/completions/gt.fish
```

---

## 速查表

### 最常用命令

```bash
# 日常工作流
gt mayor attach          # 启动 Mayor
gt convoy list           # 查看 Convoys
gt sling <bead> <rig>    # 分配工作
gt agents                # 查看智能体
gt status                # 查看状态
gt done                  # 完成工作（Polecat）
```

### 故障排查

```bash
gt doctor                # 健康检查
gt mail inbox            # 检查消息
gt polecat cleanup       # 清理卡住的智能体
gt convoy refresh        # 刷新状态
```

---

## 下一步

- [第一个任务](./level1-first-task.md) - 实践操作
- [架构概览](./level2-architecture.md) - 理解系统架构
