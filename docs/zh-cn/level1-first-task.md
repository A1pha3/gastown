# 第一个任务

> **Level 1** ⭐ | 入门教程
>
> 本教程引导你完成在 Gas Town 中的第一个开发任务。

## 学习目标

完成本教程后，你将能够：

- [ ] 使用 Mayor 创建一个 Convoy
- [ ] 分配工作给 Polecat
- [ ] 跟踪工作进度
- [ ] 理解基本的工作流程

---

## 前置准备

在开始之前，确保已完成：

```bash
# 1. 已安装 Gas Town
which gt

# 2. 已初始化工作空间
ls ~/gt

# 3. 已添加一个测试项目
gt rig list
```

---

## 任务场景

假设我们要完成一个简单的任务：**在项目中添加一个 README 文件**

---

## 步骤 1：启动 Mayor

Mayor 是你的主要 AI 协调者：

```bash
cd ~/gt
gt mayor attach
```

你会进入一个 Mayor 会话，这是与 AI 协作者交互的主要界面。

---

## 步骤 2：创建工作项

在 Mayor 会话中，告诉 Mayor 你想做什么：

```
我想在 gastown 项目中添加一个 README 文件，描述项目的基本信息。
```

Mayor 会：
1. 理解你的需求
2. 创建相应的 Beads（工作项）
3. 将工作组织成 Convoy

---

## 步骤 3：查看创建的 Convoy

在新的终端窗口中：

```bash
cd ~/gt
gt convoy list
```

你会看到类似输出：

```
CONVOYS
┌──────────────────────────────────────────────────────────┐
│ ID    名称              Beads     状态      创建时间      │
├──────────────────────────────────────────────────────────┤
│ hq-1  添加 README       1         active    2 分钟前     │
└──────────────────────────────────────────────────────────┘
```

```bash
# 查看详细信息
gt convoy show hq-1
```

---

## 步骤 4：分配工作给 Polecat

Mayor 会自动建议如何分配工作。你也可以手动分配：

```bash
# 查看可用的 beads
bd list --type=task

# 分配工作
gt sling <bead-id> gastown
```

这会：
1. 创建一个新的 Polecat
2. 将工作放入 Polecat 的 Hook
3. Polecat 开始执行任务

---

## 步骤 5：监控 Polecat 进度

```bash
# 查看所有活跃的智能体
gt agents

# 查看 Polecat 的状态
gt polecat status --rig gastown
```

你会看到类似输出：

```
POLECATS in gastown
┌──────────────────────────────────────────────┐
│ 名称        状态      当前任务     最后活跃  │
├──────────────────────────────────────────────┤
│ toast-1    working    gt-abc12    30 秒前   │
└──────────────────────────────────────────────┘
```

---

## 步骤 6：Polecat 完成工作

当 Polecat 完成任务后，它会：

```bash
# 1. 推送更改到 Git
# 2. 提交到合并队列
# 3. 清理自己的工作区
# 4. 通知 Witness

# 你会看到
gt polecat list --rig gastown
# (已完成的 Polecat 不会出现在列表中)
```

---

## 步骤 7：审查工作结果

```bash
cd ~/gt/gastown/mayor/rig
git log -1 --oneline
```

你会看到 Polecat 创建的提交：

```
abc1234 Add README file (gastown/polecats/toast-1 <your@email.com>)
```

---

## 步骤 8：合并工作（可选）

如果有 Refinery 运行，它会自动处理合并。否则：

```bash
# 查看合并队列
gt refinery queue --rig gastown

# 手动合并（如果需要）
git merge polecat/toast-1-feature-xyz
```

---

## 工作流程总结

```
1. gt mayor attach          # 启动 Mayor
   └── 告诉 Mayor 你想做什么

2. gt convoy create         # Mayor 创建 Convoy
   └── 自动或手动分配工作

3. gt sling <bead> <rig>    # 分配给 Polecat
   └── Polecat 自动开始工作

4. gt agents / gt convoy list  # 监控进度
   └── 查看工作状态

5. Polecat 完成后自动清理
   └── 工作被推送到 Git
```

---

## 常见问题

### Mayor 没有响应？

```bash
# 检查 Mayor 是否在运行
gt mayor status

# 重启 Mayor
gt mayor detach
gt mayor attach
```

### Polecat 卡住了？

```bash
# 查看 Polecat 状态
gt polecat list --rig gastown --stuck

# 清理卡住的 Polecat
gt polecat cleanup --rig gastown --stuck
```

### 找不到创建的 Bead？

```bash
# 检查 Beads 是否创建成功
bd list --all

# 按类型过滤
bd list --type=task
bd list --type=issue
```

---

## 下一步

恭喜！你已经完成了第一个 Gas Town 任务。接下来：

- [基础命令](./level1-basic-commands.md) - 学习更多命令
- [架构概览](./level2-architecture.md) - 理解系统如何工作
- [Beads 系统](./level2-beads.md) - 深入了解工作追踪
