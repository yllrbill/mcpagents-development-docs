# MCPAgents - Claude Code 项目上下文

> 本文件供 Claude Code 自动加载，包含项目核心信息、工作流命令和验收标准。

---

## 项目概述

MCPAgents 是跨平台 AI Chat 客户端 + MCP 运行时，采用 Dart/Flutter 开发。

### 核心架构

- **单写者 Daemon**: `mcpagentsd` 是唯一状态管理者
- **SSE 事件流**: 实时推送 + Last-Event-ID 断线续传
- **CAS 状态机**: Compare-And-Set 保证并发安全

### 关键路径

| 组件 | 路径 |
|------|------|
| Daemon | `apps/mcpagentsd/` |
| CLI | `apps/mcpagents_cli/` |
| GUI | `lib/` |
| 文档 | `docs/guide/` |
| 配置 | `%APPDATA%\MCPAgents\` |

---

## 文档体系

### 入口文件

- **首页**: `docs/guide/index.md`
- **文档规范**: `docs/guide/00_DOC_RULES.md`
- **MkDocs 配置**: `mkdocs.yml`

### 目录结构

```
docs/guide/
├── index.md                    # 首页（阅读路线）
├── 00_DOC_RULES.md             # 文档规范 + 术语表
├── 10_PROJECT_STRUCTURE.md     # 项目结构
├── 20_CORE_MODULE.md           # 核心模块
├── adr/                        # 架构决策记录
├── reference/                  # 配置参考
├── runbook/                    # 运维手册
├── howto/                      # 开发指南
├── extensions/                 # 扩展模块文档
└── diagrams/                   # Mermaid 图表
```

### 文档构建命令

```powershell
# 安装依赖
pip install -r docs/guide/requirements-docs.txt

# 本地预览（开发模式）
cd D:\claude1\MCPAgents
mkdocs serve

# 构建静态站点
mkdocs build

# 严格模式构建（CI 验收）
mkdocs build --strict

# 部署到 GitHub Pages
mkdocs gh-deploy
```

---

## 修改文档的工作流

### 必须遵循的流程

1. **先只读，不写**: 使用 Read 工具读取相关文件
2. **输出修改计划**: 说明要改什么、为什么改
3. **执行修改**: 使用 Edit 工具（不是 Write）
4. **运行校验**: `mkdocs build` 或 `mkdocs build --strict`
5. **汇报结果**: 说明改了什么、校验是否通过

### 修改规则

- **先 Read 再 Edit**: 修改任何文件前必须先读取，否则工具会报错
- **Windows 文件名**: 不要仅改大小写来"重命名"（Windows 不区分大小写）
- **新增页面**: 必须更新 `mkdocs.yml` 的 `nav`，并在 `index.md` 补链接
- **新增 ADR**: 必须更新 `adr/README.md` 索引和 `00_DOC_RULES.md` ADR 列表

### Definition of Done（验收标准）

- [ ] `mkdocs build` 通过（无 ERROR）
- [ ] 发布级修改需 `mkdocs build --strict` 通过
- [ ] 新增/修改的链接可跳转
- [ ] 新增页面已加入导航
- [ ] 术语使用符合 `00_DOC_RULES.md` 术语表

---

## 常用命令速查

### 文档相关

```powershell
# 预览文档站点
mkdocs serve

# 构建（开发）
mkdocs build

# 构建（严格/CI）
mkdocs build --strict

# 检查特定文件链接
# 看 mkdocs build 输出的 WARNING
```

### Daemon 相关

```powershell
# 启动 Daemon
D:\claude1\MCPAgents\apps\mcpagentsd\bin\mcpagentsd.exe serve --port 8787

# 健康检查
curl http://127.0.0.1:8787/v1/health

# 停止
curl -X POST http://127.0.0.1:8787/admin/shutdown -H "Authorization: Bearer TOKEN"
```

### 代码格式化

```powershell
# Dart 格式化
dart format .

# Dart 分析
dart analyze
```

---

## 项目关键决策（ADR 摘要）

| ADR | 决策 | 理由 |
|-----|------|------|
| ADR-0001 | 单写者 Daemon | 避免 CRDT 复杂性，SQLite 足够 |
| ADR-0002 | SSE 事件流 | 比 WebSocket 更适合单向推送 |
| ADR-0003 | CAS 状态机 | 无锁、无死锁、SQLite 原生支持 |

详见 `docs/guide/adr/`

---

## 注意事项

### Windows 特有问题

1. **路径分隔符**: 使用 `\` 或 `/` 都可以，但 PowerShell 推荐 `\`
2. **文件名大小写**: Windows 不区分，不要用大小写变化来重命名
3. **终端编码**: 中文可能显示乱码，但不影响实际功能

### Claude Code 工具使用

1. **Read before Edit**: 必须先读文件才能编辑
2. **Edit vs Write**: 优先用 Edit（增量修改），Write 会覆盖整个文件
3. **Bash 路径**: Windows 下用 PowerShell 命令，注意路径引号

---

## 相关文档

- [文档规范](docs/guide/00_DOC_RULES.md)
- [运维手册](docs/guide/runbook/daemon_ops.md)
- [故障排查](docs/guide/runbook/troubleshooting.md)
- [扩展开发](docs/guide/howto/add_extension.md)
