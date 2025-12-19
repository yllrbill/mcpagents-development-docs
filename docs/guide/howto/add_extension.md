# 如何开发 MCPAgents 扩展模块

> 本指南介绍如何为 MCPAgents 开发新的扩展模块、Provider 或 MCP Server。

---

## 1. 扩展类型概述

MCPAgents 支持多种扩展方式：

| 扩展类型 | 难度 | 位置 | 需要重新编译 |
|----------|------|------|--------------|
| 新 LLM Provider | ⭐⭐⭐ | `packages/mcpagents_core/lib/src/providers/` | ✅ |
| 新 MCP Server | ⭐⭐ | 外部 + 配置 | ❌ |
| 新 GUI 页面 | ⭐⭐⭐ | `lib/` | ✅ |
| 新 CLI 命令 | ⭐⭐ | `apps/mcpagents_cli/lib/` | ✅ |
| 新路由策略 | ⭐⭐⭐⭐ | `packages/mcpagents_core/lib/src/router/` | ✅ |

---

## 2. 添加新 LLM Provider

### 2.1 创建 Provider 类

**位置**: `packages/mcpagents_core/lib/src/providers/`

```dart
// my_provider.dart
import 'package:mcpagents_core/src/providers/base_provider.dart';

class MyProvider extends BaseProvider {
  @override
  String get name => 'myprovider';

  @override
  String get displayName => 'My Custom Provider';

  @override
  List<String> get defaultModels => ['my-model-1', 'my-model-2'];

  @override
  bool get supportsStreaming => true;

  @override
  Future<void> initialize(ProviderConfig config) async {
    _apiKey = config.apiKey ?? await _loadApiKeyFromPath(config.apiKeyPath);
    _baseUrl = config.baseUrl ?? 'https://api.myprovider.com/v1';
  }

  @override
  Future<LlmResponse> complete(LlmRequest request) async {
    final response = await _httpClient.post(
      Uri.parse('$_baseUrl/chat/completions'),
      headers: {
        'Authorization': 'Bearer $_apiKey',
        'Content-Type': 'application/json',
      },
      body: jsonEncode({
        'model': request.model,
        'messages': request.messages.map((m) => m.toJson()).toList(),
        'stream': false,
      }),
    );

    return LlmResponse.fromJson(jsonDecode(response.body));
  }

  @override
  Stream<LlmDelta> completeStream(LlmRequest request) async* {
    final request = await _httpClient.send(
      Request('POST', Uri.parse('$_baseUrl/chat/completions'))
        ..headers.addAll({
          'Authorization': 'Bearer $_apiKey',
          'Content-Type': 'application/json',
        })
        ..body = jsonEncode({
          'model': request.model,
          'messages': request.messages.map((m) => m.toJson()).toList(),
          'stream': true,
        }),
    );

    await for (final chunk in request.stream.transform(utf8.decoder).transform(LineSplitter())) {
      if (chunk.startsWith('data: ')) {
        final data = chunk.substring(6);
        if (data == '[DONE]') break;

        final json = jsonDecode(data);
        yield LlmDelta(
          content: json['choices'][0]['delta']['content'] ?? '',
          finishReason: json['choices'][0]['finish_reason'],
        );
      }
    }
  }
}
```

### 2.2 注册 Provider

**文件**: `packages/mcpagents_core/lib/src/providers/provider_registry.dart`

```dart
// 添加导入
import 'my_provider.dart';

class ProviderRegistry {
  static final Map<String, BaseProvider Function()> _factories = {
    'openai': () => OpenAIProvider(),
    'claude': () => ClaudeProvider(),
    'ollama': () => OllamaProvider(),
    // 添加你的 Provider
    'myprovider': () => MyProvider(),
  };
}
```

### 2.3 添加配置支持

**更新**: `packages/mcpagents_core/lib/src/config/provider_config.dart`

```dart
enum ProviderType {
  openai,
  claude,
  ollama,
  poe,
  myprovider,  // 添加枚举
}
```

### 2.4 添加模型定义

**文件**: `%APPDATA%\MCPAgents\models.json`

```json
{
  "models": {
    "myprovider:my-model-1": {
      "provider": "myprovider",
      "model_id": "my-model-1",
      "supports_vision": false,
      "supports_audio": false,
      "supports_tools": true,
      "context_window": 32000,
      "cost_weight": 0.5
    }
  }
}
```

### 2.5 测试

```dart
// test/providers/my_provider_test.dart
void main() {
  group('MyProvider', () {
    late MyProvider provider;

    setUp(() async {
      provider = MyProvider();
      await provider.initialize(ProviderConfig(
        apiKey: 'test-key',
      ));
    });

    test('complete returns valid response', () async {
      final response = await provider.complete(LlmRequest(
        model: 'my-model-1',
        messages: [Message.user('Hello')],
      ));

      expect(response.content, isNotEmpty);
    });

    test('completeStream yields deltas', () async {
      final deltas = <LlmDelta>[];

      await for (final delta in provider.completeStream(LlmRequest(
        model: 'my-model-1',
        messages: [Message.user('Hello')],
      ))) {
        deltas.add(delta);
      }

      expect(deltas, isNotEmpty);
    });
  });
}
```

---

## 3. 添加新 MCP Server

### 3.1 使用现有 MCP Server

最简单的方式是配置现有的 MCP Server：

```json
// %APPDATA%\MCPAgents\assets\mcp_server.json
{
  "mcpServers": {
    "my-tool": {
      "command": "npx",
      "args": ["-y", "@myorg/mcp-server-mytool"],
      "env": {
        "MY_API_KEY": "xxx"
      },
      "transport": "stdio"
    }
  }
}
```

### 3.2 开发自定义 MCP Server

#### 使用 Python (mcp 库)

```python
# my_mcp_server.py
from mcp.server import Server
from mcp.server.stdio import stdio_server

app = Server("my-tool")

@app.list_tools()
async def list_tools():
    return [
        {
            "name": "my_tool_action",
            "description": "Performs a custom action",
            "inputSchema": {
                "type": "object",
                "properties": {
                    "input": {"type": "string", "description": "Input text"}
                },
                "required": ["input"]
            }
        }
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict):
    if name == "my_tool_action":
        result = do_something(arguments["input"])
        return {"content": [{"type": "text", "text": result}]}
    raise ValueError(f"Unknown tool: {name}")

if __name__ == "__main__":
    import asyncio
    asyncio.run(stdio_server(app).run())
```

#### 使用 Node.js

```javascript
// my-mcp-server/index.js
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";

const server = new Server({
  name: "my-tool",
  version: "1.0.0",
}, {
  capabilities: {
    tools: {}
  }
});

server.setRequestHandler("tools/list", async () => ({
  tools: [{
    name: "my_tool_action",
    description: "Performs a custom action",
    inputSchema: {
      type: "object",
      properties: {
        input: { type: "string" }
      },
      required: ["input"]
    }
  }]
}));

server.setRequestHandler("tools/call", async (request) => {
  if (request.params.name === "my_tool_action") {
    const result = doSomething(request.params.arguments.input);
    return { content: [{ type: "text", text: result }] };
  }
  throw new Error(`Unknown tool: ${request.params.name}`);
});

const transport = new StdioServerTransport();
await server.connect(transport);
```

### 3.3 注册 MCP Server

```json
// mcp_server.json
{
  "mcpServers": {
    "my-tool": {
      "command": "python",
      "args": ["D:/path/to/my_mcp_server.py"],
      "transport": "stdio"
    }
  }
}
```

### 3.4 配置安全策略

```json
// canonical_config.json
{
  "mcp": {
    "security": {
      "allowed_tools": ["my-tool.*"],
      "blocked_tools": []
    }
  }
}
```

---

## 4. 添加新 GUI 功能

### 4.1 添加新页面

**位置**: `lib/page/`

```dart
// lib/page/my_feature_page.dart
import 'package:flutter/material.dart';

class MyFeaturePage extends StatefulWidget {
  const MyFeaturePage({super.key});

  @override
  State<MyFeaturePage> createState() => _MyFeaturePageState();
}

class _MyFeaturePageState extends State<MyFeaturePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('My Feature')),
      body: Center(
        child: Text('My new feature'),
      ),
    );
  }
}
```

### 4.2 添加路由

**文件**: `lib/router.dart`

```dart
import 'page/my_feature_page.dart';

final router = GoRouter(
  routes: [
    // ... 现有路由
    GoRoute(
      path: '/my-feature',
      builder: (context, state) => const MyFeaturePage(),
    ),
  ],
);
```

### 4.3 添加侧边栏入口

**文件**: `lib/widgets/sidebar.dart`

```dart
ListTile(
  leading: const Icon(Icons.new_feature),
  title: const Text('My Feature'),
  onTap: () => context.go('/my-feature'),
),
```

---

## 5. 添加新 CLI 命令

### 5.1 创建命令类

**位置**: `apps/mcpagents_cli/lib/commands/`

```dart
// my_command.dart
import 'package:args/command_runner.dart';

class MyCommand extends Command {
  @override
  String get name => 'mycommand';

  @override
  String get description => 'Does something useful';

  MyCommand() {
    argParser.addOption(
      'input',
      abbr: 'i',
      help: 'Input value',
      mandatory: true,
    );
    argParser.addFlag(
      'verbose',
      abbr: 'v',
      help: 'Verbose output',
      defaultsTo: false,
    );
  }

  @override
  Future<void> run() async {
    final input = argResults!['input'] as String;
    final verbose = argResults!['verbose'] as bool;

    if (verbose) {
      print('Processing: $input');
    }

    // 实现逻辑
    final result = await doSomething(input);
    print('Result: $result');
  }
}
```

### 5.2 注册命令

**文件**: `apps/mcpagents_cli/bin/main.dart`

```dart
import 'package:mcpagents_cli/commands/my_command.dart';

void main(List<String> args) {
  final runner = CommandRunner('mcpagents', 'MCPAgents CLI')
    ..addCommand(DaemonCommand())
    ..addCommand(ChatCommand())
    ..addCommand(MyCommand());  // 添加命令

  runner.run(args);
}
```

### 5.3 测试

```bash
dart run bin/main.dart mycommand --input "test" --verbose
```

---

## 6. 测试与发布

### 6.1 运行测试

```bash
# 单元测试
cd packages/mcpagents_core
dart test

# 集成测试
cd apps/mcpagentsd
dart test

# 全部测试
dart test --coverage
```

### 6.2 代码格式化

```bash
dart format .
dart analyze
```

### 6.3 文档更新

扩展开发完成后，需要更新相关文档：

1. **如果添加了新 Provider**:
   - 更新 `docs/guide/extensions/32_LLM_INTEGRATIONS.md`
   - 更新 `docs/guide/reference/config_reference.md`

2. **如果添加了新 MCP Server**:
   - 更新 `docs/guide/extensions/35_MCP_SERVERS.md`

3. **如果添加了新 GUI 功能**:
   - 更新 `docs/guide/extensions/33_SESSION_GUI.md`

### 6.4 提交流程

```bash
# 1. 创建分支
git checkout -b feature/my-extension

# 2. 提交更改
git add .
git commit -m "feat: add MyProvider for XYZ integration"

# 3. 推送并创建 PR
git push origin feature/my-extension
```

---

## 扩展开发清单

- [ ] 代码实现完成
- [ ] 单元测试通过
- [ ] 集成测试通过
- [ ] 代码格式化 (`dart format .`)
- [ ] 静态分析通过 (`dart analyze`)
- [ ] 文档更新
- [ ] 配置示例添加
- [ ] CHANGELOG 更新

---

## 相关文档

- [项目结构](../10_PROJECT_STRUCTURE.md)
- [核心模块](../20_CORE_MODULE.md)
- [LLM 集成](../extensions/32_LLM_INTEGRATIONS.md)
- [MCP 服务器](../extensions/35_MCP_SERVERS.md)
