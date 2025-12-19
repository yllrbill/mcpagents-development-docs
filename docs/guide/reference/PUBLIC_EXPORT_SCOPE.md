# 公开文档导出范围

> 定义哪些文件/目录允许同步到公开仓库 `yllrbill/mcpagents-development-docs`

---

## 同步目标

- **公开仓库**: https://github.com/yllrbill/mcpagents-development-docs
- **用途**: 供建议型 LLM（如 Claude、ChatGPT）阅读项目文档
- **同步方式**: GitHub Actions 自动同步 / 手动脚本

---

## 允许公开的目录/文件（白名单）

### 文档目录

| 源路径 | 说明 | 公开 |
|--------|------|------|
| `docs/guide/` | 完整文档站点 | ✅ |
| `mkdocs.yml` | MkDocs 配置 | ✅ |
| `CLAUDE.md` | 项目上下文说明 | ✅ |

### Claude Code 配置

| 源路径 | 说明 | 公开 |
|--------|------|------|
| `.claude/commands/*.md` | 工作流命令 | ✅ |
| `.claude/settings.json` | 权限白名单（脱敏） | ✅ |
| `.claude/settings.local.json` | 本地配置 | ❌ 禁止 |

### 代码文件（按需添加）

| 源路径 | 说明 | 公开 |
|--------|------|------|
| `packages/mcpagents_core/lib/src/models/` | 核心数据模型 | 🔄 待定 |
| `packages/mcpagents_core/lib/src/services/` | 核心服务接口 | 🔄 待定 |
| `apps/mcpagentsd/lib/src/router.dart` | HTTP 路由定义 | 🔄 待定 |

---

## 禁止公开的内容（黑名单）

### 绝对禁止

- [ ] 任何包含 `token`、`apikey`、`secret`、`password` 的文件
- [ ] `.env` / `.env.*` 环境变量文件
- [ ] `*_tokens.json` / `credentials.json` 等凭证文件
- [ ] `.claude/settings.local.json`（可能含本机路径）
- [ ] 包含绝对路径（`D:\`、`C:\Users\`）的配置文件

### 需要脱敏后才能公开

- [ ] 配置示例文件（移除真实 API Key）
- [ ] 日志文件（移除用户数据）

---

## 敏感信息扫描规则

同步前必须扫描以下关键词：

```
# API Keys / Tokens
token
apikey
api_key
secret
password
cookie
bearer

# 云服务商
GOOGLE_CLOUD_
OPENAI_
ANTHROPIC_
GROK_
POE_
DEEPSEEK_
DASHSCOPE_

# 本机路径
D:\
C:\Users\
/home/
/Users/

# 其他敏感
private_key
ssh_key
credentials
```

---

## 同步流程

### 自动同步（GitHub Actions）

触发条件：
- `docs/guide/**` 变更
- `.claude/**` 变更
- `mkdocs.yml` 变更
- `CLAUDE.md` 变更

### 手动同步

```powershell
# 运行同步脚本
.\scripts\sync_to_public.ps1
```

---

## 添加新的公开目录

1. 编辑本文件，在"允许公开的目录/文件"表格中添加新条目
2. 确认不包含敏感信息
3. 更新同步脚本/GitHub Actions 配置
4. 运行一次同步并验证

---

## 版本历史

| 日期 | 变更 |
|------|------|
| 2025-12-19 | 初始版本：docs/guide, .claude/commands, CLAUDE.md |
