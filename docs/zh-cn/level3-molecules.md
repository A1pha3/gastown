# Molecules 工作流

> **Level 3** ⭐⭐⭐ | 进阶分析
>
> 本文档深入讲解 Molecules 工作流系统的设计与实现。

## 学习目标

完成本章节学习后，你将能够：

### 基础目标
- 理解 Molecule 的生命周期
- 掌握 Formula → Protomolecule → Molecule 的转换流程
- 知道如何导航和执行多步骤工作流

### 进阶目标
- 分析 Wisp vs Molecule 的使用场景
- 理解 Hook 与 Molecule 的绑定机制
- 掌握工作流状态的实时追踪

### 专家目标
- 设计自定义 Formula
- 实现复杂的多步骤工作流
- 优化工作流执行效率

---

## 第一部分：Molecule 生命周期

### 状态转换图

```
Formula (TOML 源模板) ─── "Ice-9" 阶段
    │
    ▼ bd cook
Protomolecule (冻结模板) ─── 固态阶段
    │
    ├─▶ bd mol pour ──▶ Molecule (持久化) ─── 液态阶段 ──▶ bd squash ──▶ Digest
    │
    └─▶ bd mol wisp ──▶ Wisp (短暂) ─── 气态阶段 ──┬▶ bd squash ──▶ Digest
                                                     └▶ bd burn ──▶ (消失)
```

### 核心概念

| 术语 | 说明 | 持久性 |
|------|------|--------|
| **Formula** | TOML 源模板，定义工作流步骤 | 永久 |
| **Protomolecule** | 冻结模板，随时可用 | 永久 |
| **Molecule** | 活跃工作流实例，有可追踪步骤 | 持久到完成 |
| **Wisp** | 短暂 molecule（用于 patrol 循环） | 短暂 |
| **Digest** | 完成 molecule 的压缩摘要 | 永久 |

---

## 第二部分：Formula 使用

### 常见错误：直接读取 Formula

**错误做法**：

```bash
# 读取 formula 文件并手动创建每个 step 的 beads
cat .beads/formulas/mol-polecat-work.formula.toml
bd create --title "步骤 1: 加载上下文" --type task
bd create --title "步骤 2: 分支设置" --type task
# ... 从 formula 描述手动创建 beads
```

**正确做法**：

```bash
# 将 formula 烹饪成 proto，然后倒入 molecule
bd cook mol-polecat-work
bd mol pour mol-polecat-work --var issue=gt-xyz
# 现在处理创建的 step beads
bd ready                    # 查找下一步
bd close <step-id>          # 完成它
```

**关键洞察**：Formulas 是源模板（类似源代码）。你在工作期间从不直接读取它们。`cook` → `pour` 流水线为你创建 step beads。你的 molecule 已经有步骤了——使用 `bd ready` 查找它们。

---

## 第三部分：导航 Molecules

### 查找位置

```bash
bd mol current              # 我在哪？
bd mol current gt-abc       # 特定 molecule 的状态
```

**输出示例**：

```
你正在处理 molecule gt-abc (Feature X)

  ✓ gt-abc.1: 设计
  ✓ gt-abc.2: 脚手架
  ✓ gt-abc.3: 实现
  → gt-abc.4: 编写测试 [in_progress] <- 你在这
  ○ gt-abc.5: 文档
  ○ gt-abc.6: 退出决策

进度: 3/6 步骤完成
```

### 无缝转换

一步关闭并前进：

```bash
bd close gt-abc.3 --continue   # 关闭并前进到下一步
bd close gt-abc.3 --no-auto    # 关闭但不自动认领下一步
```

**旧方式（3 个命令）**：

```bash
bd close gt-abc.3
bd ready --parent=gt-abc
bd update gt-abc.4 --status=in_progress
```

**新方式（1 个命令）**：

```bash
bd close gt-abc.3 --continue
```

### 转换输出

```
✓ 关闭 gt-abc.3: 实现功能

Molecule 中的下一个就绪:
  gt-abc.4: 编写测试

→ 已标记 in_progress（使用 --no-auto 跳过）
```

### Molecule 完成时

```
✓ 关闭 gt-abc.6: 退出决策

Molecule gt-abc 完成！所有步骤已关闭。
考虑: bd mol squash gt-abc --summary '...'
```

---

## 第四部分：Molecule 命令

### Beads 操作 (bd)

```bash
# Formulas
bd formula list              # 可用 formulas
bd formula show <name>       # Formula 详情
bd cook <formula>            # Formula → Proto

# Molecules (数据操作)
bd mol list                  # 可用 protos
bd mol show <id>             # Proto 详情
bd mol pour <proto>          # 创建 mol
bd mol wisp <proto>          # 创建 wisp
bd mol bond <proto> <parent> # 附加到现有 mol
bd mol squash <id>           # 压缩为 digest
bd mol burn <id>             # 丢弃 wisp
bd mol current               # 我在当前 molecule 的哪？
```

### Agent 操作 (gt)

```bash
# Hook 管理
gt hook                      # 我的 Hook 上有什么？
gt mol current               # 我接下来应该做什么？
gt mol progress <id>         # molecule 执行进度
gt mol attach <bead> <mol>   # 将 molecule pin 到 bead
gt mol detach <bead>         # 从 bead unpin molecule

# Agent 生命周期
gt mol burn                  # 烧掉附加的 molecule
gt mol squash                # 压缩附加的 molecule
gt mol step done <step>      # 完成 molecule 步骤
```

---

## 第五部分：Polecat 工作流

### Hook 管理

Polecats 通过 hook 接收工作——一个附加到 issue 的 pinned molecule。

```bash
gt hook                        # 我的 Hook 上有什么？
gt mol attach-from-mail <id>   # 从邮件消息附加工作
gt done                        # 信号完成（同步、提交到 MQ、通知 Witness）
```

### Polecat 工作流总结

```
1. 在 Hook 上生成带工作
2. gt hook                 # 什么被 hooked?
3. bd mol current          # 我在哪？
4. 执行当前步骤
5. bd close <step> --continue
6. 如果还有步骤: 转到 3
7. gt done                 # 信号完成
```

### Wisp vs Molecule 决策

| 问题 | Molecule | Wisp |
|------|----------|------|
| 需要审计追踪？ | 是 | 否 |
| 会连续重复？ | 否 | 是 |
| 是离散交付物？ | 是 | 否 |
| 是运营例行程序？ | 否 | 是 |

**指导原则**：
- Polecats 使用 **常规 molecules**（每次分配有审计价值）
- Patrol 智能体（Witness、Refinery、Deacon）使用 **wisps**（防止累积）

---

## 第六部分：设计自定义 Formula

### Formula 结构

```toml
description = "工作流描述"
formula = "formula-name"
version = 1

# 变量定义
[vars.variable_name]
description = "变量说明"
required = true

# 步骤定义
[[steps]]
id = "step-id"
title = "步骤标题"
description = "详细说明"

# 依赖关系
[[steps]]
id = "dependent-step"
title = "依赖步骤"
description = "说明"
needs = ["step-id"]  # 依赖前面的步骤
```

### 示例：代码审查 Formula

```toml
description = "标准代码审查流程"
formula = "code-review"
version = 1

[vars.pr_number]
description = "要审查的 PR 编号"
required = true

[[steps]]
id = "fetch-context"
title = "获取上下文"
description = "克隆仓库并切换到 PR 分支"

[[steps]]
id = "review-code"
title = "审查代码"
description = "逐文件审查变更，留下评论"
needs = ["fetch-context"]

[[steps]]
id = "verify-tests"
title = "验证测试"
description = "运行测试套件确保通过"
needs = ["review-code"]

[[steps]]
id = "approve"
title = "批准或请求变更"
description = "根据发现批准或请求修改"
needs = ["verify-tests"]
```

### 使用 Formula

```bash
# 1. 将 Formula 添加到系统
cp my-formula.formula.toml ~/.beads/formulas/

# 2. 烹饪成 proto
bd cook code-review

# 3. 倒入带变量的 molecule
bd mol pour code-review --var pr_number=42

# 4. 执行步骤
bd ready
bd close <step-id> --continue
```

---

## 第七部分：最佳实践

### 1. 实时关闭步骤

**关键**：在工作前标记 `in_progress`，完成后立即标记 `closed`。永远不要在结束时批量关闭步骤。

**原因**：Molecules **就是**账本——每个步骤关闭都是一个带时间戳的 CV 条目。批量关闭破坏时间线并违反 HOP 的核心承诺。

### 2. 使用 --continue 保持推进

```bash
# 保持动力的最佳方式
bd close gt-abc.3 --continue
```

### 3. 用 bd mol current 检查进度

```bash
# 恢复工作前始终知道你在哪
bd mol current
```

### 4. 压缩完成的 molecules

```bash
# 为审计追踪创建 digests
bd mol squash gt-abc --summary '完成了用户认证系统'
```

### 5. 烧掉例行 wisps

```bash
# 不要累积短暂的 patrol 数据
bd mol burn <wisp-id>
```

---

## 第八部分：故障排查

### Molecule 卡住

```bash
# 检查状态
bd mol current

# 强制推进
bd ready --force

# 如果需要，重新附加
gt mol attach <bead> <molecule>
```

### 步骤顺序错误

```bash
# 检查依赖
bd mol show <molecule-id>

# 手动调整（谨慎使用）
bd update <step-id> --status=closed
```

### Wisp 累积

```bash
# 列出所有 wisps
bd mol list --type=wisp

# 批量清理
bd mol burn --all-wisps
```

---

## 延伸阅读

- [Molecules 概念](../concepts/molecules.md) - 英文原版
- [Polecat 生命周期](../concepts/polecat-lifecycle.md) - 智能体工作流
- [Formula 参考](../formula-resolution.md) - Formula 解析
