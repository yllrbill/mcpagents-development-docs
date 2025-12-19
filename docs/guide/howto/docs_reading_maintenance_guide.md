# Claude Code 研读与维护指南（Docs Guide）

本指南用于让 Claude Code 在 MCPAgents 仓库中 **稳定、可验证** 地阅读和维护 `docs/guide` 文档站点（MkDocs + Material）。

---

## 0. 最重要的原则（必须遵守）

1. **先读后写**：任何 Write/Edit 之前，先 Read 目标文件与相关索引文件。
2. **改动必须可验收**：完成修改后必须执行 `mkdocs build`；发布级改动必须 `mkdocs build --strict`。
3. **保持导航一致**：新增/移动/删除页面时，必须同步更新：
   - `mkdocs.yml` 的 nav
   - `docs/guide/index.md`（或入口索引页）
   - 若涉及 ADR：`docs/guide/adr/README.md` 与 `00_DOC_RULES.md`
4. **Windows 大小写陷阱**：不要通过仅改变文件名大小写来"重命名"（Windows 会把它们视为同名）。

---

## 1. 研读入口与阅读路线（Claude Code 应该怎么读）

### 1.1 默认入口（必须）

- `docs/guide/index.md`：站点入口与阅读路线
- `docs/guide/00_DOC_RULES.md`：术语、模板、写作规范
- `mkdocs.yml`：站点导航与构建规则

### 1.2 根据任务快速补上下文（按需）

| 任务类型 | 阅读路径 |
|----------|----------|
| 架构理解 | `10_PROJECT_STRUCTURE.md` → `20_CORE_MODULE.md` → `adr/*` → `diagrams/*` |
| 运维排障 | `runbook/daemon_ops.md` → `runbook/troubleshooting.md` |
| 扩展开发 | `howto/add_extension.md` → `extensions/*` |
| 安全隐私 | `extensions/36_SECURITY_PRIVACY.md` |
| 配置详情 | `reference/config_reference.md` |

---

## 2. Claude Code 标准工作流（每次文档任务都按这套走）

> 使用项目命令：
> - `/project:docs-task <任务描述>`（通用流程）
> - `/project:docs-build`、`/project:docs-strict`、`/project:docs-fix-links`、`/project:docs-add-adr`（专项）
> - `/project:docs-study <关键词>`（研读并产出摘要）

### Step A：只读与计划

1. Read 入口与规则文件（见 1.1）
2. Read 任务涉及的页面
3. 输出修改计划（列出要改哪些文件、原因、影响范围）

### Step B：执行修改

- Edit/Write 相关页面（遵守模板与术语）
- 如新增页面：补 `mkdocs.yml nav` + 入口索引链接

### Step C：校验

1. `mkdocs build`
2. 若是发布级/合并前：`mkdocs build --strict`
3. 若启用 markdownlint：执行 lint（按仓库约定）

> **说明**：MkDocs 的 `--strict` 会让任何 WARNING 导致构建失败，因此适合作为 CI 验收门槛。

### Step D：汇报

输出：
- 改动文件清单
- 每个文件 1~3 条摘要
- 校验结果（build/strict/lint）
- 若仍有 TODO：明确 TODO 的补证据位置（应查哪个目录/文件）

---

## 3. 常见问题处理手册（Claude Code 快速修复）

### 3.1 锚点不存在（no such anchor）

**原因**：标题改动导致自动生成的锚点变化。

**做法**：
- 打开目标页面，定位标题，改链接锚点为页面实际生成的锚点（不要手写猜测）。
- 修复后 `mkdocs build --strict` 再验证。

**MkDocs 锚点生成规则**：
- 空格 → `-`
- 大写 → 小写
- 中文保留
- 特殊字符移除

### 3.2 文件不存在（target is not found）

**原因**：页面被移动/删除/重命名。

**做法**：
- 更新链接到正确路径，或移除该链接。
- 若重要内容被引用：考虑移动到 `docs/guide/reference/` 再链接。

### 3.3 snippets 未生效（--8<-- 原样显示）

**原因**：缺少 `pymdownx.snippets` 或 base_path 配置不正确。

**做法**：
- 确认 `mkdocs.yml` 的 `markdown_extensions` 含 `pymdownx.snippets`
- 必要时设置 base_path 为 docs_dir 相对路径
- 重新 `mkdocs build` 验证

### 3.4 Mermaid 图不渲染

**原因**：`pymdownx.superfences` 的 mermaid fence 配置缺失。

**做法**：
- 确认 `mkdocs.yml` 中 `pymdownx.superfences.custom_fences` 包含 mermaid 配置
- 使用 ` ```mermaid ` 代码块包裹图表代码

---

## 4. Material 站点阅读技巧（让研读更快）

### 4.1 键盘快捷键

| 快捷键 | 功能 |
|--------|------|
| `F` / `S` / `/` | 打开搜索框 |
| `P` / `,` | 上一页 |
| `N` / `.` | 下一页 |
| `↑` `↓` | 搜索结果中上下选择 |
| `Enter` | 打开选中结果 |
| `Esc` | 关闭搜索 |

### 4.2 高效研读建议

1. **先搜后读**：搜索关键词（如 "Run / CAS / SSE / Scope / audit / routing"）再进入页面，比手动翻目录更快。

2. **多标签页**：右键在新标签页打开多个相关页面，方便对比。

3. **跟踪阅读路径**：在笔记中记录 "从哪篇跳过来的"，便于以后回溯。

4. **关注 ADR**：架构决策记录是理解"为什么这样设计"的最佳入口。

---

## 5. 文档变更 PR Checklist（提交前必过）

- [ ] 入口与导航一致（index/nav/ADR 索引已同步）
- [ ] 术语统一（Daemon/Run/Router/MCP/SSE 等，参见 `00_DOC_RULES.md#术语表`）
- [ ] `mkdocs build` 通过
- [ ] 发布/合并：`mkdocs build --strict` 通过
- [ ] 重要结论有证据（链接到仓库路径/文件）
- [ ] 新增页面已加入导航
- [ ] 新增 ADR 已更新索引

---

## 6. 项目命令速查

| 命令 | 用途 |
|------|------|
| `/project:docs-task <描述>` | 通用文档维护任务 |
| `/project:docs-build` | 构建文档站点 |
| `/project:docs-strict` | 严格模式构建（CI 级验收） |
| `/project:docs-fix-links` | 修复失效链接 |
| `/project:docs-add-adr <标题>` | 添加新 ADR |
| `/project:docs-study <关键词>` | 研读主题并产出摘要 |

---

## 相关文档

- [文档规范](../00_DOC_RULES.md)
- [扩展开发指南](add_extension.md)
- [Daemon 运维手册](../runbook/daemon_ops.md)
- [CLAUDE.md](../../../CLAUDE.md) - 项目上下文说明书
