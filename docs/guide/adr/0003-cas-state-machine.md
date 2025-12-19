# ADR-0003: CAS 状态机并发控制

## 状态

**Accepted** (2025-12)

---

## 上下文

MCPAgents 的 Run 生命周期有多个状态转换路径：

```
queued → running → succeeded
queued → running → failed
queued → running → cancelling → canceled
queued → canceled
queued → timedOut
running → timedOut
```

这些转换可能被多个并发操作触发：

1. **Worker 线程**: pick up 任务、完成任务
2. **超时检查器**: 定时检查队列/执行超时
3. **用户取消**: API 请求取消正在执行的 Run
4. **LLM 回调**: 流式响应完成

需要一种机制确保状态转换的原子性和正确性。

---

## 决策

**采用 Compare-And-Set (CAS) 模式**进行状态转换。

### 核心原则

1. **状态转换只能通过 CAS 操作**
2. **SQL WHERE 条件包含当前状态检查**
3. **返回 affected rows 判断是否成功**
4. **失败不重试，由调用者处理竞态**

### 实现示例

```dart
// run_repository.dart
Future<bool> claimForExecution(String runId) async {
  _db.execute('''
    UPDATE runs
    SET status = 'running', started_at = ?
    WHERE id = ? AND status = 'queued'
  ''', [DateTime.now().toIso8601String(), runId]);
  return _db.updatedRows > 0;
}

Future<bool> markSucceeded(String runId) async {
  _db.execute('''
    UPDATE runs
    SET status = 'succeeded', completed_at = ?
    WHERE id = ? AND status IN ('running', 'cancelling')
  ''', [DateTime.now().toIso8601String(), runId]);
  return _db.updatedRows > 0;
}
```

### 状态转换表

| 当前状态 | 目标状态 | 触发者 | CAS 条件 |
|----------|----------|--------|----------|
| `queued` | `running` | Worker | `status = 'queued'` |
| `queued` | `canceled` | 用户 | `status = 'queued'` |
| `queued` | `timedOut` | 超时检查 | `status = 'queued'` |
| `running` | `succeeded` | LLM 完成 | `status IN ('running', 'cancelling')` |
| `running` | `failed` | 错误/fallback 耗尽 | `status IN ('running', 'cancelling')` |
| `running` | `cancelling` | 用户 | `status = 'running'` |
| `running` | `timedOut` | 超时检查 | `status = 'running'` |
| `cancelling` | `canceled` | Worker 确认 | `status = 'cancelling'` |

---

## 理由

### 对比备选方案

| 方案 | 优点 | 缺点 |
|------|------|------|
| **CAS (采用)** | 简单、无锁、SQLite 原生支持 | 竞态时需要调用者处理失败 |
| 悲观锁 (SELECT FOR UPDATE) | 保证串行 | SQLite 不支持行锁 |
| 乐观锁 (version 字段) | 通用 | 需要额外字段，CAS 已足够 |
| 应用层锁 (Mutex) | 直观 | 跨进程不可靠，Daemon 重启丢失 |
| 消息队列 | 解耦 | 过度设计，引入外部依赖 |

### 为什么 CAS 足够？

1. **SQLite 写入串行**: SQLite 的写锁保证了 UPDATE 原子性
2. **状态机语义清晰**: 每个转换的前置条件明确
3. **失败是合法场景**: 竞态失败说明状态已被其他操作改变，是预期行为

### 为什么不用分布式锁？

1. 单写者架构下只有一个 Daemon 进程
2. 进程内并发用 SQLite 事务足够
3. 引入 Redis/ZooKeeper 过度设计

---

## 后果

### 正面影响

1. **无死锁风险**: 不使用悲观锁，不会死锁
2. **高并发友好**: 竞态操作快速失败，不阻塞
3. **实现简单**: 标准 SQL，无外部依赖
4. **可审计**: 每次状态变更都有对应事件

### 负面影响

1. **竞态处理**: 调用者需要处理 CAS 失败
   - 缓解：失败是合法场景，通常忽略或记录日志
2. **多步骤操作**: 复杂流程需要事务包装
   - 缓解：使用 SQLite 事务

### 需要注意

- CAS 失败不应该重试（可能导致 ABA 问题）
- 终态（succeeded/failed/canceled/timedOut/rejected）不可再转换
- `running → running` 的 retry/fallback 不改变状态，只发事件

---

## 关键不变量

| 不变量 | 保证方式 |
|--------|----------|
| 状态只能前进，不能后退 | CAS 条件限制 |
| 终态不可变 | 终态不在 WHERE 条件中 |
| 每个 Run 只有一个执行者 | `queued → running` CAS 保证 |
| 取消和完成可能竞态 | `cancelling` 中间态处理 |

---

## 代码位置

- Run 仓库: `apps/mcpagentsd/lib/src/services/run_repository.dart`
- Run 服务: `apps/mcpagentsd/lib/src/services/run_service.dart`
- 状态定义: `packages/mcpagents_core/lib/src/models/run.dart`

---

## 参考

- [Compare-and-swap (Wikipedia)](https://en.wikipedia.org/wiki/Compare-and-swap)
- [Optimistic Concurrency Control](https://martinfowler.com/eaaCatalog/optimisticOfflineLock.html)
- [SQLite Transactions](https://www.sqlite.org/lang_transaction.html)
