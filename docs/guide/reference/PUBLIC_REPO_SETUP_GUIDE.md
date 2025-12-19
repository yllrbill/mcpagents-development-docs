# 公共仓库部署指南

> 将 MCPAgents 文档部署到公共仓库 `yllrbill/mcpagents-development-docs` 的完整步骤。

---

## 前置条件

- [x] 私有仓库同步工作流已配置
- [x] 白名单清单已定义
- [x] 溯源文件生成逻辑已实现
- [ ] 公共仓库已创建
- [ ] Deploy Key 已配置

---

## Step 1: 创建公共仓库

1. 访问 https://github.com/new
2. 仓库名称：`mcpagents-development-docs`
3. 可见性：**Public**
4. 初始化：勾选 "Add a README file"
5. 点击 "Create repository"

---

## Step 2: 配置 Deploy Key

### 2.1 生成 SSH 密钥对

```powershell
# 在本地生成
ssh-keygen -t ed25519 -C "mcpagents-docs-sync" -f mcpagents_deploy_key -N ""
```

生成两个文件：
- `mcpagents_deploy_key` - 私钥
- `mcpagents_deploy_key.pub` - 公钥

### 2.2 添加公钥到公共仓库

1. 打开公共仓库：https://github.com/yllrbill/mcpagents-development-docs
2. Settings → Deploy keys → Add deploy key
3. Title: `Private Repo Sync`
4. Key: 粘贴 `mcpagents_deploy_key.pub` 的内容
5. **勾选** "Allow write access"
6. 点击 "Add key"

### 2.3 添加私钥到私有仓库 Secrets

1. 打开私有仓库：https://github.com/daodao97/chatmcp（或你的私有仓库）
2. Settings → Secrets and variables → Actions → New repository secret
3. Name: `PUBLIC_REPO_DEPLOY_KEY`
4. Secret: 粘贴 `mcpagents_deploy_key` 的**完整内容**（包括 BEGIN 和 END 行）
5. 点击 "Add secret"

### 2.4 安全处理

```powershell
# 删除本地密钥文件
Remove-Item mcpagents_deploy_key, mcpagents_deploy_key.pub
```

---

## Step 3: 配置公共仓库 CI

### 3.1 创建工作流文件

在公共仓库创建 `.github/workflows/docs-ci.yml`，内容如下：

```yaml
# 文档 CI：验证 + 部署到 GitHub Pages
name: Docs CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  # Job 1: 验证文档
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install dependencies
        run: pip install mkdocs-material pymdown-extensions

      - name: Build docs (strict mode)
        run: |
          mkdocs build --strict
          echo "✅ MkDocs strict build passed"

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'

      - name: Run markdownlint
        run: |
          npm install -g markdownlint-cli
          markdownlint docs/guide/**/*.md --config .markdownlint.json || true

      - name: Show source revision
        run: |
          if [ -f "docs/guide/reference/SOURCE_REVISION.txt" ]; then
            echo "📋 Source Revision:"
            cat docs/guide/reference/SOURCE_REVISION.txt
          fi

  # Job 2: 部署到 GitHub Pages
  deploy:
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    needs: validate
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Setup Python
        uses: actions/setup-python@v5
        with:
          python-version: '3.11'

      - name: Install MkDocs
        run: pip install mkdocs-material pymdown-extensions

      - name: Build site
        run: mkdocs build

      - name: Setup Pages
        uses: actions/configure-pages@v4

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: 'site'

      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

---

## Step 4: 启用 GitHub Pages

1. 打开公共仓库 Settings → Pages
2. Source: 选择 **GitHub Actions**
3. 保存

首次部署后，站点地址为：
```
https://yllrbill.github.io/mcpagents-development-docs/
```

---

## Step 5: 开启安全防护

### 5.1 启用 Secret Scanning

1. 公共仓库 Settings → Code security and analysis
2. Secret scanning: **Enable**
3. Push protection: **Enable**

### 5.2 配置分支保护（可选）

1. Settings → Branches → Add rule
2. Branch name pattern: `main`
3. 勾选：
   - Require a pull request before merging
   - Require status checks to pass before merging
   - 选择 `validate` 作为必须通过的检查

---

## Step 6: 首次同步

### 方式 A: 手动同步

```powershell
cd D:\claude1\MCPAgents
.\scripts\sync_to_public.ps1
```

### 方式 B: 触发 GitHub Actions

推送任何文档变更到私有仓库的 main 分支，自动触发同步。

---

## 验证清单

- [ ] 公共仓库已创建且为 Public
- [ ] Deploy Key 已配置（公钥在公共仓库，私钥在私有仓库 Secrets）
- [ ] 公共仓库 CI 工作流已创建
- [ ] GitHub Pages 已启用（Source: GitHub Actions）
- [ ] Secret scanning 和 Push protection 已开启
- [ ] 首次同步已完成
- [ ] Pages 站点可访问

---

## 部署后的使用

### 文档站点地址

```
https://yllrbill.github.io/mcpagents-development-docs/
```

### 新对话快速入门

```
这是 MCPAgents 的公开文档站点：
https://yllrbill.github.io/mcpagents-development-docs/

请先阅读：
- 首页（项目概述和阅读路线）
- 文档规范（术语表）
- 核心模块（架构详解）

以 reference/SOURCE_REVISION.txt 的 commit SHA 为版本基准。

然后给我：下一步开发建议 + 风险点 + 3 个可验收任务。
```

---

## 维护说明

### Deploy Key 轮换

建议每 90 天轮换一次：
1. 生成新密钥对
2. 更新公共仓库的 Deploy Key
3. 更新私有仓库的 Secret
4. 删除旧密钥

### 添加新的公开目录

1. 编辑 `docs/guide/reference/PUBLIC_EXPORT_SCOPE.md`
2. 更新同步工作流和脚本的白名单
3. 递增 `WHITELIST_VERSION`
4. 运行同步验证

---

## 相关文档

- [公开文档范围](PUBLIC_EXPORT_SCOPE.md)
- [LLM 快速入门模板](LLM_QUICKSTART_TEMPLATE.md)
- [文档研读与维护指南](../howto/docs_reading_maintenance_guide.md)
