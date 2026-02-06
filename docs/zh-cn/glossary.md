# Gas Town 术语表

> 本文档提供 Gas Town 中所有核心术语的权威定义。

## 使用说明

- **术语条目**：每个术语按中文拼音排序
- **格式规范**：中文术语 + 英文原文 + 标准发音（可选）+ 简要定义
- **深度链接**：点击术语可跳转到详细文档

---

## A

### Agent / 智能体
AI 编码助手的抽象实体。在 Gas Town 中，智能体可以是 Mayor、Deacon、Witness、Refinery、Polecat 或 Crew 成员。

**相关**：[Agent 生命周期](./level2-agent-lifecycle.md)

### Agent Bead / 智能体 Bead
存储智能体生命周期状态的 Bead 记录。每个智能体都有对应的 Agent Bead，用于追踪其状态和历史。

**格式**：`{prefix}-{role}-{name}`，例如 `gt-witness-main`

---

## B

### Bead / Bead
存储在 JSONL 格式中的原子工作单元。Bead 是 Gas Town 工作追踪系统的核心数据结构，可以代表 issues、任务、epics 或任何可追踪的工作项。

**ID 格式**：`{prefix}{5字符ID}`，例如 `gt-abc12`

**相关**：[Beads 系统](./level2-beads.md)

### BD_ACTOR
标识智能体的环境变量，使用斜杠分隔的路径格式。

**格式**：`{rig}/{role}/{name}` 或 `{rig}/{role}`

**示例**：`gastown/polecats/toast`、`gastown/witness`

**相关**：[身份与归因](./level4-identity.md)

### Boot / Boot 狗
一种特殊的 Dog，每 5 分钟检查一次 Deacon，确保看门狗本身仍在工作。这创建了一个问责链。

**相关**：[看门狗链设计](../design/watchdog-chain.md)

---

## C

### Convoy / 车队
将相关 Beads 打包分配给多个智能体的工作追踪单元。Convoy 是主要的协调机制，用于跟踪跨多个智能体的工作进度。

**创建**：`gt convoy create "名称" [beads...]`

**相关**：[Convoy 概念](../concepts/convoy.md)

### Crew / 成员
长命的命名智能体，用于持续协作。与短命的 Polecats 不同，Crew 成员在会话间保持上下文，适合长期工作关系。

**相关**：[Agent 生命周期](./level2-agent-lifecycle.md)

---

## D

### Deacon / 执事
守护进程信标，运行持续的 Patrol 周期。Deacon 确保工作者活动、监控系统健康并在智能体无响应时触发恢复。

**位置**：Town 级别（`~/gt/deacon/`）

**相关**：[Dog Pool 架构](../design/dog-pool-architecture.md)

### Dog / 狗
Deacon 的维护智能体团队，处理后台任务如清理、健康检查和系统维护。

**类型**：Boot、Cleanup、Health Check、Notification

### Dolt
具有 Git 风格版本控制的 SQL 数据库，是 Gas Town 的存储后端。

**端口**：3307（默认）

**相关**：[Dolt 存储架构](../design/dolt-storage.md)

---

## E

### Escalation / 升级
当智能体无法自行解决问题时，将问题上报给更高层级的机制。

**流程**：Polecat → Witness → Mayor → 人类

**相关**：[升级系统设计](../design/escalation-system.md)

---

## F

### Formula / 公式
TOML 格式的工作流源模板。Formulas 定义了常见操作的可重用模式，如 patrol 循环、代码审查或部署。

**位置**：`.beads/formulas/`

**相关**：[Molecules 工作流](./level3-molecules.md)

---

## G

### GUPP / Gas Town 通用推进原则
**"如果你的 Hook 上有工作，你必须执行它"**

这是确保智能体自主运行的核心原则。

**相关**：[推进原则](../concepts/propulsion-principle.md)

---

## H

### Handoff / 移交
通过 `/handoff` 命令实现的智能体会话刷新。当上下文已满或智能体需要全新开始时，移交将工作状态转移到新会话。

**命令**：`gt handoff`

### Hook / 钩子
每个智能体的特殊 pinned Bead，作为主要工作队列。当工作出现在你的 Hook 上时，GUPP 规定你必须执行它。

**查看**：`gt hook`

### HQ Bead / 总部 Bead
存储在 Town 级别（`~/gt/.beads/`）的 Bead，使用 `hq-` 前缀。

**用途**：跨 Rig 协调、Mayor 邮件、智能体身份

---

## L

### Level / 级别
Gas Town 两级架构中的层级。

| 级别 | 位置 | 前缀 | 用途 |
|------|------|------|------|
| **Town** | `~/gt/` | `hq-*` | 跨 Rig 协调 |
| **Rig** | `<rig>/mayor/rig/` | 项目前缀 | 实现工作 |

---

## M

### Mail / 邮件
智能体间通过 Beads 系统路由的异步消息。Mail 使用 `type=message` 的 Beads，通过 `gt mail` 处理路由。

**相关**：[Mail 协议](../design/mail-protocol.md)

### Mayor / 市长
主要 AI 协调者，负责创建 Convoys、协调工作分配和通知用户重要事件。Mayor 从 Town 级别运作，对所有 Rig 有可见性。

**启动**：`gt mayor attach`

### MEOW / 工作的分子表达
将大型目标分解为详细智能体指令的原则。由 Beads、Epics、Formulas 和 Molecules 支持。

**目标**：确保工作被分解为可追踪的原子单元

### Merge Queue / 合并队列
由 Refinery 管理的工作队列，处理来自 Polecats 的合并请求。

**相关**：[Refinery 设计](../design/refinery.md)

### Molecule / 分子
持久的链式 Bead 工作流。Molecules 代表多步骤过程，其中每个步骤都被追踪为 Bead。它们在智能体重启后幸存。

**相关**：[Molecules 工作流](./level3-molecules.md)

### Mol / Mol
Molecule 的简称。

---

## N

### NDI / 非确定性幂等性
通过编排可能不可靠的流程实现有用结果的总体目标。持久 Beads 和监督智能体（Witness、Deacon）确保最终工作流完成。

**相关**：[故障恢复](./level3-fault-recovery.md)

### Nudging / 轻推
智能体间通过 `gt nudge` 实现实时消息传递。Nudging 允许即时通信，无需通过邮件系统。

---

## O

### Operational Plane / 操作平面
三层数据平面之一，存储高变更频率的实时数据。

**内容**：进行中的工作、状态变更、分配、patrol 结果、molecule 转换、心跳

**相关**：[数据平面设计](./level4-data-planes.md)

---

## P

### Patrol / 巡逻
维持系统心跳的短暂循环。Patrol 智能体（Deacon、Witness）持续循环健康检查并根据需要触发操作。

### Polecat / 鼬
短命的工作智能体，生成合并请求。Polecats 被生成用于特定任务，完成工作，然后被清理。它们在隔离的 git worktrees 中工作以避免冲突。

**生命周期**：生成 → 工作 → 完成 → 自我清理（`gt done`）

**相关**：[Polecat 生命周期](../concepts/polecat-lifecycle.md)

### Protomolecule / 原分子
分子模板类，用于实例化 Molecules。Protomolecules 定义了工作流的结构和步骤，而不绑定到特定工作项。

---

## R

### Refinery / 精炼厂
管理 Rig 合并队列的智能体。Refinery 智能地合并来自 Polecats 的变更，处理冲突并确保代码在变更到达主分支前的质量。

**位置**：`<rig>/refinery/`

### Rig / 钻井平台
Gas Town 管理的特定 Git 仓库的项目容器。每个 Rig 有其自己的 Polecats、Refinery、Witness 和 Crew 成员。

**添加**：`gt rig add <name> <repo>`

### Role Bead / 角色 Bead
存储在 Town Beads 中带有 `hq-` 前缀的全局模板。

**示例**：`hq-mayor-role`、`hq-witness-role`

### Route / 路由
`routes.jsonl` 文件将 issue ID 前缀映射到 rig 位置（相对于 town root）。

**格式**：`{"prefix":"gt-","path":"gastown/mayor/rig"}`

---

## S

### Seance / 降神会
通过 `gt seance` 与前几代智能体通信。允许智能体查询前代以获取早期工作的上下文和决策。

### Slinging / 投掷
通过 `gt sling` 将工作分配给智能体。当你向 Polecat 或 Crew 成员投掷工作时，你将其放在他们的 Hook 上执行。

**命令**：`gt sling <bead-id> <rig>`

---

## T

### Town / 镇
管理总部（例如 `~/gt/`）。Town 协调跨多个 Rigs 的工作者和智能体，并容纳镇级智能体如 Mayor 和 Deacon。

**初始化**：`gt install ~/gt`

---

## W

### Witness / 见证人
监控 Polecats 和 Refinery 的 Patrol 智能体。Witness 监控进度、检测卡住的智能体，并可触发恢复操作。

**位置**：`<rig>/witness/`

**相关**：[Witness 设计](../design/witness.md)

### Wisp / 火花
在运行后被销毁的短暂 Beads。Wisps 是用于不需要永久跟踪的短暂操作的工作项。

**相关**：[Molecules 工作流](./level3-molecules.md)

---

## 核心原则速查

| 原则 | 英文 | 含义 |
|------|------|------|
| **MEOW** | Molecular Expression of Work | 将大型目标分解为可追踪的步骤 |
| **GUPP** | Gas Town Universal Propulsion Principle | "如果你的 Hook 上有工作，你必须执行它" |
| **NDI** | Nondeterministic Idempotence | 通过编排实现最终一致性 |

---

## 架构层级速查

```
Town 级别（跨 Rig 协调）
├── Mayor / 市长
├── Deacon / 执事
└── Dogs / 狗

Rig 级别（项目执行）
├── Witness / 见证人
├── Refinery / 精炼厂
├── Polecats / 鼬
└── Crew / 成员
```

---

## 延伸阅读

- [英文术语表](../glossary.md)
- [架构概览](./level2-architecture.md)
- [命令参考](../reference.md)
