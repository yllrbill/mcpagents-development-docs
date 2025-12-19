# 文档严格构建（CI 级验收）

使用 `--strict` 模式构建，任何 WARNING 都会导致构建失败。

## 步骤

1. 运行严格构建：
   ```powershell
   cd D:\claude1\MCPAgents
   mkdocs build --strict
   ```

2. 如果失败，查看具体 WARNING：
   - `missing anchor`：目标页面的锚点不存在
   - `target is not found`：链接指向的文件不存在
   - `link ... but the target ... is not found`：外链指向 docs_dir 外

3. 修复策略：
   - 锚点问题：打开目标页面，复制正确的锚点链接
   - 文件不存在：修正路径或移除失效链接
   - 外链：如果是故意的外链可以忽略，或把文件移入 docs/guide/

4. 重复直到通过

## 验收标准

- `mkdocs build --strict` 无 WARNING 无 ERROR
- 适用于：发版、合并 PR、CI 流水线
