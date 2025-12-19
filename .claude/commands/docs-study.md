# 研读主题并产出摘要

按关键词研读文档，产出摘要、知识点和待补清单。

## 参数

- `$ARGUMENTS`：研读主题关键词（如 "SSE"、"CAS"、"路由策略"）

## 研读流程

### 1. 搜索定位

在以下位置搜索关键词 `$ARGUMENTS`：

```
docs/guide/index.md
docs/guide/20_CORE_MODULE.md
docs/guide/adr/*.md
docs/guide/reference/*.md
docs/guide/runbook/*.md
docs/guide/extensions/*.md
```

### 2. 阅读并提取

对每个匹配文件：
- Read 全文
- 提取与关键词相关的核心概念
- 记录定义、规则、约束
- 标记代码位置引用

### 3. 产出摘要

输出格式：

```markdown
## 主题：$ARGUMENTS

### 核心概念
- 概念 1：定义...
- 概念 2：定义...

### 关键规则/约束
1. 规则 1
2. 规则 2

### 代码位置
- `path/to/file.dart:行号` - 说明

### 相关文档
- [文档名](路径) - 关联说明

### 知识缺口（待补）
- [ ] 缺口 1 - 应查 xxx 目录
- [ ] 缺口 2 - 需要代码验证
```

### 4. 可选：生成学习笔记

如果用户要求，可以将摘要写入：
`docs/guide/notes/{关键词}_study_notes.md`

## 常用研读主题

| 主题 | 核心文档 |
|------|----------|
| Run 生命周期 | `20_CORE_MODULE.md`, `adr/0003-cas-state-machine.md` |
| SSE 事件流 | `adr/0002-sse-event-streaming.md`, `20_CORE_MODULE.md` |
| CAS 状态机 | `adr/0003-cas-state-machine.md` |
| 单写者架构 | `adr/0001-single-writer-daemon.md` |
| 智能路由 | `20_CORE_MODULE.md`, `reference/config_reference.md` |
| MCP 工具 | `extensions/35_MCP_SERVERS.md` |
| 安全策略 | `extensions/36_SECURITY_PRIVACY.md` |
| 配置系统 | `reference/config_reference.md` |

## 验收标准

- [ ] 关键词在所有相关文档中被检索
- [ ] 核心概念有明确定义
- [ ] 规则/约束有文档引用支撑
- [ ] 知识缺口已标注待补来源
