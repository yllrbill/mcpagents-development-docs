# MCPAgents Session Status (Public)

> 脱敏版会话状态，用于跨对话恢复进展。
> 完整内部版本见私有仓 `docs/SESSION_STATE.md`。

---

## Metadata

| 项目 | 值 |
|------|-----|
| 当前里程碑 | M2 — Fault Injection Tests |
| 会话编号 | 2025-12-20_m2_fault_injection |
| 更新时间 | 2025-12-20 17:50 (UTC+8) |
| 当前分支 | main |
| 最新 commit | f90055b |

---

## TL;DR (30 秒恢复区)

### 已完成

- M1: Golden path smoke tests (`scripts/smoke_e2e.ps1/.sh`)
- M2: Fault injection tests - 3 个场景
  - `test_port_conflict`: 端口冲突快速失败
  - `test_appdata_unavailable`: 数据目录不可用
  - `test_crash_recovery`: 崩溃后重启恢复
- CI 集成: nightly (~04:00 UTC) + workflow_dispatch
- 文档更新: `SOURCE_REVISION.txt`, `daemon_ops.md` §9

### 进行中

- 验证公共仓同步结果

### 阻塞

- 无

### 下一步 (唯一最高优先级)

验证 `daemon_ops.md` §9 故障注入测试章节已同步到公共仓

---

## 关键变更

| Commit | 说明 |
|--------|------|
| 5493c26 | feat(m2): add fault injection tests and nightly CI |
| f90055b | docs(session): update SESSION_STATE.md for M2 checkpoint |

---

## 验证命令

```powershell
# 运行故障注入测试 (需要先停止 daemon)
powershell -ExecutionPolicy Bypass -File scripts\fault_injection\test_port_conflict.ps1
powershell -ExecutionPolicy Bypass -File scripts\fault_injection\test_appdata_unavailable.ps1 -Port 8789
powershell -ExecutionPolicy Bypass -File scripts\fault_injection\test_crash_recovery.ps1 -Port 8790
```

### 预期结果

- 所有测试退出码为 0 (PASS)
- 退出码 2 = 前置条件不满足 (附可操作建议)

---

## 契约与不变量

- 免鉴权端点: `/v1/health`, `/v1/version`, `/v1/pair`
- 其他端点必须鉴权 (401)
- 测试退出码: 0=PASS, 1=FAIL, 2=PRECONDITION_FAILURE

---

## 回滚点

- 回滚到: f8dc94a (M2 之前)

---

## 交接备注

- PowerShell 5.1 不支持 `??` 运算符，需用 if/elseif
- Git Bash 下 `taskkill /PID` 会被转换为路径，用 PowerShell `Stop-Process` 替代
- 故障注入测试使用不同端口 (8787/8789/8790) 避免冲突
