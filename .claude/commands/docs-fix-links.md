# 修复文档链接

批量修复文档中的失效链接和锚点。

## 步骤

1. 运行严格构建获取问题清单：
   ```powershell
   cd D:\claude1\MCPAgents
   mkdocs build --strict 2>&1 | Select-String "WARNING|INFO.*anchor"
   ```

2. 分析问题类型：

   **锚点不存在** (`no such anchor`)：
   - 原因：目标页面标题变化，锚点 ID 也变了
   - 修复：打开目标页面，找到正确标题，复制其锚点

   **文件不存在** (`target is not found`)：
   - 原因：文件被移动/删除/重命名
   - 修复：更新链接路径或移除失效链接

   **外链** (指向 `docs_dir` 外)：
   - 如 `../DAEMON_DEVELOPMENT_GUIDE.md`
   - 选项 A：保留（不用 --strict）
   - 选项 B：复制/移动文件到 docs/guide/

3. 逐个修复：
   ```
   Read 问题文件 → Edit 修正链接 → mkdocs build 验证
   ```

4. 验证：
   ```powershell
   mkdocs build --strict
   ```

## 常见锚点格式

MkDocs 生成的锚点规则：
- 空格 → `-`
- 大写 → 小写
- 中文保留
- 特殊字符移除

示例：
- `## 1. 启动与停止` → `#1-启动与停止`
- `## Quick Start` → `#quick-start`
