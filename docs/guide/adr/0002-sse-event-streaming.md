# ADR-0002: SSE 事件流协议

## 状态

**Accepted** (2025-12)

---

## 上下文

MCPAgents 需要将 LLM 响应实时推送给客户端：

1. **流式响应**: LLM 逐 token 输出，需要实时显示
2. **状态变更**: Run 生命周期事件需要即时通知
3. **断线续传**: 网络不稳定时需要恢复丢失的事件

需要选择一个实时通信协议。

---

## 决策

**采用 Server-Sent Events (SSE)** 作为实时通信协议。

### 事件格式

```
id: run_abc123:1
event: run.started
data: {"run_id":"run_abc123","seq":1,...}

id: run_abc123:2
event: llm.delta
data: {"run_id":"run_abc123","seq":2,"data":{"delta":"Hello"}}

id: run_abc123:3
event: run.succeeded
data: {"run_id":"run_abc123","seq":3,...}
```

### 断线续传

```bash
# 方式一：Query Parameter
GET /v1/runs/{id}/events?after=run_abc123:5

# 方式二：Last-Event-ID Header (SSE 标准)
GET /v1/runs/{id}/events
Last-Event-ID: run_abc123:5
```

### 心跳机制

```
:heartbeat

```
- 间隔：15 秒
- 格式：SSE 注释行（不增加 seq）

---

## 理由

### 对比备选方案

| 特性 | SSE | WebSocket | gRPC Streaming | HTTP Long Polling |
|------|-----|-----------|----------------|-------------------|
| 单向/双向 | 单向 (S→C) | 双向 | 双向 | 单向 (S→C) |
| 协议 | HTTP/1.1 | 独立协议 | HTTP/2 | HTTP/1.1 |
| 浏览器原生 | ✅ EventSource | ✅ WebSocket | ❌ 需要 grpc-web | ❌ 需要封装 |
| 断线续传 | ✅ Last-Event-ID | ❌ 需自实现 | ❌ 需自实现 | ❌ 需自实现 |
| 代理兼容 | ✅ 标准 HTTP | ⚠️ 需要升级 | ⚠️ 需要 HTTP/2 | ✅ 标准 HTTP |
| 实现复杂度 | 低 | 中 | 高 | 低 |

### 为什么选择 SSE？

1. **协议标准化**: W3C 标准，浏览器原生 EventSource API
2. **断线续传**: 内置 Last-Event-ID 机制，无需自实现
3. **代理友好**: 标准 HTTP，穿透企业代理/CDN
4. **实现简单**: Dart shelf 原生支持
5. **符合场景**: 我们只需要服务端→客户端的单向推送

### 为什么不选 WebSocket？

1. 需要单独的协议升级握手
2. 断线续传需要自己实现
3. 某些企业代理会阻断 WebSocket
4. 双向通信能力我们不需要（客户端用 HTTP POST）

### 为什么不选 gRPC Streaming？

1. 需要 HTTP/2，某些环境不支持
2. 浏览器需要 grpc-web 转换层
3. Dart 服务端 gRPC 支持不如 HTTP
4. 调试困难（二进制协议）

---

## 后果

### 正面影响

1. **浏览器兼容**: Web 版可直接使用 EventSource
2. **断线恢复**: Last-Event-ID 自动处理
3. **调试友好**: 纯文本协议，curl 可直接测试
4. **CDN/代理穿透**: Cloudflare Tunnel 直接支持

### 负面影响

1. **单向限制**: 客户端请求仍需 HTTP POST
   - 缓解：这正是我们需要的模式
2. **连接数限制**: 浏览器对同域 SSE 连接数有限制（6 个）
   - 缓解：单个连接可复用多个 Run
3. **无消息确认**: 服务端不知道客户端是否收到
   - 缓解：终态事件持久化到 DB，重连时补发

### 需要注意

- Event ID 不能包含 `\0`, `\n`, `\r`（SSE 规范要求）
- 心跳间隔要小于代理超时（通常 30-60 秒）
- 大事件需要考虑分片或引用

---

## 代码位置

- SSE 推送: `apps/mcpagentsd/lib/src/router.dart`
- 事件服务: `apps/mcpagentsd/lib/src/services/event_service.dart`
- 事件定义: `packages/mcpagents_core/lib/src/events/event.dart`
- GUI 重连: `lib/provider/sse_subscription_mixin.dart`

---

## 参考

- [W3C Server-Sent Events](https://html.spec.whatwg.org/multipage/server-sent-events.html)
- [MDN EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)
- [RFC 8895 - Server-Sent Events](https://www.rfc-editor.org/rfc/rfc8895.html)
