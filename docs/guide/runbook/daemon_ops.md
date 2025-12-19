# Daemon 运维手册

> MCPAgents Daemon 日常运维、部署和监控操作指南。

---

## 目录

1. [启动与停止](#1-启动与停止)
2. [健康检查](#2-健康检查)
3. [日志管理](#3-日志管理)
4. [配置管理](#4-配置管理)
5. [监控指标](#5-监控指标)
6. [备份与恢复](#6-备份与恢复)
7. [升级流程](#7-升级流程)
8. [文档站点](#8-文档站点)

---

## 1. 启动与停止

### 1.1 启动方式

#### 方式 A：托盘启动（推荐桌面用户）

```powershell
# 双击或命令行启动
D:\claude1\MCPAgents\apps\mcpagentsd\bin\mcpagentsd_tray.exe
```

特点：
- 最小化到系统托盘
- 开机自启动（可配置）
- 右键菜单管理

#### 方式 B：CLI 启动

```powershell
cd D:\claude1\MCPAgents\apps\mcpagents_cli
dart run bin/main.dart daemon start
```

#### 方式 C：直接启动

```powershell
# 前台运行
D:\claude1\MCPAgents\apps\mcpagentsd\bin\mcpagentsd.exe serve --port 8787

# 后台运行
Start-Process -FilePath "D:\claude1\MCPAgents\apps\mcpagentsd\bin\mcpagentsd.exe" `
  -ArgumentList "serve", "--port", "8787" -WindowStyle Hidden
```

### 1.2 停止方式

#### 优雅停止（推荐）

```powershell
# 通过 API 停止（需要 admin scope）
curl -X POST http://127.0.0.1:8787/admin/shutdown `
  -H "Authorization: Bearer YOUR_ADMIN_TOKEN"
```

#### 强制停止

```powershell
# 查找进程
Get-Process | Where-Object { $_.Name -like "*mcpagentsd*" }

# 终止进程
Stop-Process -Name "mcpagentsd" -Force
```

### 1.3 重启

```powershell
# 通过 CLI
dart run bin/main.dart daemon restart

# 手动
Stop-Process -Name "mcpagentsd" -Force
Start-Sleep -Seconds 2
Start-Process "D:\claude1\MCPAgents\apps\mcpagentsd\bin\mcpagentsd.exe" -ArgumentList "serve"
```

---

## 2. 健康检查

### 2.1 基础健康检查

```powershell
# 简单检查
curl http://127.0.0.1:8787/v1/health

# 预期响应
# {
#   "status": "healthy",
#   "version": "0.5.6",
#   "uptime_seconds": 3600,
#   "timestamp": "2025-12-19T10:00:00Z"
# }
```

### 2.2 详细状态

```powershell
curl http://127.0.0.1:8787/v1/health/detailed -H "Authorization: Bearer TOKEN"

# 响应包含：
# - 数据库连接状态
# - MCP Server 连接状态
# - Provider 可用性
# - 队列长度
# - 内存使用
```

### 2.3 自动化健康检查脚本

```powershell
# scripts/health_check.ps1
$uri = "http://127.0.0.1:8787/v1/health"
$timeout = 5

try {
    $response = Invoke-RestMethod -Uri $uri -TimeoutSec $timeout
    if ($response.status -eq "healthy") {
        Write-Host "✅ Daemon is healthy" -ForegroundColor Green
        exit 0
    } else {
        Write-Host "⚠️ Daemon is degraded: $($response.status)" -ForegroundColor Yellow
        exit 1
    }
} catch {
    Write-Host "❌ Daemon is unreachable" -ForegroundColor Red
    exit 2
}
```

---

## 3. 日志管理

### 3.1 日志位置

| 日志类型 | 路径 |
|----------|------|
| Daemon 日志 | `%APPDATA%\MCPAgents\logs\daemon.log` |
| 事件日志 | `%APPDATA%\MCPAgents\events\YYYY-MM-DD.jsonl` |
| MCP 日志 | `%APPDATA%\MCPAgents\logs\mcp\{server_name}.log` |
| 错误日志 | `%APPDATA%\MCPAgents\logs\error.log` |

### 3.2 日志级别

```json
// canonical_config.json
{
  "daemon": {
    "verbose": true,  // 开启详细日志
    "log_level": "debug"  // debug | info | warn | error
  }
}
```

### 3.3 查看实时日志

```powershell
# 实时跟踪
Get-Content "$env:APPDATA\MCPAgents\logs\daemon.log" -Wait -Tail 50

# 过滤错误
Get-Content "$env:APPDATA\MCPAgents\logs\daemon.log" | Select-String "ERROR|WARN"
```

### 3.4 日志轮转

日志自动按日期轮转，默认保留 7 天。

```json
// canonical_config.json
{
  "logging": {
    "max_days": 7,
    "max_size_mb": 100,
    "compress_old": true
  }
}
```

### 3.5 清理日志

```powershell
# 清理超过 7 天的日志
Get-ChildItem "$env:APPDATA\MCPAgents\logs\*.log" |
  Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-7) } |
  Remove-Item

# 清理事件日志
Get-ChildItem "$env:APPDATA\MCPAgents\events\*.jsonl" |
  Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-30) } |
  Remove-Item
```

---

## 4. 配置管理

### 4.1 配置文件位置

| 文件 | 用途 | 热重载 |
|------|------|--------|
| `canonical_config.json` | 主配置 | ❌ |
| `routing.json` | 路由策略 | ✅ |
| `models.json` | 模型能力 | ✅ |
| `daemon_tokens.json` | 鉴权 Token | ❌ |

### 4.2 修改配置

```powershell
# 编辑配置
notepad "$env:APPDATA\MCPAgents\canonical_config.json"

# 验证 JSON 格式
Get-Content "$env:APPDATA\MCPAgents\canonical_config.json" | ConvertFrom-Json
```

### 4.3 热重载配置

```powershell
# routing.json 和 models.json 支持热重载
# 修改后 1-2 秒自动生效，无需重启

# 验证配置已加载
curl http://127.0.0.1:8787/v1/config -H "Authorization: Bearer TOKEN"
```

### 4.4 配置备份

```powershell
# 备份所有配置
$backupDir = "$env:APPDATA\MCPAgents\backups\$(Get-Date -Format 'yyyy-MM-dd_HHmmss')"
New-Item -ItemType Directory -Path $backupDir -Force
Copy-Item "$env:APPDATA\MCPAgents\*.json" -Destination $backupDir
```

---

## 5. 监控指标

### 5.1 内置指标端点

```powershell
curl http://127.0.0.1:8787/metrics -H "Authorization: Bearer TOKEN"
```

返回 Prometheus 格式指标：

```
# HELP mcpagents_runs_total Total number of runs
# TYPE mcpagents_runs_total counter
mcpagents_runs_total{status="succeeded"} 1234
mcpagents_runs_total{status="failed"} 56

# HELP mcpagents_run_duration_seconds Run execution duration
# TYPE mcpagents_run_duration_seconds histogram
mcpagents_run_duration_seconds_bucket{le="1"} 100
mcpagents_run_duration_seconds_bucket{le="5"} 450
mcpagents_run_duration_seconds_bucket{le="30"} 890

# HELP mcpagents_queue_depth Current queue depth
# TYPE mcpagents_queue_depth gauge
mcpagents_queue_depth 5
```

### 5.2 关键指标

| 指标 | 告警阈值 | 说明 |
|------|----------|------|
| `mcpagents_queue_depth` | > 100 | 队列积压 |
| `mcpagents_run_duration_seconds_p99` | > 60s | 响应变慢 |
| `mcpagents_error_rate` | > 5% | 错误率高 |
| `mcpagents_memory_bytes` | > 2GB | 内存泄漏 |

### 5.3 Grafana Dashboard

导入预置 Dashboard：`docs/guide/runbook/grafana_dashboard.json`

---

## 6. 备份与恢复

### 6.1 需要备份的数据

| 数据 | 路径 | 重要性 |
|------|------|--------|
| 数据库 | `%APPDATA%\MCPAgents\chatmcp.db` | 🔴 关键 |
| 配置 | `%APPDATA%\MCPAgents\*.json` | 🔴 关键 |
| Token | `%APPDATA%\MCPAgents\daemon_tokens.json` | 🔴 关键 |
| 日志 | `%APPDATA%\MCPAgents\logs\` | 🟡 重要 |
| 事件 | `%APPDATA%\MCPAgents\events\` | 🟢 可选 |

### 6.2 备份脚本

```powershell
# scripts/backup.ps1
param(
    [string]$BackupPath = "$env:USERPROFILE\MCPAgents_Backups"
)

$timestamp = Get-Date -Format "yyyy-MM-dd_HHmmss"
$backupDir = Join-Path $BackupPath $timestamp

# 停止 Daemon
Stop-Process -Name "mcpagentsd" -Force -ErrorAction SilentlyContinue
Start-Sleep -Seconds 2

# 创建备份目录
New-Item -ItemType Directory -Path $backupDir -Force

# 复制数据
$source = "$env:APPDATA\MCPAgents"
Copy-Item "$source\chatmcp.db" -Destination $backupDir
Copy-Item "$source\*.json" -Destination $backupDir
Copy-Item "$source\logs" -Destination $backupDir -Recurse

# 压缩
Compress-Archive -Path $backupDir -DestinationPath "$backupDir.zip"
Remove-Item $backupDir -Recurse -Force

Write-Host "✅ Backup created: $backupDir.zip"
```

### 6.3 恢复流程

```powershell
# 1. 停止 Daemon
Stop-Process -Name "mcpagentsd" -Force

# 2. 解压备份
Expand-Archive -Path "backup.zip" -DestinationPath "$env:TEMP\restore"

# 3. 恢复文件
Copy-Item "$env:TEMP\restore\*" -Destination "$env:APPDATA\MCPAgents" -Recurse -Force

# 4. 启动 Daemon
Start-Process "D:\claude1\MCPAgents\apps\mcpagentsd\bin\mcpagentsd.exe" -ArgumentList "serve"
```

---

## 7. 升级流程

### 7.1 升级前检查

```powershell
# 1. 检查当前版本
curl http://127.0.0.1:8787/v1/health | ConvertFrom-Json | Select-Object version

# 2. 备份数据
.\scripts\backup.ps1

# 3. 检查队列是否为空
curl http://127.0.0.1:8787/v1/runs?status=queued,running
```

### 7.2 升级步骤

```powershell
# 1. 优雅停止
curl -X POST http://127.0.0.1:8787/admin/shutdown -H "Authorization: Bearer ADMIN_TOKEN"

# 2. 等待进程退出
while (Get-Process -Name "mcpagentsd" -ErrorAction SilentlyContinue) {
    Start-Sleep -Seconds 1
}

# 3. 替换二进制
Copy-Item "new_mcpagentsd.exe" -Destination "D:\claude1\MCPAgents\apps\mcpagentsd\bin\mcpagentsd.exe" -Force

# 4. 启动新版本
Start-Process "D:\claude1\MCPAgents\apps\mcpagentsd\bin\mcpagentsd.exe" -ArgumentList "serve"

# 5. 验证
Start-Sleep -Seconds 5
curl http://127.0.0.1:8787/v1/health
```

### 7.3 回滚

```powershell
# 使用备份恢复
.\scripts\restore.ps1 -BackupFile "backup_2025-12-19.zip"
```

---

## 快速参考

### 常用命令

```powershell
# 启动
mcpagentsd.exe serve --port 8787

# 健康检查
curl http://127.0.0.1:8787/v1/health

# 查看日志
Get-Content "$env:APPDATA\MCPAgents\logs\daemon.log" -Wait

# 停止
curl -X POST http://127.0.0.1:8787/admin/shutdown -H "Authorization: Bearer TOKEN"
```

### 端口占用检查

```powershell
netstat -ano | findstr ":8787"
Get-Process -Id (Get-NetTCPConnection -LocalPort 8787).OwningProcess
```

### 进程锁清理

```powershell
# 如果 Daemon 异常退出，可能残留锁文件
Remove-Item "$env:APPDATA\MCPAgents\daemon.lock" -Force
```

---

## 8. 文档站点

### 8.1 本地预览

```powershell
# 安装依赖
pip install -r docs/guide/requirements-docs.txt

# 启动预览服务
cd D:\claude1\MCPAgents
mkdocs serve

# 访问 http://127.0.0.1:8000
```

### 8.2 构建静态站点

```powershell
cd D:\claude1\MCPAgents
mkdocs build

# 输出目录：site/
```

### 8.3 部署到 GitHub Pages

```powershell
cd D:\claude1\MCPAgents
mkdocs gh-deploy
```

---

## 相关文档

- [故障排查](troubleshooting.md)
- [配置参考](../reference/config_reference.md)
- [架构决策](../adr/0001-single-writer-daemon.md)
