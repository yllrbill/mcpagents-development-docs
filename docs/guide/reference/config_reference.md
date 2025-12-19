# 配置参考手册

> MCPAgents 所有配置文件的字段级说明、默认值和使用示例。

---

## 配置文件索引

| 文件 | 位置 | 热重载 | 用途 |
|------|------|--------|------|
| `canonical_config.json` | `%APPDATA%\MCPAgents\` | ❌ | 主配置（Provider、MCP 等）|
| `routing.json` | `%APPDATA%\MCPAgents\` | ✅ | 路由策略 |
| `models.json` | `%APPDATA%\MCPAgents\` | ✅ | 模型能力定义 |
| `daemon_tokens.json` | `%APPDATA%\MCPAgents\` | ❌ | 鉴权 Token |
| `mcp_server.json` | `assets/` | ❌ | MCP Server 定义 |

---

## 1. canonical_config.json

主配置文件，Daemon 启动时加载。

### 完整结构

```json
{
  "version": "1.0",
  "daemon": {
    "bind": "127.0.0.1",
    "port": 8787,
    "auto_start": true,
    "verbose": false
  },
  "providers": {
    "openai": {
      "enabled": true,
      "api_key": "sk-xxx",
      "base_url": "https://api.openai.com/v1",
      "default_model": "gpt-4o"
    },
    "claude": {
      "enabled": true,
      "api_key": "sk-ant-xxx",
      "base_url": "https://api.anthropic.com",
      "default_model": "claude-sonnet-4-20250514"
    },
    "ollama": {
      "enabled": true,
      "base_url": "http://localhost:11434",
      "default_model": "qwen3:8b"
    },
    "poe": {
      "enabled": true,
      "api_key_path": "D:\\tools\\AI\\poe\\poeapikey.txt"
    }
  },
  "security": {
    "require_auth": true,
    "default_scopes": ["read", "run"],
    "token_expiry_days": 30
  },
  "mcp": {
    "enabled": true,
    "config_path": "assets/mcp_server.json",
    "security": {
      "whitelist_mode": true,
      "allowed_tools": ["filesystem.*", "git.*"],
      "blocked_tools": ["shell.*"],
      "max_argument_size_bytes": 1048576
    }
  },
  "tunnel": {
    "enabled": false,
    "hostname": "your-tunnel.example.com",
    "token": null
  }
}
```

### 字段说明

#### daemon

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `bind` | string | `"127.0.0.1"` | 监听地址 |
| `port` | int | `8787` | 监听端口 |
| `auto_start` | bool | `true` | GUI 启动时自动启动 Daemon |
| `verbose` | bool | `false` | 详细日志 |

#### providers.{name}

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `enabled` | bool | `false` | 是否启用 |
| `api_key` | string | - | API 密钥 |
| `api_key_path` | string | - | API 密钥文件路径（优先） |
| `base_url` | string | Provider 默认 | API 端点 |
| `default_model` | string | Provider 默认 | 默认模型 |

#### security

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `require_auth` | bool | `true` | 是否需要 Token 鉴权 |
| `default_scopes` | array | `["read", "run"]` | 新设备默认权限 |
| `token_expiry_days` | int | `30` | Token 过期天数 |

#### mcp.security

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `whitelist_mode` | bool | `true` | 白名单模式 |
| `allowed_tools` | array | `[]` | 允许的工具（支持通配符）|
| `blocked_tools` | array | `[]` | 禁止的工具 |
| `max_argument_size_bytes` | int | `1048576` | 参数最大字节数 |

---

## 2. routing.json

路由策略配置，支持热重载。

### 完整结构

```json
{
  "default_mode": "stable_cost",
  "task_mode_overrides": {
    "code": "quality_max",
    "mcp": "quality_max",
    "mcpParse": "quality_max"
  },
  "privacy_force_local": false,
  "privacy_exclude_providers": [
    "dashscope",
    "deepseek",
    "baidu",
    "moonshot",
    "zhipu"
  ],
  "provider_priorities": {},
  "throughput_settings": {
    "max_concurrency": 20,
    "timeout_seconds": 60,
    "max_retries_per_model": 1,
    "enable_cache": true,
    "cache_ttl_minutes": 5,
    "fast_failover": true
  }
}
```

### 字段说明

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `default_mode` | enum | `"stable_cost"` | 默认路由策略 |
| `task_mode_overrides` | object | `{}` | 任务类型 → 策略覆盖 |
| `privacy_force_local` | bool | `false` | 隐私任务强制本地 |
| `privacy_exclude_providers` | array | `[...]` | 隐私任务排除的 Provider |

### 路由策略枚举

| 模式 | 说明 | 优先级 |
|------|------|--------|
| `stable_cost` | 成本优先 | ollama → poe → direct |
| `quality_max` | 质量优先 | direct → poe → ollama |
| `throughput_max` | 吞吐优先 | direct (跳过 Poe) + 缓存 |

### throughput_settings

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `max_concurrency` | int | `20` | 最大并发数 |
| `timeout_seconds` | int | `60` | 超时秒数 |
| `max_retries_per_model` | int | `1` | 每模型最大重试 |
| `enable_cache` | bool | `true` | 启用请求缓存 |
| `cache_ttl_minutes` | int | `5` | 缓存 TTL |
| `fast_failover` | bool | `true` | 快速失败切换 |

### 热重载说明

修改 `routing.json` 后，**无需重启 Daemon**，下次请求自动加载新配置。

---

## 3. models.json

模型能力定义，支持热重载。

### 完整结构

```json
{
  "models": {
    "openai:gpt-4o": {
      "provider": "openai",
      "model_id": "gpt-4o",
      "supports_vision": true,
      "supports_audio": false,
      "supports_tools": true,
      "supports_strict_tool_schema": true,
      "supports_structured_outputs": true,
      "context_window": 128000,
      "cost_weight": 0.8
    },
    "openai:gpt-4o-audio-preview": {
      "provider": "openai",
      "model_id": "gpt-4o-audio-preview",
      "supports_vision": true,
      "supports_audio": true,
      "supports_tools": true,
      "context_window": 128000,
      "cost_weight": 0.9
    },
    "ollama:qwen3:8b": {
      "provider": "ollama",
      "model_id": "qwen3:8b",
      "supports_vision": false,
      "supports_audio": false,
      "supports_tools": false,
      "context_window": 32768,
      "cost_weight": 0.0
    },
    "ollama:llava:latest": {
      "provider": "ollama",
      "model_id": "llava:latest",
      "supports_vision": true,
      "supports_audio": false,
      "supports_tools": false,
      "context_window": 4096,
      "cost_weight": 0.0
    }
  },
  "provider_defaults": {
    "openai": {
      "supports_vision": true,
      "supports_audio": false,
      "supports_tools": true
    },
    "ollama": {
      "supports_vision": false,
      "supports_audio": false,
      "supports_tools": false
    }
  },
  "allow_unsafe_capability_overrides": false
}
```

### 字段说明

#### models.{key}

键格式：`{provider}:{model_id}`

| 字段 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `provider` | string | - | Provider 名称 |
| `model_id` | string | - | 模型 ID |
| `supports_vision` | bool | Provider 默认 | 支持图像输入 |
| `supports_audio` | bool | Provider 默认 | 支持音频输入 |
| `supports_tools` | bool | Provider 默认 | 支持工具调用 |
| `supports_strict_tool_schema` | bool | `false` | 支持 strict schema |
| `supports_structured_outputs` | bool | `false` | 支持 structured outputs |
| `context_window` | int | `4096` | 上下文窗口 |
| `cost_weight` | float | `0.5` | 成本权重 (0.0-1.0) |

#### 安全策略

| 字段 | 说明 |
|------|------|
| `allow_unsafe_capability_overrides` | 设为 `true` 允许扩展能力（如将 `supports_audio: false → true`）|

默认只允许"收窄"能力，避免虚假能力承诺导致路由失败。

### 热重载说明

修改 `models.json` 后，**无需重启 Daemon**，约 1-2 秒后自动加载。

---

## 4. daemon_tokens.json

鉴权 Token 存储。

### 结构

```json
{
  "primary_token": "xxx",
  "tokens": [
    {
      "token": "xxx",
      "created_at": "2025-12-19T00:00:00Z",
      "scopes": ["read", "run", "config", "admin"],
      "device_id": null,
      "device_name": "primary"
    },
    {
      "token": "yyy",
      "created_at": "2025-12-19T10:00:00Z",
      "scopes": ["read", "run"],
      "device_id": "device_123",
      "device_name": "Termux CLI",
      "last_seen_at": "2025-12-19T14:30:00Z"
    }
  ]
}
```

### Scope 权限

| Scope | 允许的操作 |
|-------|-----------|
| `read` | GET /v1/chats, /v1/runs, /v1/config |
| `config` | PUT /v1/config, /v1/config/providers |
| `run` | POST /v1/runs, /v1/runs/:id/cancel |
| `admin` | 所有操作，包括 shutdown |

---

## 5. mcp_server.json

MCP Server 定义。

### 结构

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "uvx",
      "args": ["mcp-server-filesystem", "--allowed-directories", "/home/user/docs"],
      "env": {},
      "transport": "stdio"
    },
    "git": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-server-git"],
      "transport": "stdio"
    },
    "remote-server": {
      "url": "http://remote-host:8080/sse",
      "transport": "sse"
    }
  }
}
```

### 字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `command` | string | 启动命令 (stdio) |
| `args` | array | 命令参数 |
| `env` | object | 环境变量 |
| `url` | string | SSE 端点 (sse) |
| `transport` | enum | `stdio` 或 `sse` |

---

## 常见配置组合

### 最小配置（仅 Ollama）

```json
// canonical_config.json
{
  "providers": {
    "ollama": {
      "enabled": true,
      "base_url": "http://localhost:11434"
    }
  }
}
```

### 隐私优先配置

```json
// routing.json
{
  "default_mode": "stable_cost",
  "privacy_force_local": true,
  "privacy_exclude_providers": ["dashscope", "deepseek", "baidu", "moonshot", "zhipu"]
}
```

### 高质量代码配置

```json
// routing.json
{
  "default_mode": "stable_cost",
  "task_mode_overrides": {
    "code": "quality_max",
    "mcp": "quality_max"
  }
}
```

### 企业批量配置

```json
// routing.json
{
  "default_mode": "throughput_max",
  "throughput_settings": {
    "max_concurrency": 50,
    "timeout_seconds": 30,
    "enable_cache": true,
    "cache_ttl_minutes": 10
  }
}
```

---

## 配置优先级

1. **命令行参数** > **环境变量** > **配置文件** > **默认值**

```bash
# 命令行覆盖配置文件
mcpagentsd.exe --port 9000 --bind 0.0.0.0
```

---

## 向后兼容策略

- 新增字段使用默认值，不破坏旧配置
- 废弃字段保留解析但忽略，打印警告
- 重大变更使用 `version` 字段区分
