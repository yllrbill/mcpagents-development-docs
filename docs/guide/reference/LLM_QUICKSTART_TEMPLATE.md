# LLM 快速入门模板

> 在新对话中使用此模板，让 LLM 快速进入项目上下文。

---

## 基础模板（复制粘贴即可）

```
这是 MCPAgents 的公开镜像仓（包含 docs/guide + .claude）。请先通读：
- docs/guide/index.md
- docs/guide/00_DOC_RULES.md
- docs/guide/20_CORE_MODULE.md
- docs/guide/reference/config_reference.md

并以 reference/SOURCE_REVISION.txt 的 SHA 作为版本基准。

然后给我：下一步开发建议 + 风险点 + 3 个可验收任务。
```

---

## 按场景定制的模板

### 场景 A：架构理解

```
这是 MCPAgents 的公开文档仓库。请阅读：
- docs/guide/index.md（项目概述）
- docs/guide/10_PROJECT_STRUCTURE.md（项目结构）
- docs/guide/20_CORE_MODULE.md（核心模块）
- docs/guide/adr/（架构决策记录）

然后回答：
1. 系统的核心架构是什么？
2. 有哪些关键的设计决策？为什么这样设计？
3. 有哪些潜在的技术债务或改进空间？
```

### 场景 B：运维排障

```
这是 MCPAgents 的公开文档仓库。请阅读：
- docs/guide/runbook/daemon_ops.md（运维手册）
- docs/guide/runbook/troubleshooting.md（故障排查）
- docs/guide/reference/config_reference.md（配置参考）

我遇到的问题是：[描述问题]

请给出：
1. 可能的原因分析
2. 诊断步骤
3. 解决方案
```

### 场景 C：扩展开发

```
这是 MCPAgents 的公开文档仓库。请阅读：
- docs/guide/howto/add_extension.md（扩展开发指南）
- docs/guide/extensions/32_LLM_INTEGRATIONS.md（LLM 集成）
- docs/guide/20_CORE_MODULE.md（核心模块）

我想要：[描述需求]

请给出：
1. 实现方案设计
2. 需要修改的文件列表
3. 测试验收点
```

### 场景 D：文档维护

```
这是 MCPAgents 的公开文档仓库。请阅读：
- docs/guide/00_DOC_RULES.md（文档规范）
- docs/guide/howto/docs_reading_maintenance_guide.md（研读与维护指南）
- .claude/commands/（工作流命令）

我需要：[描述文档任务]

请按照文档规范执行，并在完成后运行 mkdocs build 验证。
```

### 场景 E：安全审查

```
这是 MCPAgents 的公开文档仓库。请阅读：
- docs/guide/extensions/36_SECURITY_PRIVACY.md（安全与隐私）
- docs/guide/reference/config_reference.md（配置参考，特别是 security 和 mcp.security）
- docs/guide/adr/0001-single-writer-daemon.md（单写者架构）

请审查并回答：
1. 当前有哪些安全机制？
2. 有哪些潜在的安全风险？
3. 建议的安全加固措施？
```

---

## 进阶用法

### 带版本基准的模板

```
这是 MCPAgents 公开文档仓库。

版本信息：
- SOURCE_REVISION.txt 中的 commit: [填入 SHA]
- 白名单版本: [填入版本号]

请基于此版本阅读文档并给出建议。
```

### 多轮对话的上下文恢复

```
我们之前讨论了 MCPAgents 的 [主题]。
公开文档仓库：https://github.com/yllrbill/mcpagents-development-docs

请继续上次的讨论，重点关注：[新的焦点]
```

---

## 公开仓库信息

| 项目 | 值 |
|------|-----|
| 仓库地址 | https://github.com/yllrbill/mcpagents-development-docs |
| 文档站点 | https://yllrbill.github.io/mcpagents-development-docs/ |
| 溯源文件 | docs/guide/reference/SOURCE_REVISION.txt |
| 白名单清单 | docs/guide/reference/PUBLIC_EXPORT_SCOPE.md |

---

## 注意事项

1. **版本对齐**：始终检查 SOURCE_REVISION.txt 确认文档版本
2. **隐私保护**：公开仓库不包含源码和敏感配置
3. **建议验证**：LLM 的建议需要在私有仓库中验证
4. **代码参考**：如需具体代码，需要查看私有仓库
