# MCPAgents 文档规范

> 本文档定义了 MCPAgents Guide 系列文档的编写规范、术语表和维护准则。

---

## 1. 文档分类 (Diátaxis)

本项目采用 [Diátaxis](https://diataxis.fr/) 框架组织文档：

| 类型 | 用途 | 示例 |
|------|------|------|
| **Tutorial** | 学习导向，手把手引导 | INDEX.md 中的阅读路线 |
| **How-to** | 任务导向，解决特定问题 | 各扩展模块的 Interfaces 章节 |
| **Reference** | 信息导向，精确技术细节 | API 端点、配置项、状态机 |
| **Explanation** | 理解导向，解释设计决策 | 各模块的 Purpose、Architecture 章节 |

---

## 2. 文档模板

每份模块文档必须包含以下章节（缺项标注"暂缺/待补充"并指出补充来源）：

```markdown
# 模块名称
一句话定位（给新读者：这份文档解决什么问题）

## 1. Purpose（目的）
- 解决什么问题
- 不解决什么问题（明确边界）

## 2. Scope & Boundaries（范围与边界）
- 属于核心还是扩展
- 依赖哪些模块
- 对外暴露哪些能力（API/事件/CLI/SDK）

## 3. Responsibilities（职责）
用"动词+对象"列表，不超过 12 条

## 4. Architecture（架构）
- 关键组件（组件表/清单）
- 关键流程（Mermaid 时序图或流程图）
- 关键不变量（必须长期成立的规则）

## 5. Interfaces（接口）
- HTTP/SSE/CLI/MCP tools 等入口点
- 输入输出结构

## 6. Data & State（数据与状态）
- 数据落盘位置（路径、db、日志）
- 状态机（若有）

## 7. Failure & Recovery（失败与恢复）
- 常见失败类型
- 重试/熔断/降级/修复流程

## 8. Security & Privacy（安全与隐私）
- 权限边界
- 审计/日志
- 最小权限原则

## 9. Config（配置）
- 配置来源（文件/env/db）
- 热更新/重载策略

## 10. Test（测试）
- 单测/集成测试/Smoke
- 关键验收点

## 11. Roadmap（路线图）
- 已完成
- Next 2~4 个迭代点
```

---

## 3. 术语表

统一使用以下术语，避免歧义：

| 术语 | 定义 | 备注 |
|------|------|------|
| **Daemon** | 后台常驻服务进程 (mcpagentsd) | 单写者，管理所有状态 |
| **Runtime** | Daemon 的运行时环境 | 与 Daemon 同义 |
| **Run** | 一次 LLM 调用的完整生命周期 | 包含状态机、事件流 |
| **Router** | HTTP 路由层 | `apps/mcpagentsd/lib/src/router.dart` |
| **SmartRouter** | 智能路由服务 | 负责模型选择、fallback |
| **EventLog** | 事件日志系统 | JSONL 格式持久化 |
| **MCP** | Model Context Protocol | Anthropic 提出的工具协议 |
| **Tool** | MCP 工具 | 由 MCP Server 提供的能力 |
| **Agent** | 自主代理 | 能调用工具、做决策的 LLM |
| **Provider** | LLM 服务商 | openai, claude, ollama 等 |
| **Canonical Config** | 统一配置视图 | Daemon 作为 single source of truth |
| **GUI** | Flutter 图形界面 | `lib/` 目录 |
| **CLI** | 命令行工具 | `apps/mcpagents_cli/` |
| **SSE** | Server-Sent Events | 实时事件推送协议 |
| **CAS** | Compare-And-Set | 并发控制原语 |

---

## 4. 代码引用规范

### 4.1 路径引用格式

使用相对于项目根目录的路径，用括号注明：

```markdown
SmartRouter 位于 (`apps/mcpagentsd/lib/src/services/smart_router.dart`)
```

### 4.2 行号引用

对于关键代码位置，使用 `文件:行号` 格式：

```markdown
CAS 状态转换实现见 `run_repository.dart:45-67`
```

### 4.3 真实性检查

- 每个关键结论必须有代码/文件路径作为证据
- 找不到证据的内容必须标注 `TODO: 待补充，应查 xxx 目录`

---

## 5. Mermaid 图表规范

### 5.1 图表类型

| 类型 | 用途 | 示例 |
|------|------|------|
| `C4Context` | 系统与外部交互 | 用户、外部服务 |
| `C4Container` | 内部容器划分 | Daemon, GUI, CLI |
| `flowchart` | 流程/决策 | 路由决策流程 |
| `sequenceDiagram` | 时序交互 | SSE 断线续传 |
| `stateDiagram-v2` | 状态机 | Run 生命周期 |

### 5.2 命名规范

Mermaid 文件放置于 `docs/guide/diagrams/`：

- `c4_context.mmd` - C4 Context 图
- `c4_container.mmd` - C4 Container 图
- `core_eventflow.mmd` - 核心事件流

### 5.3 内联 vs 引用

- 简单图表（< 20 行）可内联在 Markdown 中
- 复杂图表应放在 `diagrams/` 目录并引用

---

## 6. 文档维护准则

### 6.1 更新原则

1. **代码先行**：代码变更后必须同步更新相关文档
2. **版本关联**：重大变更在 Roadmap 中标注版本号
3. **废弃标注**：废弃内容使用 `~~删除线~~` 并注明替代方案

### 6.2 审阅检查清单

- [ ] 所有路径引用可访问
- [ ] 术语使用一致
- [ ] Mermaid 图表可渲染
- [ ] 无死链
- [ ] INDEX.md 阅读路线覆盖新手/贡献者/架构理解者

### 6.3 版本号

文档版本跟随项目版本：当前 **v0.5.6**

---

## 7. 文件命名规范

```
docs/guide/
├── INDEX.md                        # 入口文档
├── 00_DOC_RULES.md                 # 本文档
├── 10_PROJECT_STRUCTURE.md         # 项目结构说明书
├── 20_CORE_MODULE.md               # 核心模块说明书
├── extensions/
│   ├── 30_TRAY.md                  # 托盘/桌面常驻
│   ├── 31_LOCAL_MODELS.md          # 本地模型/推理worker
│   ├── 32_LLM_INTEGRATIONS.md      # LLM 集成与自动化开发
│   ├── 33_SESSION_GUI.md           # 会话GUI
│   ├── 34_DESKTOP_AGENTS.md        # 桌面Agents
│   ├── 35_MCP_SERVERS.md           # MCP 服务器生态
│   ├── 36_SECURITY_PRIVACY.md      # 安全与隐私
│   └── 37_ADULT_NSFW.md            # 成人/NSFW 扩展
└── diagrams/
    ├── c4_context.mmd
    ├── c4_container.mmd
    └── core_eventflow.mmd
```

---

## 8. 特殊标记

| 标记 | 含义 |
|------|------|
| `TODO:` | 待完成内容 |
| `FIXME:` | 已知问题待修复 |
| `NOTE:` | 重要提示 |
| `WARNING:` | 警告信息 |
| `DEPRECATED:` | 已废弃，将移除 |

---

## 9. ADR (Architecture Decision Records) 规范

### 9.1 何时写 ADR

在以下情况下需要创建 ADR：

- 引入新的核心架构模式
- 选择特定技术/库/协议
- 定义系统边界和约束
- 改变已有架构决策

### 9.2 ADR 模板

每个 ADR 遵循 Michael Nygard 模板：

```markdown
# ADR-XXXX: 标题

## 状态
Proposed / Accepted / Deprecated / Superseded by ADR-YYYY

## 上下文
我们面临什么问题？需要做什么决策？

## 决策
我们决定采用什么方案？

## 理由
为什么选择这个方案而不是其他备选？

## 后果
- 正面影响
- 负面影响
- 需要注意的事项

## 备选方案（可选）
考虑过但未采用的其他方案及其原因。
```

### 9.3 编号规则

- 格式：`XXXX-short-title.md`
- 编号：四位数字，从 0001 开始递增
- 一旦创建，编号不复用
- Superseded 的 ADR 保留，在头部标注被谁取代

### 9.4 存放位置

ADR 文档存放于 `docs/guide/adr/` 目录：

```
docs/guide/adr/
├── README.md                          # ADR 索引
├── 0001-single-writer-daemon.md       # 单写者架构决策
├── 0002-sse-event-streaming.md        # SSE 事件流决策
└── 0003-cas-state-machine.md          # CAS 状态机决策
```

### 9.5 现有 ADR 列表

| 编号 | 标题 | 状态 |
|------|------|------|
| [ADR-0001](adr/0001-single-writer-daemon.md) | 单写者 Daemon 架构 | ✅ 已采纳 |
| [ADR-0002](adr/0002-sse-event-streaming.md) | SSE 事件流协议 | ✅ 已采纳 |
| [ADR-0003](adr/0003-cas-state-machine.md) | CAS 状态机并发控制 | ✅ 已采纳 |
