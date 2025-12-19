# 文档维护任务

通用的文档维护工作流。

## 参数

- `$ARGUMENTS`：具体任务描述

## 标准工作流

你将维护 `docs/guide` 文档站点（MkDocs + Material）。

**流程必须是**：先只读（不要写）→ 输出修改计划 → 执行修改 → 运行校验 → 汇报结果。

### 1. 开始时请依次 Read

- `docs/guide/index.md`
- `docs/guide/00_DOC_RULES.md`
- `mkdocs.yml`
- `docs/guide/runbook/daemon_ops.md`（文档构建段落）

### 2. 修改规则

- Windows 下不要仅改文件名大小写来"重命名"
- 修改任何文件前先 Read，再 Edit
- 新增页面必须更新 `mkdocs.yml` 的 nav，并在 index/索引中补链接（若适用）
- 新增 ADR 必须更新 `adr/README.md` 和 `00_DOC_RULES.md`

### 3. 验收

- `mkdocs build`（通过）
- 如是发布级修改：`mkdocs build --strict`（通过）
- 如项目启用 markdownlint：lint 通过

### 4. 汇报

修改完成后，说明：
- 改了哪些文件
- 改动摘要
- 校验结果

## 常用子任务

| 任务 | 推荐命令 |
|------|----------|
| 构建文档 | `/project:docs-build` |
| 严格构建 | `/project:docs-strict` |
| 修复链接 | `/project:docs-fix-links` |
| 添加 ADR | `/project:docs-add-adr` |
