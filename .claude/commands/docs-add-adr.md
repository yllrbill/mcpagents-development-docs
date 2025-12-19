# 添加新 ADR

创建新的架构决策记录 (Architecture Decision Record)。

## 参数

- `$ARGUMENTS`：ADR 标题（如 "智能路由策略"）

## 步骤

1. 确定编号：
   ```powershell
   Get-ChildItem "D:\claude1\MCPAgents\docs\guide\adr\*.md" |
     Where-Object { $_.Name -match '^\d{4}' } |
     Sort-Object Name |
     Select-Object -Last 1
   ```
   新编号 = 最后一个 + 1

2. 创建 ADR 文件：
   使用模板创建 `docs/guide/adr/XXXX-short-title.md`

3. ADR 模板：
   ```markdown
   # ADR-XXXX: 标题

   ## 状态

   **Proposed** (YYYY-MM)

   ---

   ## 上下文

   我们面临什么问题？需要做什么决策？

   ---

   ## 决策

   我们决定采用什么方案？

   ---

   ## 理由

   ### 对比备选方案

   | 方案 | 优点 | 缺点 |
   |------|------|------|
   | **方案 A (采用)** | ... | ... |
   | 方案 B | ... | ... |

   ---

   ## 后果

   ### 正面影响

   1. ...

   ### 负面影响

   1. ...

   ---

   ## 代码位置

   - 相关代码：`path/to/file.dart`
   ```

4. 更新索引文件：
   - `docs/guide/adr/README.md`：添加到 ADR 索引表
   - `docs/guide/00_DOC_RULES.md`：添加到 9.5 现有 ADR 列表

5. 更新导航：
   - `mkdocs.yml`：在 `架构决策 (ADR)` 下添加新条目

6. 验证：
   ```powershell
   mkdocs build
   ```

## 验收标准

- [ ] ADR 文件已创建且格式正确
- [ ] `adr/README.md` 索引已更新
- [ ] `00_DOC_RULES.md` ADR 列表已更新
- [ ] `mkdocs.yml` nav 已更新
- [ ] `mkdocs build` 通过
