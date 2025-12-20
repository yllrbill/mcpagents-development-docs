# MCPAgents Session Status (Public)

> 脱敏版会话状态，用于跨对话恢复进展。
> 完整内部版本见私有仓 `docs/SESSION_STATE.md`。

---

## Metadata

| 项目 | 值 |
|------|-----|
| 当前里程碑 | M1 — 本机"好用"+全流程稳定 |
| 会话编号 | 2025-12-20_m1_ci_hardening |
| 更新时间 | 2025-12-20 22:30 (UTC+8) |
| 当前分支 | feature/m0-stability-suite |
| 最新 commit | pending (M1 CI changes) |

---

## TL;DR (30 秒恢复区)

### 已完成

- **M0 稳定性验证** 6 项核心测试全部通过
- **EchoClient mock provider** — 解决 LLM 依赖问题，测试完全确定性
- **并发压测完成** — 确认系统瓶颈和最佳运行区间
  - 系统默认 maxConcurrency=5
  - 最佳吞吐区间：4-8 并发（峰值 21.1 runs/sec）
  - Server P95 延迟稳定在 ~200ms
- **SSE 断线续传测试** — JSONL 验证通过，seq 单调递增
- **30 分钟单写者压力测试通过**
  - 3772 runs，100% 成功率
  - 内存增长仅 0.04 MB（无泄漏）
  - 吞吐量稳定 2.09 runs/sec
- 测试脚本：
  - `m0_stability_suite.ps1/.sh` — 统一回归入口（双平台）
  - `test_echo_concurrency.ps1` — 并发压测
  - `test_sse_replay.ps1` — SSE 重连测试
  - `test_30min_stress.ps1` — 30 分钟长跑测试
- **M1 Task 1 CI 更新**
  - `daemon-ci.yml` 添加 M0 Stability 短版 PR 门禁（Linux + Windows）
  - `m0-nightly-stress.yml` 新增 30 分钟长跑 nightly workflow
  - 并发结论写入 `config_reference.md`

### 进行中

- M1 Task 1: 合并 feature/m0-stability-suite 到 main（待 PR）
- M1 Task 2: Router 热加载 + 决策可解释
- M1 Task 3: Tool Sandbox + UI Automation

### 阻塞

- 无

### 下一步 (唯一最高优先级)

创建 PR 合并 M0 稳定性套件到 main，然后开始 M1 Task 2

---

## 关键变更

| Commit | 说明 |
|--------|------|
| 81343fc | feat(m0): add 30-minute single-writer stress test |
| 2de2719 | docs(session): update SESSION_STATE for M0 concurrency test |
| 6a1a740 | feat(m0): add concurrency stress test and SSE replay test |
| b7386c0 | feat(m0): add echo provider for deterministic testing |
| f0fc433 | feat(m0): add M0 stability regression suite |

---

## 性能基准（EchoClient 压测）

| 并发数 | 吞吐量 | Server P95 | Client P95 | 备注 |
|--------|--------|------------|------------|------|
| 4 | 16.4 runs/s | 200ms | 231ms | 最佳性能 |
| 8 | 21.1 runs/s | 190ms | 481ms | 峰值吞吐 |
| 16 | 7.0 runs/s | 202ms | 2284ms | 队列拥塞 |

## 30 分钟长跑压力测试

| 指标 | 结果 |
|------|------|
| 总 Runs | 3772 |
| 成功率 | 100% |
| 内存增长 | 0.04 MB |
| 吞吐量 | 2.09 runs/sec |
| Server P95 | 233ms |
| Server P99 | 247ms |

---

## 验证命令

```powershell
# 启动 daemon
cd apps\mcpagentsd
dart run bin/main.dart --port 8787

# M0 稳定性套件（确定性测试）
powershell -ExecutionPolicy Bypass -File scripts\m0_stability_suite.ps1 -UseEcho -QuickMode

# 并发压测
powershell -ExecutionPolicy Bypass -File scripts\test_echo_concurrency.ps1 -ConcurrentRuns 4 -TotalRuns 20

# SSE 重连测试
powershell -ExecutionPolicy Bypass -File scripts\test_sse_replay.ps1 -UseEcho
```

### 预期结果

- M0 套件：5 PASS, 0 FAIL, 2 SKIP
- 并发压测（4 并发）：16+ runs/sec，P95 < 250ms

---

## 契约与不变量

- 免鉴权端点: `/v1/health`, `/v1/version`, `/v1/pair`
- 其他端点必须鉴权 (401)
- 测试退出码: 0=PASS, 1=FAIL, 2=PRECONDITION_FAILURE
- Queue 并发: 默认 maxConcurrency=5
- Echo Provider: `preferred_provider: "echo"`, `preferred_model: "echo"`

---

## 回滚点

- 回滚到: 67505ac (M0 验证前)

---

## 交接备注

- **使用 echo provider 进行确定性测试**：`-UseEcho` 开关
- **并发配置**：系统默认 maxConcurrency=5，最佳区间 4-8
- PowerShell 转义问题：在 Bash 中调用 PowerShell 时 `$` 符号需要特殊处理
- API 返回 `run_id`（snake_case），不是 `runId`
- JSONL 事件：seq 从 1 开始，单调递增，终态事件是最后一条
