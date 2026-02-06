# Gas Town 中文文档

Gas Town 是一个多智能体编排系统，用于协调多个 AI 编码助手（Claude、Codex、Gemini、Cursor）协同工作。

## 文档等级说明

本文档采用**四级渐进式学习体系**，帮助不同水平的读者从入门到精通：

| 等级 | 定位 | 目标读者 | 认知模式 |
|------|------|----------|----------|
| **Level 1** ⭐ | 入门教程 | 初次接触 Gas Town 的用户 | 模仿 → 执行 |
| **Level 2** ⭐⭐ | 核心概念 | 系统学习核心概念和用法 | 理解 → 应用 |
| **Level 3** ⭐⭐⭐ | 进阶分析 | 深入原理，解决复杂问题 | 分析 → 优化 |
| **Level 4** ⭐⭐⭐⭐ | 专家设计 | 架构设计，制定技术决策 | 创造 → 设计 |

## 文档目录

### Level 1 入门教程 ⭐

| 文档 | 说明 | 预计时间 |
|------|------|----------|
| [快速入门](./level1-quickstart.md) | 5 分钟上手 Gas Town | 5 分钟 |
| [第一个任务](./level1-first-task.md) | 完成第一个开发任务 | 15 分钟 |
| [基础命令](./level1-basic-commands.md) | 常用命令速查 | 10 分钟 |

### Level 2 核心概念 ⭐⭐

| 文档 | 说明 | 预计时间 |
|------|------|----------|
| [架构概览](./level2-architecture.md) | 两级架构与核心组件 | 30 分钟 |
| [Beads 系统](./level2-beads.md) | 工作追踪的核心数据结构 | 25 分钟 |
| [Agent 生命周期](./level2-agent-lifecycle.md) | Polecat、Witness、Refinery | 30 分钟 |
| [Mail 通信](./level2-mail.md) | 智能体间异步通信 | 20 分钟 |

### Level 3 进阶分析 ⭐⭐⭐

| 文档 | 说明 | 预计时间 |
|------|------|----------|
| [Molecules 工作流](./level3-molecules.md) | 多步骤工作流的设计与实现 | 45 分钟 |
| [分布式协调](./level3-distributed-coordination.md) | Mayor/Deacon 协调机制 | 40 分钟 |
| [故障恢复](./level3-fault-recovery.md) | NDI 原则与容错设计 | 35 分钟 |
| [性能优化](./level3-performance.md) | 大规模智能体部署 | 30 分钟 |

### Level 4 专家设计 ⭐⭐⭐⭐

| 文档 | 说明 | 预计时间 |
|------|------|----------|
| [架构设计决策](./level4-architecture-decisions.md) | 设计权衡与历史演进 | 60 分钟 |
| [数据平面设计](./level4-data-planes.md) | 三层数据平面架构 | 50 分钟 |
| [身份与归因](./level4-identity.md) | 通用归因系统设计 | 40 分钟 |
| [扩展性设计](./level4-scalability.md) | 联邦与插件系统 | 55 分钟 |

## 学习路径建议

### 路径一：用户路径

**目标**：使用 Gas Town 完成日常开发工作

```
Level 1 → Level 2 (选择所需章节)
```

### 路径二：开发者路径

**目标**：参与 Gas Town 开发或二次开发

```
Level 1 → Level 2 → Level 3 → Level 4 (按需阅读)
```

### 路径三：架构师路径

**目标**：深入理解设计原理，构建类似系统

```
Level 1 (快速浏览) → Level 2 → Level 3 → Level 4 (完整阅读)
```

## 核心概念速览

### 三大核心原则

| 原则 | 英文 | 含义 |
|------|------|------|
| **MEOW** | Molecular Expression of Work | 将大型目标分解为可追踪的步骤 |
| **GUPP** | Gas Town Universal Propulsion Principle | "如果你的 Hook 上有工作，你必须执行它" |
| **NDI** | Nondeterministic Idempotence | 通过编排实现最终一致性 |

### 两级架构

```
┌─────────────────────────────────────────────────────┐
│                    Town (~/gt/)                      │
│  ┌─────────┐  ┌─────────┐  ┌─────────────────────┐  │
│  │ Mayor   │  │ Deacon  │  │ Dogs                │  │
│  │ 协调者   │  │ 守护进程 │  │ 维护智能体           │  │
│  └─────────┘  └─────────┘  └─────────────────────┘  │
└─────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        ▼                 ▼                 ▼
┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│   Rig A     │  │   Rig B     │  │   Rig C     │
│ ┌─────────┐ │  │ ┌─────────┐ │  │ ┌─────────┐ │
│ │Witness  │ │  │ │Witness  │ │  │ │Witness  │ │
│ │Refinery │ │  │ │Refinery │ │  │ │Refinery │ │
│ │Polecats │ │  │ │Polecats │ │  │ │Polecats │ │
│ └─────────┘ │  │ └─────────┘ │  │ └─────────┘ │
└─────────────┘  └─────────────┘  └─────────────┘
```

### 核心术语

| 术语 | 说明 |
|------|------|
| **Town** | 工作空间根目录（如 `~/gt/`），协调所有智能体 |
| **Rig** | 项目容器，包装一个 Git 仓库 |
| **Polecat** | 短命的工作智能体，完成特定任务后自我清理 |
| **Witness** | Rig 级别的监控智能体 |
| **Refinery** | 管理合并队列的智能体 |
| **Bead** | 存储在 JSONL 格式中的原子工作单元 |
| **Molecule** | 可在智能体重启后幸存的多步骤工作流 |
| **Hook** | 智能体的主要工作队列（特殊的 pinned bead） |
| **Convoy** | 将相关 beads 打包分配给多个智能体的工作单元 |

## 常用命令速查

### 工作空间管理

```bash
gt install ~/gt --git    # 初始化工作空间
gt rig add <name> <repo> # 添加项目
gt crew add <name>       # 创建成员工作区
```

### 智能体操作

```bash
gt mayor attach          # 启动 Mayor 会话
gt sling <bead-id> <rig> # 分配工作给智能体
gt agents                # 列出活跃智能体
gt done                  # 完成工作（Polecat 必须执行）
```

### 工作追踪

```bash
gt convoy create <name> [beads...]  # 创建工作追踪单元
gt convoy list                     # 列出所有 convoy
gt convoy show                     # 显示详情
```

### Beads 操作

```bash
bd formula list           # 列出可用公式
bd cook <formula>         # 执行公式
bd mol pour <formula>     # 创建可追踪实例
bd mol current            # 查看当前进度
```

## 贡献指南

欢迎对本中文文档提出改进建议：

1. Fork 本仓库
2. 创建翻译/改进分支：`git checkout -b zh-cn-improvement`
3. 提交 Pull Request

## 参考资源

- [英文文档](../) - 官方英文文档
- [术语表](./glossary.md) - 完整术语参考
- [命令参考](../reference.md) - 完整命令手册
