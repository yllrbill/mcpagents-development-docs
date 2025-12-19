# Architecture Decision Records (ADR)

> 记录 MCPAgents 项目中的关键架构决策，避免未来反复争论和遗忘设计意图。

---

## ADR 索引

| 编号 | 标题 | 状态 | 日期 |
|------|------|------|------|
| [ADR-0001](0001-single-writer-daemon.md) | 单写者 Daemon 架构 | ✅ 已采纳 | 2025-12 |
| [ADR-0002](0002-sse-event-streaming.md) | SSE 事件流协议 | ✅ 已采纳 | 2025-12 |
| [ADR-0003](0003-cas-state-machine.md) | CAS 状态机并发控制 | ✅ 已采纳 | 2025-12 |

---

## ADR 模板

每个 ADR 遵循 Michael Nygard 模板：

```markdown
# ADR-XXXX: 标题

## 状态
Proposed / Accepted / Deprecated / Superseded by ADR-YYYY

## 上下文
我们面临什么问题？需要做什么决策？

## 决策
我们决定采用什么方案？

## 理由
为什么选择这个方案而不是其他备选？

## 后果
- 正面影响
- 负面影响
- 需要注意的事项

## 备选方案（可选）
考虑过但未采用的其他方案及其原因。
```

---

## ADR 编号规则

- 格式：`XXXX-short-title.md`
- 编号：四位数字，从 0001 开始递增
- 一旦创建，编号不复用
- Superseded 的 ADR 保留，在头部标注被谁取代

---

## 何时写 ADR

- 引入新的核心架构模式
- 选择特定技术/库/协议
- 定义系统边界和约束
- 改变已有架构决策

---

## 相关文档

- [00_DOC_RULES.md](../00_DOC_RULES.md) - 文档规范
- [20_CORE_MODULE.md](../20_CORE_MODULE.md) - 核心模块说明
