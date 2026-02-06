# 身份与归因系统设计

> **Level 4** ⭐⭐⭐⭐ | 专家级文档
>
> 本文档深入分析 Gas Town 的通用归因系统设计，这是实现大规模智能体协调的关键基础设施。

## 学习目标

完成本章节学习后，你将能够：

### 基础目标
- 理解为什么 AI 系统需要身份归因
- 掌握 BD_ACTOR 格式规范和设计原理
- 理解 Git 提交中的双字段归因模型

### 进阶目标
- 分析智能体身份与人类所有者的关系
- 理解持久身份与临时会话的分离设计
- 掌握基于归因的能力路由机制

### 专家目标
- 设计类似的归因系统
- 实现 A/B 模型对比和能力评估
- 构建跨工作空间的技能累积系统

---

## 第一部分：为什么需要身份归因？

### 问题场景

当你部署 20+ 个 AI 智能体协同工作时，会出现以下问题：

| 场景 | 无归因 | 有归因 |
|------|--------|--------|
| **Bug 调试** | "AI 写坏了" | "gastown/polecats/toast 在 3f2a1 提交中引入" |
| **质量追踪** | 无法量化 | 对比智能体的修订次数和完成率 |
| **合规审计** | 无法回答 | "steve@example.com 的团队在 Q3 完成了..." |
| **能力路由** | 随机分配 | "发送 Go 任务给有 Go 记录的智能体" |

### 核心洞察

**洞察**：在规模化 AI 部署中，匿名工作是技术债，而非隐私保护。

**设计原则**：

> **Universal Attribution（通用归因）**：每一个动作、每一个提交、每一个 Bead 更新都必须链接到特定的身份。

---

## 第二部分：BD_ACTOR 格式设计

### 格式规范

`BD_ACTOR` 环境变量使用斜杠分隔的路径格式标识智能体：

```
{rig}/{role}/{name}
```

### 各角色的格式

| 角色类型 | 格式 | 示例 |
|----------|------|------|
| **Mayor** | `mayor` | `mayor` |
| **Deacon** | `deacon` | `deacon` |
| **Witness** | `{rig}/witness` | `gastown/witness` |
| **Refinery** | `{rig}/refinery` | `gastown/refinery` |
| **Crew** | `{rig}/crew/{name}` | `gastown/crew/joe` |
| **Polecat** | `{rig}/polecats/{name}` | `gastown/polecats/toast` |

### 为什么使用斜杠？

**设计决策**：采用斜杠格式而非点分隔或冒号分隔

**理由**：

```
理由一：与文件系统路径一致
├── 直观反映层级关系
└── 便于解析和理解

理由二：与邮件地址统一
├── gt mail send gastown/witness
└── 路径即地址

理由三：支持通配符查询
├── bd audit --actor=gastown/*
└── bd audit --actor=*/polecats/*
```

**格式对比**：

| 方案 | 示例 | 可读性 | 可解析性 | 扩展性 |
|------|------|--------|----------|--------|
| **斜杠** | `gastown/polecats/toast` | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |
| **点分隔** | `gastown.polecats.toast` | ⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **冒号** | `gastown:polecats:toast` | ⭐⭐⭐ | ⭐⭐⭐⭐ | ⭐⭐⭐ |
| **@ 符号** | `toast@gastown.polecats` | ⭐⭐⭐ | ⭐⭐⭐ | ⭐⭐ |

---

## 第三部分：归因模型

### 三字段归因

Gas Town 在三个不同的系统中实现归因：

#### 1. Git 提交

```bash
GIT_AUTHOR_NAME="gastown/crew/joe"      # 执行者（智能体）
GIT_AUTHOR_EMAIL="steve@example.com"    # 所有者（人类）
```

**Git 日志中的表现**：

```
abc123 Fix bug (gastown/crew/joe <steve@example.com>)
```

**设计含义**：

| 字段 | 含义 | 永久性 | 用途 |
|------|------|--------|------|
| `AUTHOR_NAME` | 智能体身份 | 永久 | 技术归因 |
| `AUTHOR_EMAIL` | 人类所有者 | 永久 | 法律归因、CV |

#### 2. Beads 记录

```json
{
  "id": "gt-xyz",
  "created_by": "gastown/crew/joe",
  "updated_by": "gastown/witness",
  "owner": "steve@example.com"
}
```

**字段用途**：

- `created_by`：从 `BD_ACTOR` 自动填充
- `updated_by`：追踪最后修改者
- `owner`：人类所有者，用于 CV 聚合

#### 3. 事件日志

```json
{
  "ts": "2025-01-15T10:30:00Z",
  "type": "sling",
  "actor": "gastown/crew/joe",
  "payload": {
    "bead": "gt-xyz",
    "target": "gastown/polecats/toast"
  }
}
```

### 环境变量自动设置

Gas Town 通过 `config.AgentEnv()` 函数统一设置环境变量：

**Polecat 环境**：

```bash
export GT_ROLE="polecat"
export GT_RIG="gastown"
export GT_POLECAT="toast"
export BD_ACTOR="gastown/polecats/toast"
export GIT_AUTHOR_NAME="gastown/polecats/toast"
export GT_ROOT="/home/user/gt"
export BEADS_DIR="/home/user/gt/gastown/.beads"
export BEADS_AGENT_NAME="gastown/toast"
export BEADS_NO_DAEMON="1"
```

**Crew 环境**：

```bash
export GT_ROLE="crew"
export GT_RIG="gastown"
export GT_CREW="joe"
export BD_ACTOR="gastown/crew/joe"
export GIT_AUTHOR_NAME="gastown/crew/joe"
export GT_ROOT="/home/user/gt"
export BEADS_DIR="/home/user/gt/gastown/.beads"
export BEADS_AGENT_NAME="gastown/joe"
export BEADS_NO_DAEMON="1"
```

---

## 第四部分：身份解析

### 程序化解析

斜杠格式支持可靠的程序化解析：

```go
// 身份到 BD_ACTOR 的转换
// Town 级别: mayor, deacon
// Rig 级别: {rig}/witness, {rig}/refinery
// 工作者: {rig}/crew/{name}, {rig}/polecats/{name}
```

### 解析结果

| 输入 | rig | role | name |
|------|-----|------|------|
| `mayor` | - | mayor | - |
| `deacon` | - | deacon | - |
| `gastown/witness` | gastown | witness | - |
| `gastown/refinery` | gastown | refinery | - |
| `gastown/crew/joe` | gastown | crew | joe |
| `gastown/polecats/toast` | gastown | polecat | toast |

### 审计查询

归因使强大的审计查询成为可能：

```bash
# 某个智能体的所有工作
bd audit --actor=gastown/crew/joe

# 某个 Rig 中的所有工作
bd audit --actor=gastown/*

# 所有 Polecat 的工作
bd audit --actor=*/polecats/*

# 按 Agent 的 Git 历史
git log --author="gastown/crew/joe"
```

---

## 第五部分：智能体身份 vs 人类身份

### 全局身份：Email

**设计决策**：Email 是人类的全局身份

**理由**：

```
理由一：Email 已在每一个 Git 提交中
├── 无需单独的"实体 bead"
└── 天然的唯一标识符

理由二：Email 可跨 Town 验证
├── Town A（家庭）和 Town B（工作）共享同一 Email
└── 实现 CV 跨工作空间聚合

理由三：Email 与法律/合规系统对接
├── 企业账户管理
└── 审计和合规要求
```

### 身份层级结构

```
steve@example.com                ← 全局身份（来自 git author）
├── Town A (home)                ← 工作空间
│   ├── gastown/crew/joe         ← 智能体执行者
│   └── gastown/polecats/toast   ← 智能体执行者
└── Town B (work)                ← 工作空间
    └── acme/polecats/nux        ← 智能体执行者
```

### 字段用途对照

| 字段 | 范围 | 用途 |
|------|------|------|
| `BD_ACTOR` | 本地（Town） | 智能体归因（调试） |
| `GIT_AUTHOR_EMAIL` | 全局 | 人类身份（CV） |
| `created_by` | 本地 | 谁创建的 bead |
| `owner` | 全局 | 谁拥有工作 |

**核心原则**：智能体执行，人类拥有

---

## 第六部分：Polecat 身份模型

### 持久身份 vs 临时会话

**核心洞察**：Polecats 有**持久身份但临时会话**

| 维度 | 持久 | 临时 |
|------|------|------|
| **身份** | ✅ Agent bead、CV 链、工作历史 | |
| **会话** | | ✅ Claude 实例、上下文窗口 |
| **沙盒** | | ✅ Git worktree、分支 |

### 类比：员工打卡

Polecats 类似于打卡上班的员工：

- 每次工作都是新会话（新的 tmux、新的 worktree）
- 但身份持续存在（工作记录、能力评估）
- 每次打卡都是新的开始，但累积历史不变

### 能力路由

基于归因的能力路由：

```bash
# 智能 A 有 Go 项目经验
gastown/polecats/claude-1  → 完成了 15 个 Go 任务，平均修订 1.2 次

# 智能 B 是新手
gastown/polecats/claude-2  → 完成了 2 个 Go 任务，平均修订 3.5 次

# 路由决策
新 Go 任务 → 分配给 gastown/polecats/claude-1
```

### 模型对比

归因使 A/B 模型对比成为可能：

```bash
# 标记智能体对应的模型
# gastown/polecats/claude-1 使用 Claude Opus
# gastown/polecats/gpt-1 使用 GPT-4

# 对比质量信号
bd stats --actor=gastown/polecats/claude-* --metric=revision-count
bd stats --actor=gastown/polecats/gpt-* --metric=revision-count
```

**指标解读**：更低的修订次数意味着更高的首次通过质量。

---

## 第七部分：技能累积系统

### 技能是推导出来的

**设计决策**：不从显式标签获取技能，而是从工作证据推导

```bash
# 按所有者聚合的所有工作（跨所有智能体）
git log --author="steve@example.com"
bd list --owner=steve@example.com

# 从证据推导技能
# - 接触的 .go 文件 → Go 技能
# - Issue 标签 → 领域技能
# - 提交模式 → 活动类型
```

### CV 构建

CV 是对工作证据的查询：

```bash
# 未来：联邦 CV 查询
bd cv steve@example.com
# 发现所有 Town，聚合工作，推导技能
```

### 多 Town 聚合

拥有多个 Town 的人类只有一个 CV：

```
steve@example.com
├── Town A（家庭）
│   ├── gastown/crew/joe
│   └── gastown/polecats/toast
└── Town B（工作）
    └── acme/polecats/nux

                ↓
        单一 CV（聚合所有证据）
```

---

## 第八部分：企业用例

### 合规与审计

**场景一：谁在过去 90 天内接触过敏感文件？**

```bash
git log --since="90 days ago" -- path/to/sensitive/file.go

# 输出：
# abc123 Fix security issue (gastown/crew/joe <steve@example.com>)
# def456 Update permissions (gastown/polecats/toast <steve@example.com>)
```

**场景二：特定智能体的所有变更**

```bash
bd audit --actor=gastown/polecats/toast --since=2025-01-01
```

### 性能追踪

**完成率统计**：

```bash
bd stats --group-by=actor

# 输出示例：
# gastown/polecats/toast: 47/50 完成 (94%)
# gastown/polecats/nux:    38/50 完成 (76%)
# gastown/crew/joe:        12/15 完成 (80%)
```

**平均周期时间**：

```bash
bd stats --actor=gastown/polecats/* --metric=cycle-time

# 输出示例：
# gastown/polecats/toast: 平均 2.3 小时
# gastown/polecats/nux:    平均 3.8 小时
```

### 质量对比

**修订次数统计**：

```bash
# 按智能体统计平均修订次数
bd stats --actor=gastown/polecats/* --metric=revision-count

# 输出示例：
# gastown/polecats/toast: 平均 1.2 次修订
# gastown/polecats/nux:    平均 2.1 次修订
```

**解读**：toast 产生更高质量的首次提交。

---

## 第九部分：设计原则总结

1. **智能体不是匿名的**：每个动作都被归因
2. **工作被拥有，而非被创作**：智能体创作，人类拥有
3. **归因是永久的**：Git 提交永久保存历史
4. **格式是可解析的**：支持程序化分析
5. **跨系统一致**：Git、Beads、事件中使用相同格式

---

## 第十部分：实现细节

### 身份解析函数

```go
// identityToBDActor 将守护进程身份转换为 BD_ACTOR 格式
// Town 级别: mayor, deacon
// Rig 级别: {rig}/witness, {rig}/refinery
// 工作者: {rig}/crew/{name}, {rig}/polecats/{name}
func identityToBDActor(identity string) string {
    // 实现细节...
}
```

### 手动覆盖

对于本地测试或调试：

```bash
export BD_ACTOR="gastown/crew/debug"
bd create --title="Test issue"
# 将显示 created_by: gastown/crew/debug
```

---

## 延伸阅读

- [Agent Identity and Attribution](../concepts/identity.md) - 英文原版
- [Polecat Lifecycle](../concepts/polecat-lifecycle.md) - 智能体生命周期
- [Mail Protocol](../design/mail-protocol.md) - 邮件通信协议
