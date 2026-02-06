# Beads 系统

> **Level 2** ⭐⭐ | 核心概念
>
> 本文档深入讲解 Beads 系统——Gas Town 工作追踪的核心数据结构。

## 学习目标

完成本章节学习后，你将能够：

### 基础目标
- 理解 Bead 的数据结构和格式
- 掌握 Bead ID 的命名规则
- 知道如何创建、查询和管理 Beads

### 进阶目标
- 理解两级 Beads 架构
- 掌握路由系统的工作原理
- 知道不同类型 Beads 的用途

---

## 第一部分：什么是 Bead？

### 定义

**Bead** 是存储在 JSONL 格式中的原子工作单元。Bead 是 Gas Town 工作追踪系统的核心，可以代表 issues、任务、epics 或任何可追踪的工作项。

### 为什么叫 Bead？

```
Bead（珠子）→ 串联成链 → Molecules（分子）→ 形成工作流
```

就像珠子可以串成项链一样，Beads 可以串联成 Molecules（工作流）。

### Bead vs Issue

| 术语 | 说明 |
|------|------|
| **Bead** | 底层数据格式（JSONL） |
| **Issue** | 工作项的语义表示 |
| **关系** | Issue 是一种类型的 Bead |

在 Gas Town 中，这两个术语经常互换使用。

---

## 第二部分：Bead ID 格式

### 格式规范

```
{prefix}{id}
```

| 部分 | 说明 | 示例 |
|------|------|------|
| **prefix** | 3-5 字符前缀，表示来源或 Rig | `gt-`, `hq-`, `bd-` |
| **id** | 5 字符的字母数字 ID | `abc12`, `x7k2m` |

### 完整示例

```
gt-abc12    # Gastown 项目的 bead
hq-mayor    # Town 级别的 Mayor bead
bd-x7k2m    # Beads 项目的 bead
```

### 前缀分配

| 层级 | 位置 | 前缀 | 说明 |
|------|------|------|------|
| **Town** | `~/gt/.beads/` | `hq-` | 跨 Rig 协调 |
| **Rig** | `<rig>/.beads/` | 项目前缀 | 项目特定工作 |

---

## 第三部分：Bead 数据结构

### JSONL 格式

每个 Bead 是 JSONL 文件中的一行：

```json
{
  "id": "gt-abc12",
  "type": "task",
  "title": "实现用户认证",
  "status": "in_progress",
  "created_by": "gastown/crew/joe",
  "created_at": "2025-01-15T10:30:00Z",
  "updated_at": "2025-01-15T14:20:00Z",
  "updated_by": "gastown/polecats/toast",
  "owner": "steve@example.com",
  "metadata": {
    "priority": "high",
    "complexity": "medium",
    "estimated_hours": 4
  }
}
```

### 核心字段

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | string | 唯一标识符 |
| `type` | string | 类型：task, issue, epic, bug 等 |
| `title` | string | 简短描述 |
| `status` | string | 状态：open, in_progress, closed 等 |
| `created_by` | string | 创建者（BD_ACTOR 格式） |
| `created_at` | timestamp | 创建时间 |
| `owner` | string | 所有者 Email |

### Bead 类型

| 类型 | 说明 | 生命周期 |
|------|------|----------|
| `task` | 可执行的任务 | 短期 |
| `issue` | 问题或缺陷 | 直到解决 |
| `epic` | 大型功能集合 | 长期 |
| `bug` | 缺陷报告 | 直到修复 |
| `message` | 邮件消息 | 短期 |

---

## 第四部分：两级 Beads 架构

### Town 级 Beads

**位置**：`~/gt/.beads/`

**前缀**：`hq-*`

**用途**：

```
├── Mayor 邮件和消息
├── Convoy 协调（跨 rig 批量工作）
├── 战略问题和决策
├── Town 级智能体 beads（Mayor、Deacon）
└── 角色定义 beads（全局模板）
```

### Rig 级 Beads

**位置**：`<rig>/mayor/rig/.beads/`

**前缀**：项目前缀

**用途**：

```
├── 项目的 bugs、features、tasks
├── 合并请求和代码审查
├── 项目特定的 molecules
└── Rig 级智能体 beads（Witness、Refinery、Polecats）
```

### 架构图

```
~/gt/
├── .beads/                    # Town 级
│   ├── issues.jsonl           # hq-* 前缀
│   └── routes.jsonl           # 路由表
│
└── gastown/                   # Rig
    └── mayor/rig/
        └── .beads/             # Rig 级
            └── issues.jsonl   # gt-* 前缀
```

---

## 第五部分：路由系统

### 路由表

`routes.jsonl` 文件将 bead ID 前缀映射到物理位置：

```jsonl
{"prefix":"hq-","path":"."}
{"prefix":"gt-","path":"gastown/mayor/rig"}
{"prefix":"bd-","path":"beads/mayor/rig"}
```

### 路由解析

当执行 `bd show gt-abc12` 时：

```
1. 提取前缀: gt-
2. 查询路由表: gastown/mayor/rig
3. 定位文件: ~/gt/gastown/mayor/rig/.beads/issues.jsonl
4. 返回 bead 数据
```

### 透明路由

```bash
# 这些命令会自动路由到正确的位置
bd show hq-mayor     # → ~/gt/.beads/
bd show gt-abc       # → ~/gt/gastown/mayor/rig/.beads/
bd show bd-xyz       # → ~/gt/beads/mayor/rig/.beads/
```

---

## 第六部分：Beads 命令

### 创建 Beads

```bash
# 基本创建
bd create --title="实现用户登录"

# 指定类型
bd create --title="修复登录 bug" --type=bug

# 带元数据
bd create --title="性能优化" --type=task \
  --metadata='{"priority": "high", "complexity": "hard"}'

# 创建 Epic
bd create --title="用户认证系统" --type=epic
```

### 查询 Beads

```bash
# 列出所有
bd list

# 按类型过滤
bd list --type=task
bd list --type=bug

# 按状态过滤
bd list --status=open
bd list --status=in_progress

# 按创建者过滤
bd list --creator="gastown/crew/joe"

# 搜索
bd search "登录"
bd search --type=task "API"
```

### 更新 Beads

```bash
# 更新状态
bd update gt-abc12 --status=closed

# 更新标题
bd update gt-abc12 --title="新标题"

# 添加/更新元数据
bd update gt-abc12 --metadata='{"completed_at": "2025-01-15T16:00:00Z"}'
```

### 删除 Beads

```bash
# 软删除（标记为删除）
bd delete gt-abc12

# 硬删除（永久移除）
bd delete gt-abc12 --hard
```

---

## 第七部分：Agent Beads

### 什么是 Agent Bead？

Agent Beads 跟踪每个智能体的生命周期状态。

| 智能体类型 | Bead 位置 | Bead ID 格式 |
|------------|-----------|--------------|
| Mayor | Town | `hq-mayor` |
| Deacon | Town | `hq-deacon` |
| Dogs | Town | `hq-dog-<name>` |
| Witness | Rig | `<prefix>-<rig>-witness` |
| Refinery | Rig | `<prefix>-<rig>-refinery` |
| Polecats | Rig | `<prefix>-<rig>-polecat-<name>` |

### Role Beads

Role Beads 是存储在 Town beads 中的全局模板：

```
hq-mayor-role     # Mayor 角色定义
hq-deacon-role    # Deacon 角色定义
hq-witness-role   # Witness 角色定义
hq-polecat-role   # Polecat 角色定义
```

每个 agent bead 通过 `role_bead` 字段引用其角色 bead。

---

## 第八部分：最佳实践

### Bead 命名

```bash
# 好的命名
bd create --title="实现用户登录功能"          # 清晰、具体
bd create --title="修复 API 超时问题"        # 包含问题类型

# 不好的命名
bd create --title="做点什么"                 # 模糊
bd create --title="工作"                     # 无信息量
```

### 类型选择

| 场景 | 使用类型 |
|------|----------|
| 可执行的独立任务 | `task` |
| 需要修复的问题 | `bug` |
| 大型功能分解 | `epic` |
| 智能体通信 | `message` |
| 文档任务 | `task` + 标签 |

### 状态转换

```
open → in_progress → review → closed
                      ↓
                   rejected
```

---

## 延伸阅读

- [Molecules 工作流](./level3-molecules.md) - Beads 串联成工作流
- [身份与归因](./level4-identity.md) - created_by 字段详解
- [Dolt 存储架构](../design/dolt-storage.md) - Beads 存储后端
