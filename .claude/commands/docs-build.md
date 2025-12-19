# 文档构建

构建 MkDocs 文档站点。

## 步骤

1. 确保依赖已安装：
   ```powershell
   pip install -r docs/guide/requirements-docs.txt
   ```

2. 运行构建：
   ```powershell
   cd D:\claude1\MCPAgents
   mkdocs build
   ```

3. 检查输出：
   - 成功：`Documentation built in X.XX seconds`
   - 失败：查看 WARNING/ERROR 信息

4. 如有 WARNING：
   - 锚点不存在：检查目标页面标题是否变化
   - 链接不存在：检查文件路径是否正确
   - 逐个修复后重新构建

## 验收标准

- `mkdocs build` 无 ERROR
- 输出目录 `site/` 生成成功
