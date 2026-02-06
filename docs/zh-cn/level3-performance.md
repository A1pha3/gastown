# 性能优化

> **Level 3** ⭐⭐⭐ | 进阶分析
>
> 本文档讲解 Gas Town 大规模部署时的性能优化策略。

## 学习目标

完成本章节学习后，你将能够：

### 基础目标
- 理解性能瓶颈的位置
- 掌握基本的优化配置
- 知道如何监控系统性能

### 进阶目标
- 分析并发控制策略
- 理解缓存机制
- 掌握资源分配优化

### 专家目标
- 设计可扩展的部署架构
- 实现自定义性能优化
- 调优大规模智能体部署

---

## 第一部分：性能瓶颈分析

### 瓶颈位置

```
┌─────────────────────────────────────────────────────┐
│                   Gas Town 系统                      │
├─────────────────────────────────────────────────────┤
│                                                     │
│  ┌─────────┐  ┌─────────┐  ┌─────────────────────┐ │
│  │Mayor    │  │Deacon   │  │Mail System         │ │
│  │决策速度  │  │监控频率  │  │邮件处理量           │ │
│  └─────────┘  └─────────┘  └─────────────────────┘ │
│       ↓            ↓                  ↓             │
│  ┌─────────────────────────────────────────────┐   │
│  │          Dolt Server (中心瓶颈)             │   │
│  │  • 查询延迟  • 并发连接  • 锁竞争           │   │
│  └─────────────────────────────────────────────┘   │
│       ↓                                                │
│  ┌─────────┐  ┌─────────┐  ┌─────────────────────┐ │
│  │Witness  │  │Refinery │  │Polecats            │ │
│  │Patrol   │  │合并队列  │  │生成速度            │ │
│  └─────────┘  └─────────┘  └─────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

### 性能指标

| 指标 | 健康值 | 警告值 | 危险值 |
|------|--------|--------|--------|
| **Dolt 查询延迟** | < 10ms | 10-50ms | > 50ms |
| **Polecat 生成时间** | < 5s | 5-15s | > 15s |
| **邮件处理延迟** | < 1s | 1-5s | > 5s |
| **Convoy 刷新时间** | < 2s | 2-10s | > 10s |
| **Patrol 周期时间** | < 30s | 30-60s | > 60s |

---

## 第二部分：Dolt 性能优化

### 连接池配置

```bash
# 设置最大连接数
gt config dolt.max_open_conns 25
gt config dolt.max_idle_conns 5
gt config dolt.conn_max_lifetime 1h
gt config dolt.conn_max_idle_time 10m
```

### 批量操作

```bash
# 不推荐：逐个查询
for id in gt-abc gt-def gt-ghi; do
    bd show $id
done

# 推荐：批量查询
bd show gt-abc gt-def gt-ghi

# 批量更新
bd update --batch file.json
```

### 索引优化

```sql
-- 添加索引以加速常用查询
CREATE INDEX idx_status ON issues(status);
CREATE INDEX idx_created_by ON issues(created_by);
CREATE INDEX idx_rig_status ON operational_state(rig, status);
CREATE INDEX idx_timestamp ON heartbeats(timestamp);
```

### 查询优化

```bash
# 使用过滤减少数据传输
bd list --status=open --type=task --limit=100

# 使用投影只获取需要的字段
bd show gt-abc --fields=id,title,status
```

---

## 第三部分：并发控制

### 并发限制

```bash
# 设置最大并发 Polecat 数
gt config polecat.max_concurrent 10

# 设置 bd 命令并发限制
gt config bd.max_concurrent 10

# 设置 sling 并发限制
gt config sling.max_concurrent 5
```

### 速率限制

```bash
# 设置请求速率
gt config rate.requests_per_second 100
gt config rate.burst_size 50
```

### 队列管理

```bash
# 查看队列状态
gt queue list

# 清空队列
gt queue clear --type=sling

# 调整队列大小
gt config sling.queue_size 100
```

---

## 第四部分：缓存策略

### 分层缓存

```
L1: 应用内存缓存
├── 热点 bead（当前 Convoy）
├── TTL: 5 分钟
└── 容量: 1000 条

L2: Dolt 查询缓存
├── 常用查询（agent list、status）
├── TTL: 15 分钟
└── 容量: 10000 条

L3: Dolt 页面缓存
├── 数据库内置
└── 自动管理
```

### 缓存配置

```bash
# 启用缓存
gt config cache.enabled true

# 设置缓存大小
gt config cache.memory_size 256MB
gt config.cache.ttl 300

# 缓存预热
gt cache warmup --type=agent
gt cache warmup --type=status
```

### 缓存失效

```bash
# 手动刷新缓存
gt cache refresh --type=agent

# 清除所有缓存
gt cache clear --all

# 清除特定缓存
gt cache clear --type=bead
```

---

## 第五部分：Tmux 性能优化

### Tmux 配置优化

```bash
# 使用 256 色模式减少渲染
export TMUX_ITERM="screen-256color"

# 限制历史行数
tmux set-option -g history-limit 1000

# 减少状态更新频率
tmux set-option -g status-interval 5

# 禁用不必要的活动监控
tmux set-option -g monitor-activity off
```

### 会话管理

```bash
# 列出所有会话
gt tmux list

# 清理死会话
gt tmux cleanup --dead

# 限制活跃会话数
gt config tmux.max_sessions 50
```

---

## 第六部分：Polecat 生成优化

### Worktree 预创建

```bash
# 预创建 worktree 池
gt worktree pool create --size=10 --rig=gastown

# 使用预创建的 worktree
gt sling gt-abc gastown --use-pool
```

### 并行生成

```bash
# 并行生成多个 Polecats
gt sling gt-abc gt-def gt-ghi --rig=gastown --parallel

# 设置并行度
gt config sling.parallelism 3
```

### 资源限制

```bash
# 设置 Polecat 资源限制
gt config polecat.max_memory 4GB
gt config polecat.max_cpu 2
```

---

## 第七部分：Refinery 优化

### 合并队列优化

```bash
# 设置队列大小
gt refinery queue.size 20

# 并行合并
gt refinery parallelism 3

# 批量合并
gt refinery merge --batch
```

### 验证优化

```bash
# 跳过不必要的验证
gt refinery verify --skip=duplicate

# 并行测试
gt refinery test --parallel

# 缓存测试结果
gt refinery test --cache
```

---

## 第八部分：监控与调优

### 性能监控

```bash
# 启用性能监控
gt monitor enable --type=performance

# 查看性能指标
gt metrics show

# 导出指标
gt metrics export --format=prometheus
```

### 性能分析

```bash
# 分析瓶颈
gt analyze --type=dolt
gt analyze --type=mail
gt analyze --type=polecat

# 生成报告
gt report --type=performance --output=report.html
```

### 自动调优

```bash
# 启用自动调优
gt autotune enable

# 调优间隔
gt config autotune.interval 1h

# 调优范围
gt config autotune.scope dolt,cache,concurrency
```

---

## 第九部分：扩展模式

### 水平扩展

```bash
# 增加 Rig 数量
gt rig add project1 https://github.com/user/project1
gt rig add project2 https://github.com/user/project2

# 每个 Rig 独立运行 Witness/Refinery
```

### 垂直扩展

```bash
# 增加单 Rig 中的 Polecat 数量
gt config polecat.max_per_rig 20

# 增加资源
gt config rig.memory 16GB
gt config rig.cpu 8
```

### 联邦扩展

```bash
# 创建多个 Town
gt install ~/gt-town1
gt install ~/gt-town2

# 同步联邦数据
gt federation sync
```

---

## 第十部分：性能基准

### 基准测试

```bash
# 运行基准测试
gt benchmark --type=dolt
gt benchmark --type=polecat
gt benchmark --type=convoy

# 对比测试
gt benchmark compare --before=config1 --after=config2
```

### 性能目标

| 场景 | Polecat 数 | Convoy 数 | 响应时间 |
|------|-----------|-----------|----------|
| **小型** | < 10 | < 5 | < 1s |
| **中型** | 10-30 | 5-20 | < 5s |
| **大型** | 30-100 | 20-50 | < 10s |
| **超大型** | > 100 | > 50 | < 30s |

---

## 延伸阅读

- [扩展性与联邦化](./level4-scalability.md) - 大规模部署
- [数据平面设计](./level4-data-planes.md) - 数据层优化
- [Dolt 存储架构](../design/dolt-storage.md) - 存储性能
