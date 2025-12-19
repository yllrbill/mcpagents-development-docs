# 故障排查指南

> MCPAgents 常见问题诊断和解决方案。

---

## 目录

1. [Daemon 启动问题](#1-daemon-启动问题)
2. [连接问题](#2-连接问题)
3. [LLM 调用问题](#3-llm-调用问题)
4. [MCP 工具问题](#4-mcp-工具问题)
5. [性能问题](#5-性能问题)
6. [数据问题](#6-数据问题)
7. [诊断工具](#7-诊断工具)

---

## 1. Daemon 启动问题

### 1.1 端口被占用

**症状**：
```
Error: Port 8787 is already in use
```

**诊断**：
```powershell
# 检查端口占用
netstat -ano | findstr ":8787"
Get-Process -Id (Get-NetTCPConnection -LocalPort 8787).OwningProcess
```

**解决方案**：
```powershell
# 方案 A：终止占用进程
Stop-Process -Id <PID> -Force

# 方案 B：使用其他端口
mcpagentsd.exe serve --port 8788

# 方案 C：修改配置
# 编辑 %APPDATA%\MCPAgents\canonical_config.json
# "daemon": { "port": 8788 }
```

### 1.2 另一个 Daemon 实例正在运行

**症状**：
```
Error: Another daemon instance is running (lock file exists)
```

**诊断**：
```powershell
# 检查锁文件
Test-Path "$env:APPDATA\MCPAgents\daemon.lock"

# 检查进程
Get-Process | Where-Object { $_.Name -like "*mcpagentsd*" }
```

**解决方案**：
```powershell
# 方案 A：停止现有实例
Stop-Process -Name "mcpagentsd" -Force

# 方案 B：清理残留锁（仅当进程确实不存在时）
Remove-Item "$env:APPDATA\MCPAgents\daemon.lock" -Force
```

### 1.3 配置文件损坏

**症状**：
```
Error: Failed to parse canonical_config.json
```

**诊断**：
```powershell
# 验证 JSON 格式
Get-Content "$env:APPDATA\MCPAgents\canonical_config.json" | ConvertFrom-Json
```

**解决方案**：
```powershell
# 方案 A：修复 JSON 语法
# 使用在线 JSON 验证器检查

# 方案 B：恢复默认配置
Copy-Item "D:\claude1\MCPAgents\assets\default_config.json" `
  -Destination "$env:APPDATA\MCPAgents\canonical_config.json"
```

### 1.4 数据库打开失败

**症状**：
```
Error: Failed to open database: chatmcp.db
```

**诊断**：
```powershell
# 检查文件权限
Get-Acl "$env:APPDATA\MCPAgents\chatmcp.db"

# 检查文件是否被锁定
try {
    [System.IO.File]::Open("$env:APPDATA\MCPAgents\chatmcp.db", 'Open', 'ReadWrite', 'None').Close()
    Write-Host "File is not locked"
} catch {
    Write-Host "File is locked"
}
```

**解决方案**：
```powershell
# 方案 A：等待其他进程释放
Start-Sleep -Seconds 5

# 方案 B：修复数据库
sqlite3 "$env:APPDATA\MCPAgents\chatmcp.db" "PRAGMA integrity_check;"

# 方案 C：从备份恢复
Copy-Item "backup\chatmcp.db" -Destination "$env:APPDATA\MCPAgents\chatmcp.db"
```

---

## 2. 连接问题

### 2.1 GUI 无法连接 Daemon

**症状**：
- GUI 显示"离线模式"
- 无法发送消息

**诊断**：
```powershell
# 1. 检查 Daemon 是否运行
curl http://127.0.0.1:8787/v1/health

# 2. 检查防火墙
Get-NetFirewallRule | Where-Object { $_.DisplayName -like "*mcpagents*" }

# 3. 检查 Token
Get-Content "$env:APPDATA\MCPAgents\daemon_tokens.json" | ConvertFrom-Json
```

**解决方案**：
```powershell
# 方案 A：启动 Daemon
Start-Process "D:\claude1\MCPAgents\apps\mcpagentsd\bin\mcpagentsd.exe" -ArgumentList "serve"

# 方案 B：添加防火墙规则
New-NetFirewallRule -DisplayName "MCPAgents Daemon" `
  -Direction Inbound -LocalPort 8787 -Protocol TCP -Action Allow

# 方案 C：重新生成 Token
# 通过 CLI 或 GUI 设置页面重新获取 Token
```

### 2.2 SSE 连接频繁断开

**症状**：
- 实时消息断断续续
- 日志显示频繁重连

**诊断**：
```powershell
# 检查网络稳定性
ping 127.0.0.1 -t

# 检查 Daemon 内存
Get-Process mcpagentsd | Select-Object WorkingSet64
```

**解决方案**：
```powershell
# 方案 A：增加超时时间
# 修改 routing.json
# "throughput_settings": { "timeout_seconds": 120 }

# 方案 B：禁用代理
# 确保 127.0.0.1 不走代理
```

### 2.3 远程连接失败（Termux/其他设备）

**症状**：
```
Error: Connection refused to 192.168.x.x:8787
```

**诊断**：
```powershell
# 检查绑定地址
Get-Content "$env:APPDATA\MCPAgents\canonical_config.json" | ConvertFrom-Json | Select-Object -ExpandProperty daemon

# 检查防火墙
Test-NetConnection -ComputerName 192.168.x.x -Port 8787
```

**解决方案**：
```powershell
# 1. 修改绑定地址
# canonical_config.json: "bind": "0.0.0.0"

# 2. 添加防火墙规则
New-NetFirewallRule -DisplayName "MCPAgents Remote" `
  -Direction Inbound -LocalPort 8787 -Protocol TCP -Action Allow -Profile Any

# 3. 重启 Daemon
```

---

## 3. LLM 调用问题

### 3.1 API Key 无效

**症状**：
```json
{"error": "Invalid API key", "provider": "openai"}
```

**诊断**：
```powershell
# 检查配置
Get-Content "$env:APPDATA\MCPAgents\canonical_config.json" |
  ConvertFrom-Json |
  Select-Object -ExpandProperty providers

# 测试 API Key
curl https://api.openai.com/v1/models -H "Authorization: Bearer sk-xxx"
```

**解决方案**：
```powershell
# 方案 A：更新配置中的 API Key
# 编辑 canonical_config.json

# 方案 B：使用 api_key_path 指向文件
# "api_key_path": "D:\\secrets\\openai_key.txt"

# 方案 C：禁用该 Provider，使用其他
# "enabled": false
```

### 3.2 模型不可用

**症状**：
```json
{"error": "Model not found", "model": "gpt-5"}
```

**诊断**：
```powershell
# 查看可用模型
curl http://127.0.0.1:8787/v1/models -H "Authorization: Bearer TOKEN"

# 检查 models.json
Get-Content "$env:APPDATA\MCPAgents\models.json" | ConvertFrom-Json
```

**解决方案**：
- 使用存在的模型
- 更新 models.json 添加新模型定义
- 检查 Provider 是否支持该模型

### 3.3 所有 Provider 都失败

**症状**：
```
Error: All providers exhausted, run failed
```

**诊断**：
```powershell
# 查看路由决策
curl "http://127.0.0.1:8787/v1/runs/{id}/events" -H "Authorization: Bearer TOKEN"

# 检查每个 Provider 状态
curl http://127.0.0.1:8787/v1/providers -H "Authorization: Bearer TOKEN"
```

**解决方案**：
- 检查网络连接
- 验证所有 API Key
- 确保至少一个 Provider 启用且可用
- 检查是否被限流

### 3.4 响应超时

**症状**：
```
Error: Request timeout after 60 seconds
```

**诊断**：
```powershell
# 测试 Provider 延迟
Measure-Command { curl https://api.openai.com/v1/models -H "Authorization: Bearer sk-xxx" }
```

**解决方案**：
```json
// routing.json
{
  "throughput_settings": {
    "timeout_seconds": 120,
    "fast_failover": true
  }
}
```

---

## 4. MCP 工具问题

### 4.1 MCP Server 启动失败

**症状**：
```
Error: Failed to start MCP server 'filesystem'
```

**诊断**：
```powershell
# 检查 MCP 配置
Get-Content "$env:APPDATA\MCPAgents\assets\mcp_server.json" | ConvertFrom-Json

# 手动启动测试
uvx mcp-server-filesystem --allowed-directories "D:\docs"
```

**解决方案**：
- 检查命令路径是否正确
- 安装缺失的依赖（uvx, npx 等）
- 检查环境变量

### 4.2 工具调用被拒绝

**症状**：
```
Error: Tool 'shell.exec' is blocked by security policy
```

**诊断**：
```powershell
# 检查安全策略
Get-Content "$env:APPDATA\MCPAgents\canonical_config.json" |
  ConvertFrom-Json |
  Select-Object -ExpandProperty mcp |
  Select-Object -ExpandProperty security
```

**解决方案**：
```json
// canonical_config.json
{
  "mcp": {
    "security": {
      "allowed_tools": ["filesystem.*", "shell.exec"],
      "blocked_tools": []
    }
  }
}
```

### 4.3 工具参数过大

**症状**：
```
Error: Tool argument exceeds maximum size (1048576 bytes)
```

**解决方案**：
```json
// canonical_config.json
{
  "mcp": {
    "security": {
      "max_argument_size_bytes": 5242880
    }
  }
}
```

---

## 5. 性能问题

### 5.1 响应变慢

**症状**：
- 首次响应延迟高
- 队列积压

**诊断**：
```powershell
# 检查队列深度
curl http://127.0.0.1:8787/v1/runs?status=queued | ConvertFrom-Json | Measure-Object

# 检查内存使用
Get-Process mcpagentsd | Select-Object WorkingSet64, CPU

# 检查数据库大小
(Get-Item "$env:APPDATA\MCPAgents\chatmcp.db").Length / 1MB
```

**解决方案**：
```powershell
# 方案 A：增加并发
# routing.json: "max_concurrency": 50

# 方案 B：清理历史数据
sqlite3 "$env:APPDATA\MCPAgents\chatmcp.db" "DELETE FROM runs WHERE completed_at < datetime('now', '-30 days');"
sqlite3 "$env:APPDATA\MCPAgents\chatmcp.db" "VACUUM;"

# 方案 C：重启 Daemon
```

### 5.2 内存持续增长

**症状**：
- Daemon 内存使用持续上升
- 最终 OOM 崩溃

**诊断**：
```powershell
# 监控内存
while ($true) {
    Get-Process mcpagentsd | Select-Object WorkingSet64
    Start-Sleep -Seconds 60
}
```

**解决方案**：
```powershell
# 方案 A：定期重启（临时）
# 设置计划任务每 24 小时重启

# 方案 B：报告问题
# 收集日志和内存 dump 提交 issue
```

---

## 6. 数据问题

### 6.1 对话历史丢失

**诊断**：
```powershell
# 检查数据库
sqlite3 "$env:APPDATA\MCPAgents\chatmcp.db" "SELECT COUNT(*) FROM chats;"

# 检查完整性
sqlite3 "$env:APPDATA\MCPAgents\chatmcp.db" "PRAGMA integrity_check;"
```

**解决方案**：
```powershell
# 从备份恢复
Copy-Item "backup\chatmcp.db" -Destination "$env:APPDATA\MCPAgents\chatmcp.db"
```

### 6.2 Run 卡在 running 状态

**症状**：
- Run 长时间处于 running 状态
- 没有后续事件

**诊断**：
```powershell
# 查看卡住的 Run
sqlite3 "$env:APPDATA\MCPAgents\chatmcp.db" `
  "SELECT id, status, started_at FROM runs WHERE status = 'running' AND started_at < datetime('now', '-10 minutes');"
```

**解决方案**：
```powershell
# 手动取消
curl -X POST "http://127.0.0.1:8787/v1/runs/{id}/cancel" -H "Authorization: Bearer TOKEN"

# 或直接更新数据库（谨慎）
sqlite3 "$env:APPDATA\MCPAgents\chatmcp.db" `
  "UPDATE runs SET status = 'failed', completed_at = datetime('now') WHERE id = 'xxx';"
```

---

## 7. 诊断工具

### 7.1 内置诊断命令

```powershell
# 完整系统诊断
mcpagentsd.exe diagnose

# 输出：
# - 配置验证
# - 数据库检查
# - Provider 连通性
# - MCP Server 状态
```

### 7.2 诊断脚本

```powershell
# scripts/diagnose.ps1

Write-Host "=== MCPAgents Diagnostics ===" -ForegroundColor Cyan

# 1. 检查 Daemon
Write-Host "`n[1] Daemon Status"
try {
    $health = Invoke-RestMethod http://127.0.0.1:8787/v1/health
    Write-Host "  ✅ Daemon running: v$($health.version)" -ForegroundColor Green
} catch {
    Write-Host "  ❌ Daemon not reachable" -ForegroundColor Red
}

# 2. 检查配置
Write-Host "`n[2] Configuration"
$configPath = "$env:APPDATA\MCPAgents\canonical_config.json"
if (Test-Path $configPath) {
    try {
        $config = Get-Content $configPath | ConvertFrom-Json
        Write-Host "  ✅ Config valid" -ForegroundColor Green
        $providers = ($config.providers.PSObject.Properties | Where-Object { $_.Value.enabled }).Name
        Write-Host "  Enabled providers: $($providers -join ', ')"
    } catch {
        Write-Host "  ❌ Config invalid: $_" -ForegroundColor Red
    }
} else {
    Write-Host "  ❌ Config not found" -ForegroundColor Red
}

# 3. 检查数据库
Write-Host "`n[3] Database"
$dbPath = "$env:APPDATA\MCPAgents\chatmcp.db"
if (Test-Path $dbPath) {
    $size = (Get-Item $dbPath).Length / 1MB
    Write-Host "  ✅ Database exists (${size:N2} MB)" -ForegroundColor Green
} else {
    Write-Host "  ❌ Database not found" -ForegroundColor Red
}

# 4. 检查端口
Write-Host "`n[4] Port 8787"
$conn = Get-NetTCPConnection -LocalPort 8787 -ErrorAction SilentlyContinue
if ($conn) {
    $proc = Get-Process -Id $conn.OwningProcess
    Write-Host "  ✅ Port in use by: $($proc.Name)" -ForegroundColor Green
} else {
    Write-Host "  ⚠️ Port not in use" -ForegroundColor Yellow
}

Write-Host "`n=== Diagnostics Complete ===" -ForegroundColor Cyan
```

### 7.3 收集诊断信息

```powershell
# 收集完整诊断包
$diagDir = "$env:TEMP\mcpagents_diag_$(Get-Date -Format 'yyyyMMdd_HHmmss')"
New-Item -ItemType Directory -Path $diagDir -Force

# 配置
Copy-Item "$env:APPDATA\MCPAgents\*.json" -Destination $diagDir

# 日志（最近 1000 行）
Get-Content "$env:APPDATA\MCPAgents\logs\daemon.log" -Tail 1000 |
  Out-File "$diagDir\daemon_log_tail.txt"

# 系统信息
Get-ComputerInfo | Out-File "$diagDir\system_info.txt"

# 压缩
Compress-Archive -Path $diagDir -DestinationPath "$diagDir.zip"

Write-Host "Diagnostic package: $diagDir.zip"
```

---

## 快速参考卡

| 问题 | 快速检查 | 快速修复 |
|------|----------|----------|
| Daemon 不响应 | `curl http://127.0.0.1:8787/v1/health` | 重启 Daemon |
| 端口占用 | `netstat -ano \| findstr :8787` | 终止占用进程或换端口 |
| API Key 无效 | 检查 `canonical_config.json` | 更新 API Key |
| MCP 工具失败 | 检查 `mcp_server.json` | 修复命令路径 |
| 响应慢 | 检查队列深度 | 增加并发/清理历史 |

---

## 相关文档

- [运维手册](daemon_ops.md)
- [配置参考](../reference/config_reference.md)
- [CAS 状态机](../adr/0003-cas-state-machine.md)
