# AskUserQuestion 工具实现详解

## 概述

本文档详细介绍了如何使用 `@anthropic-ai/claude-agent-sdk` 实现一个自定义的 `AskUser` 工具，该工具可以向用户提问并等待用户选择答案。

---

## 核心概念

### 1. Claude Agent SDK

Claude Agent SDK 是 Anthropic 提供的 TypeScript/JavaScript SDK，用于构建 AI Agent 应用。其核心功能：

- **query()** - 向 Claude 发送提示并获取流式响应
- **tool()** - 创建自定义 MCP 工具
- **createSdkMcpServer()** - 将工具包装成 MCP 服务器配置

### 2. MCP (Model Context Protocol)

MCP 是一个标准协议，允许 AI 模型调用外部工具和服务。自定义工具通过 MCP 注册到 Agent 系统中。

### 3. Elicitation (征询机制)

当 Agent 需要用户确认或输入时，会触发 elicitation 机制。

**组件职责：**

| 组件 | 负责方 | 说明 |
|------|--------|------|
| `onElicitation` 回调 | 你（开发者） | 实现 UI 显示和用户交互 |
| 返回结果给 MCP | 你（开发者） | 决定返回什么内容 |
| Claude 接收结果 | SDK 自动 | 工具结果传给模型 |

简单说：`onElicitation` 就是你写代码显示对话框/等待用户输入的地方。

**两种实现方式：**

1. **直接读取（当前项目）** - 工具内直接用 `readline` 读取
2. **onElicitation 回调（标准方式）** - 通过 `elicitInput()` 触发 SDK 回调

详见「处理器函数逻辑」章节。

### 4. Promise 与异步处理

**Promise 是 JavaScript 中处理异步操作的核心机制。**

在本项目中，工具需要等待用户在终端输入，这个过程是异步的：

```javascript
return new Promise((resolve) => {
  rl.question('Enter: ', (answer) => {
    // 用户输入完成后调用
    resolve(answer);
  });
});
```

**为什么需要 Promise？**

JavaScript 是单线程、非阻塞的。如果用同步方式等待用户输入，会卡住整个程序，破坏 Agent 的流式响应体验。

Promise 的作用：

| 传统同步方式 | Promise 异步方式 |
|-------------|-----------------|
| 程序卡住等待 | 程序继续运行 |
| 阻塞整个进程 | 不影响其他操作 |
| 用户输入后继续 | 用户输入后通过 resolve 回调继续 |

**执行流程：**

```
Agent 调用工具
    ↓
显示问题 "What is your favorite..."
    ↓
返回 Promise（暂停当前执行，但不阻塞程序）
    ↓
用户输入 "1"
    ↓
Promise resolve，通知完成
    ↓
工具返回结果给 Agent
    ↓
Agent 处理结果并回复
```

**简单类比：**

把 Promise 想象成一个**外卖订单**：
- 下单后（返回 Promise）你可以做其他事
- 饭做好了（用户输入），订单完成
- 吃饭（继续执行后续逻辑）

---

## 完整代码

```javascript
#!/usr/bin/env node

import { query, tool, createSdkMcpServer } from '@anthropic-ai/claude-agent-sdk';
import { z } from 'zod';

async function main() {
  console.log('Starting SDK with custom AskUser tool...\n');

  // ============ 1. 创建自定义工具 ============
  const askUserTool = tool(
    'AskUser',                              // 工具名称
    'Ask the user a multiple-choice question and wait for their response',  // 描述
    z.object({                             // 输入参数 schema
      question: z.string().describe('The question to ask'),
      options: z.array(
        z.object({
          label: z.string().describe('Option label'),
          description: z.string().optional().describe('Option description'),
        })
      ).describe('Available options'),
    }),
    async (args) => {                      // 处理器函数
      const { question, options } = args;
      
      console.log('\n========================================');
      console.log('🤖 Custom AskUser Tool');
      console.log('========================================');
      console.log(question);
      console.log('Options:');
      options.forEach((opt, i) => {
        console.log(`  [${i + 1}] ${opt.label}${opt.description ? ` - ${opt.description}` : ''}`);
      });
      console.log('========================================\n');

      // 读取用户输入
      const readline = await import('readline');
      const rl = readline.createInterface({
        input: process.stdin,
        output: process.stdout,
      });

      return new Promise((resolve) => {
        rl.question('Enter your choice (by number): ', (answer) => {
          rl.close();
          const index = parseInt(answer, 10) - 1;
          if (index >= 0 && index < options.length) {
            const selected = options[index];
            console.log(`Selected: ${selected.label}\n`);
            resolve({
              content: [{
                type: 'text',
                text: JSON.stringify({ selectedOption: selected.label }, null, 2),
              }],
            });
          } else {
            console.log('Invalid choice.\n');
            resolve({
              content: [{ type: 'text', text: 'No option selected.' }],
              isError: true,
            });
          }
        });
      });
    },
    { alwaysLoad: true }                   // 始终加载，不延迟
  );

  // ============ 2. 创建 MCP 服务器配置 ============
  const mcpServer = createSdkMcpServer({
    name: 'custom-ask-tool',
    version: '1.0.0',
    tools: [askUserTool],
    alwaysLoad: true,
  });

  // ============ 3. 调用 query ============
  const response = query({
    prompt: 'Use the AskUser tool to ask me: "What is your favorite programming language?" with these exact options:\n1. JavaScript\n2. Python\n3. TypeScript\n4. Rust\n\nAfter getting the answer, tell me what I selected.',
    options: {
      mcpServers: {
        'custom-ask': mcpServer,
      },
      permissionMode: 'bypassPermissions',
      allowDangerouslySkipPermissions: true,
    },
  });

  // ============ 4. 处理响应流 ============
  for await (const message of response) {
    if (message.type === 'assistant') {
      console.log('\n========================================');
      console.log('💬 Assistant');
      console.log('========================================');
      const assistantMsg = message.message;
      if (assistantMsg?.content) {
        for (const block of assistantMsg.content) {
          if (block.type === 'text') {
            console.log(block.text);
          } else if (block.type === 'tool_use') {
            console.log(`\n[Using tool: ${block.name}]`);
          }
        }
      }
      console.log('========================================\n');
    } else if (message.type === 'result') {
      console.log('\n========================================');
      console.log('✅ Result');
      console.log('========================================');
      const result = message.result;
      if (result) {
        console.log(JSON.stringify(result, null, 2));
      }
      if (message.errors) {
        console.log('Errors:', message.errors);
      }
      console.log('========================================\n');
      break;
    }
  }
}

main().catch(console.error);
```

---

## 代码详解

### 1. 导入依赖

```javascript
import { query, tool, createSdkMcpServer } from '@anthropic-ai/claude-agent-sdk';
import { z } from 'zod';
```

| 导入 | 说明 |
|------|------|
| `query` | SDK 核心函数，向 Claude 发送提示并获取响应 |
| `tool` | 创建 MCP 工具的工厂函数 |
| `createSdkMcpServer` | 将工具包装成 MCP 服务器配置 |
| `zod` | 参数类型校验库 |

---

### 2. 创建自定义工具

```javascript
const askUserTool = tool(
  'AskUser',                               // 工具名称
  '描述信息',                               // 工具描述（Agent 会看到）
  z.object({ ... }),                       // 输入参数 schema
  async (args) => { ... },                 // 处理器函数
  { alwaysLoad: true }                     // 配置选项
);
```

**参数说明：**

| 参数 | 类型 | 说明 |
|------|------|------|
| 第一个参数 | string | 工具名称，Agent 通过此名称调用工具 |
| 第二个参数 | string | 工具描述，告诉 Agent 工具的用途 |
| 第三个参数 | Zod Schema | 定义输入参数的结构和类型 |
| 第四个参数 | function | 工具实际执行的处理器 |
| 第五个参数 | object | 额外配置，如 `alwaysLoad` |

**Zod Schema 详解：**

Zod 是一个 TypeScript-first 的模式声明和验证库，在 MCP 工具中用于定义参数结构。

**为什么需要 Zod Schema？**

```
Schema 定义 → SDK → 告诉 Claude 这个工具需要什么参数
    ↓
Claude 理解参数要求
    ↓
Claude 生成符合 schema 的参数
    ↓
SDK 验证参数 → 调用工具处理器
```

**常用 Zod 类型：**

| 方法 | 说明 | 示例 |
|------|------|------|
| `z.string()` | 字符串类型 | `"hello"` |
| `z.number()` | 数字类型 | `42`, `3.14` |
| `z.boolean()` | 布尔类型 | `true`, `false` |
| `z.array(schema)` | 数组类型 | `[1, 2, 3]` |
| `z.object({})` | 对象类型 | `{ name: "..." }` |
| `z.enum([...])` | 枚举类型 | `"A"`, `"B"`, `"C"` |

**修饰符方法：**

| 方法 | 说明 | 示例 |
|------|------|------|
| `.describe('...')` | 添加描述，影响 Claude 理解 | 告诉 Agent 参数用途 |
| `.optional()` | 可选字段，可不传 | `name: z.string().optional()` |
| `.nullable()` | 可为 null | `name: z.string().nullable()` |
| `.default(value)` | 默认值 | `page: z.number().default(1)` |

**参数结构详解：**

```javascript
z.object({
  question: z.string().describe('The question to ask'),
  options: z.array(
    z.object({
      label: z.string().describe('Option label'),
      description: z.string().optional().describe('Option description'),
    })
  ).describe('Available options'),
})
```

**结构图：**

```
z.object({ ... })
    │
    ├── question: z.string()      ← 字符串类型
    │            └── .describe()  ← 描述：告诉 Claude 这个参数是什么
    │
    └── options: z.array(...)    ← 数组类型
                └── z.object({})  ← 数组元素是对象
                      │
                      ├── label: z.string()    ← 字符串类型
                      │            └── .describe()
                      │
                      └── description: z.string().optional()
                                              ├── .optional()  ← 可选字段
                                              └── .describe()
```

**.describe() 的作用：**

`.describe()` 不仅仅是注释，它会被传递给 Claude：

```javascript
question: z.string().describe('The question to ask')
//                          ↑
//                    Claude 会看到这个描述
```

| 写法 | 效果 |
|------|------|
| 有 describe | Claude 理解清晰，知道这是"要问的问题" |
| 无 describe | Claude 可能误解参数用途 |

**.optional() 的作用：**

标记字段为可选，用户可以不提供：

```javascript
description: z.string().optional()
// 表示：description 可以不传
```

**Claude 生成的参数示例：**

```javascript
{
  question: "What is your favorite language?",
  options: [
    { label: "JavaScript", description: "Web language" },
    { label: "Python", description: "AI language" },
    { label: "TypeScript" },  // 可选字段不提供
    { label: "Rust" }
  ]
}
```

---

### 3. 处理器函数逻辑

```javascript
async (args) => {
  const { question, options } = args;
  
  // 1. 显示问题和选项到终端
  console.log(question);
  options.forEach(...);
  
  // 2. 使用 readline 读取用户输入
  const readline = await import('readline');
  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
  });
  
  // 3. 返回 Promise 等待用户响应
  return new Promise((resolve) => {
    rl.question('Enter your choice (by number): ', (answer) => {
      // 验证输入并返回结果
      resolve({ content: [...] });
    });
  });
}
```

**关键点：**

1. **异步处理** - 使用 `async` 函数处理异步操作
2. **readline 模块** - Node.js 内置模块，用于读取终端输入
3. **Promise** - 返回 Promise 让工具可以等待用户输入（详见下一节）
4. **MCP 响应格式** - 返回 `{ content: [...] }` 是标准格式

**Promise 在工具中的完整使用：**

```javascript
async (args) => {
  const { question, options } = args;
  
  // 1. 显示问题（同步，立即执行）
  console.log(question);
  
  // 2. 创建 readline 接口（同步）
  const readline = await import('readline');
  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
  });
  
  // 3. 返回 Promise（关键！）
  // 工具执行到这里暂停，但程序继续运行
  // 当用户输入完成后，Promise 会被 resolve
  return new Promise((resolve) => {
    rl.question('Enter your choice (by number): ', (answer) => {
      rl.close();
      
      // 4. 用户输入后的回调
      const index = parseInt(answer, 10) - 1;
      if (index >= 0 && index < options.length) {
        resolve({
          content: [{ type: 'text', text: JSON.stringify(...) }],
        });
      } else {
        resolve({ content: [{ type: 'text', text: 'Invalid' }], isError: true });
      }
    });
  });
  
  // 5. 这里不会执行，直到 Promise 被 resolve
}
```

**MCP 工具响应格式：**

```javascript
{
  content: [{ type: 'text', text: '响应内容' }],
  isError: false  // 可选，标记是否出错
}
```

---

### 3.1 两种交互方式对比

**方式一：工具内直接读取（当前实现）**

工具处理器内部直接使用 `readline` 读取用户输入：

```javascript
tool('AskUser', ..., async (args) => {
  const rl = readline.createInterface({ input: process.stdin, output: process.stdout });
  return new Promise((resolve) => {
    rl.question('Enter:', (answer) => {
      resolve({ content: [{ type: 'text', text: answer }] });
    });
  });
});
```

适用场景：简单的 CLI 工具、调试环境

**方式二：onElicitation 回调（标准 MCP 方式）**

工具通过 `elicitInput()` 触发 SDK 的 `onElicitation` 回调：

```javascript
tool('AskUser', ..., async (args, extra) => {
  const result = await extra.server.elicitInput({
    message: args.question,
    requestedSchema: { type: 'object', ... }
  });
  return { content: [{ type: 'text', text: JSON.stringify(result) }] };
});

// 在 query 中配置
query({
  options: {
    onElicitation: async (request) => {
      console.log(request.message);
      const answer = await rl.question('Your choice: ');
      return { action: 'accept', content: { selectedOption: answer } };
    }
  }
});
```

适用场景：外部 MCP 服务器、需要复杂 UI（如 Web 对话框、Electron 窗口）

**对比表：**

| 方式 | 实现复杂度 | 灵活性 | 适用场景 |
|------|----------|--------|----------|
| 直接读取 | 低 | 低 | CLI 工具、快速调试 |
| onElicitation | 高 | 高 | 外部 MCP、需要自定义 UI |

**onElicitation 工作流程：**

```
Claude 调用工具
    ↓
MCP 工具 handler 调用 elicitInput()
    ↓
SDK 触发 onElicitation 回调
    ↓
你的代码显示 UI，等待用户响应
    ↓
返回结果给 MCP handler
    ↓
MCP handler 返回结果给 Claude
```

**onElicitation 回调参数：**

```javascript
onElicitation: async (request) => {
  request.message      // 要显示的问题
  request.mode        // 模式（form/...）
  request.requestedSchema  // 请求的数据结构
}
```

**onElicitation 返回格式：**

```javascript
return {
  action: 'accept',           // 'accept' | 'decline'
  content: { selectedOption: 'xxx' }
};
```

---

### 4. 创建 MCP 服务器配置

```javascript
const mcpServer = createSdkMcpServer({
  name: 'custom-ask-tool',    // MCP 服务器名称
  version: '1.0.0',            // 版本号
  tools: [askUserTool],        // 注册的工具列表
  alwaysLoad: true,            // 始终加载
});
```

**配置说明：**

| 配置 | 说明 |
|------|------|
| `name` | MCP 服务器的唯一标识 |
| `version` | 版本号 |
| `tools` | 数组，可注册多个工具 |
| `alwaysLoad` | true = 工具始终可用；false = 需要时再加载 |

---

### 5. 调用 query 函数

```javascript
const response = query({
  prompt: '使用 AskUser 工具问用户...',
  options: {
    mcpServers: {
      'custom-ask': mcpServer,
    },
    permissionMode: 'bypassPermissions',
    allowDangerouslySkipPermissions: true,
  },
});
```

**关键配置：**

| 配置 | 说明 |
|------|------|
| `prompt` | 给 Agent 的指令 |
| `mcpServers` | 注册自定义 MCP 服务器 |
| `permissionMode` | 权限模式，控制工具调用的授权行为 |
| `allowDangerouslySkipPermissions` | 允许跳过权限确认（配合 bypassPermissions 使用） |

**permissionMode 选项详解：**

| 模式 | 说明 | 使用场景 |
|------|------|----------|
| `'default'` | 标准行为，危险操作会提示授权 | 默认值，CLI 交互环境 |
| `'acceptEdits'` | 自动接受文件编辑操作，其他仍需授权 | 半自动化脚本 |
| `'bypassPermissions'` | 跳过所有权限检查，工具直接执行 | 开发调试、受信任环境 |
| `'plan'` | 仅计划模式，不执行任何工具操作 | 查看 Agent 会执行哪些操作 |
| `'dontAsk'` | 不询问，直接拒绝未预批准的操作 | 严格安全环境 |

**为什么这个配置很关键？**

Claude Agent SDK 默认会检查工具调用的权限。当你使用 MCP 自定义工具时：

```
不加 permissionMode: 'bypassPermissions' 时的流程：

Agent 调用 AskUser 工具
    ↓
SDK 检测到工具调用请求
    ↓
检查权限 → 需要用户确认
    ↓
没有交互式确认机制（无 TTY/终端）
    ↓
跳过工具执行，返回权限提示文本
    ↓
Agent 回复："I need your permission to use the AskUser tool..."
    ↓
工具处理器从未被执行！
```

```
加上 permissionMode: 'bypassPermissions' 后的流程：

Agent 调用 AskUser 工具
    ↓
SDK 检测到工具调用请求
    ↓
检查权限 → bypassPermissions = 直接放行
    ↓
工具处理器执行
    ↓
显示问题，等待用户输入
    ↓
工具正常返回结果
```

**安全警告：**

```javascript
permissionMode: 'bypassPermissions',
allowDangerouslySkipPermissions: true,
```

这两个配置组合会**跳过所有工具权限检查**，在生产环境中使用需谨慎。只有在：
- 开发调试环境
- 完全受信任的工具
- 非交互式自动化脚本

的情况下才建议使用。

---

### 6. 处理响应流

```javascript
for await (const message of response) {
  if (message.type === 'assistant') {
    // 处理助手消息
  } else if (message.type === 'result') {
    // 处理最终结果
    break;
  }
}
```

**消息类型：**

| 类型 | 说明 |
|------|------|
| `assistant` | 助手在思考过程中的中间消息（文本、工具调用等） |
| `result` | 最终结果 |

**assistant 消息内容类型：**

- `type: 'text'` - 文本内容
- `type: 'tool_use'` - 工具调用请求

---

## 完整数据流转详解

### 整体架构图

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          用户代码 (Node.js)                              │
│                                                                         │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐              │
│  │   query()    │───▶│  MCP Server  │───▶│ Tool Handler │              │
│  │              │    │  (create)    │    │ (readline)   │              │
│  └──────────────┘    └──────────────┘    └──────────────┘              │
│         │                   │                   │                      │
│         │         Schema 定义工具参数          显示/输入                 │
└─────────│───────────────────│───────────────────│──────────────────────┘
          │                   │                   │
          │        JSON-RPC 协议通信              │
          │                   │                   ▼
          │                   │         ┌──────────────┐
          │                   │         │  终端 (TTY)  │
          │                   │         │  用户输入    │
          │                   │         └──────────────┘
          │                   │
          ▼                   ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                      Claude Agent SDK                                    │
│                                                                         │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐              │
│  │  Transport   │───▶│  Permission  │───▶│   LLM 调用   │              │
│  │  (通信层)    │    │   检查       │    │              │              │
│  └──────────────┘    └──────────────┘    └──────────────┘              │
│                                                   │                      │
│                           ┌───────────────────────┘                      │
│                           ▼                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐              │
│  │ Claude API   │◀──▶│   工具调用   │◀───│   工具列表   │              │
│  │              │    │   请求       │    │  (Schema)    │              │
│  └──────────────┘    └──────────────┘    └──────────────┘              │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────────────────┐
│                        Claude API (Anthropic)                            │
│                                                                         │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐              │
│  │  System      │    │   Tools      │    │   User      │              │
│  │  Prompt     │    │  (Schema)    │    │  Message    │              │
│  └──────────────┘    └──────────────┘    └──────────────┘              │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

### 第一阶段：初始化（用户代码 → SDK）

**步骤 1：定义 Zod Schema**

```javascript
const askUserTool = tool(
  'AskUser',
  'Ask the user a multiple-choice question',
  z.object({
    question: z.string().describe('The question to ask'),
    options: z.array(
      z.object({
        label: z.string().describe('Option label'),
        description: z.string().optional().describe('Option description'),
      })
    ).describe('Available options'),
  }),
  async handler(args) { /* ... */ },
  { alwaysLoad: true }
);
```

**发生了什么：**
```
Zod Schema → JavaScript 对象
    ↓
{
  name: 'AskUser',
  description: '...',
  inputSchema: {
    type: 'object',
    properties: {
      question: { type: 'string', description: '...' },
      options: {
        type: 'array',
        items: {
          type: 'object',
          properties: {
            label: { type: 'string', description: '...' },
            description: { type: 'string', description: '...' }
          },
          required: ['label']  // description 是 optional，不会在这里
        }
      }
    },
    required: ['question', 'options']
  }
}
```

**步骤 2：创建 MCP Server 配置**

```javascript
const mcpServer = createSdkMcpServer({
  name: 'custom-ask-tool',
  tools: [askUserTool],
  alwaysLoad: true,
});
```

**发生了什么：**
```
SDK 内部创建 MCP Server 实例
    ↓
注册工具到内部工具注册表
    ↓
生成 MCP JSON-RPC 响应（tools/list）
```

**步骤 3：调用 query()**

```javascript
const response = query({
  prompt: 'Use the AskUser tool to ask me...',
  options: {
    mcpServers: { 'custom-ask': mcpServer },
    permissionMode: 'bypassPermissions',
  },
});
```

**发生了什么：**
```
query() 启动 Claude Code 子进程
    ↓
传输 MCP 服务器配置给子进程
    ↓
子进程启动 Claude Code CLI
    ↓
CLI 启动并连接 SDK
```

---

### 第二阶段：API 请求构建（SDK → Claude API）

**步骤 4：SDK 准备 API 请求**

Claude Agent SDK 构建发送给 Claude API 的请求：

```javascript
{
  model: 'claude-sonnet-4-20250514',
  max_tokens: 8192,
  system: [
    "You are Claude Code, an AI assistant...",
    "Available tools: ...",
  ],
  tools: [
    {
      name: "mcp__custom-ask__AskUser",
      description: "Ask the user a multiple-choice question",
      input_schema: {
        type: "object",
        properties: {
          question: {
            type: "string",
            description: "The question to ask"
          },
          options: {
            type: "array",
            items: {
              type: "object",
              properties: {
                label: { type: "string", description: "Option label" },
                description: { type: "string", description: "Option description" }
              },
              required: ["label"]
            }
          }
        },
        required: ["question", "options"]
      }
    }
  ],
  messages: [
    { role: "user", content: "Use the AskUser tool to ask me..." }
  ]
}
```

**关键点：Zod Schema 被转换为 Claude API 的 tools format**

```
Zod: z.string().describe('The question to ask')
    ↓
API: { type: "string", description: "The question to ask" }
```

---

### 第三阶段：模型推理（Claude API）

**步骤 5：Claude 理解工具和请求**

Claude 模型接收请求后：

1. **阅读系统提示**：了解自己是 Claude Code，有工具可用
2. **阅读工具列表**：看到 `mcp__custom-ask__AskUser` 工具
3. **阅读 Schema**：理解工具需要 `question` 和 `options` 参数
4. **分析用户请求**：理解用户想让 AI 问一个问题

```
Claude 思考过程：

用户说："Use the AskUser tool to ask me..."

工具要求：
- question: string (要问的问题)
- options: array (选项列表)
  - label: string (选项标签)
  - description?: string (可选描述)

我应该调用工具，参数：
- question: "What is your favorite programming language?"
- options: [
    { label: "JavaScript" },
    { label: "Python" },
    { label: "TypeScript" },
    { label: "Rust" }
  ]
```

**步骤 6：Claude 生成工具调用**

Claude 返回：

```json
{
  "role": "assistant",
  "content": [
    {
      "type": "tool_use",
      "id": "toolu_xxx",
      "name": "mcp__custom-ask__AskUser",
      "input": {
        "question": "What is your favorite programming language?",
        "options": [
          { "label": "JavaScript" },
          { "label": "Python" },
          { "label": "TypeScript" },
          { "label": "Rust" }
        ]
      }
    }
  ]
}
```

---

### 第四阶段：工具执行（SDK → 工具处理器）

**步骤 7：SDK 接收工具调用**

```
Claude API 返回工具调用请求
    ↓
SDK 解析：name = "mcp__custom-ask__AskUser"
    ↓
SDK 查找工具：从 MCP Server 获取工具定义
    ↓
SDK 提取参数：
{
  question: "What is your favorite programming language?",
  options: [ ... ]
}
```

**步骤 8：权限检查**

```
permissionMode 检查
    ↓
bypassPermissions = true → 直接放行
    ↓
不等待用户确认
```

**步骤 9：调用工具处理器**

```javascript
const result = await toolHandler({
  question: "What is your favorite programming language?",
  options: [
    { label: "JavaScript" },
    { label: "Python" },
    { label: "TypeScript" },
    { label: "Rust" }
  ]
});
```

---

### 第五阶段：用户交互

**步骤 10：工具处理器显示问题**

```javascript
async (args) => {
  const { question, options } = args;
  
  // 显示问题
  console.log(question);
  // 输出: "What is your favorite programming language?"
  
  // 显示选项
  options.forEach((opt, i) => {
    console.log(`  [${i + 1}] ${opt.label}`);
  });
  // 输出:
  //   [1] JavaScript
  //   [2] Python
  //   [3] TypeScript
  //   [4] Rust
  
  // 等待用户输入
  const rl = readline.createInterface({
    input: process.stdin,
    output: process.stdout,
  });
  
  return new Promise((resolve) => {
    rl.question('Enter your choice (by number): ', (answer) => {
      // 用户输入 "1"
      rl.close();
      resolve({ content: [{ type: 'text', text: '...' }] });
    });
  });
}
```

**终端显示：**

```
========================================
🤖 Custom AskUser Tool
========================================
What is your favorite programming language?
Options:
  [1] JavaScript
  [2] Python
  [3] TypeScript
  [4] Rust
========================================

Enter your choice (by number): _
```

**步骤 11：用户输入**

用户在终端输入 `1`：

```
Enter your choice (by number): 1
Selected: JavaScript
```

---

### 第六阶段：结果返回

**步骤 12：工具返回结果**

```javascript
resolve({
  content: [{
    type: 'text',
    text: JSON.stringify({ selectedOption: 'JavaScript' })
  }]
});
```

**步骤 13：SDK 接收结果**

```
工具处理器返回结果
    ↓
SDK 接收：{ content: [{ type: 'text', ... }] }
    ↓
SDK 格式化结果
    ↓
添加到消息历史
```

**步骤 14：继续对话**

SDK 发送新的 API 请求，包含工具结果：

```javascript
{
  messages: [
    { role: "user", content: "Use the AskUser tool..." },
    { role: "assistant", content: [{ type: "tool_use", ... }] },
    {
      role: "user",
      content: [{
        type: "tool_result",
        tool_use_id: "toolu_xxx",
        content: "{\"selectedOption\": \"JavaScript\"}"
      }]
    }
  ]
}
```

**步骤 15：Claude 处理结果并回复**

Claude 看到工具返回了 `"JavaScript"`，生成回复：

```
好的，您选择了 JavaScript！
```

---

### 第七阶段：响应流输出

**步骤 16：SDK 输出响应**

```
for await (const message of response) {
  if (message.type === 'assistant') {
    console.log(message.message.content);
    // 输出: "好的，您选择了 JavaScript！"
  }
}
```

---

### 完整时序图

```
用户代码                    SDK                      Claude API                  用户
   │                         │                          │                        │
   │ query(prompt)           │                          │                        │
   │────────────────────────▶│                          │                        │
   │                         │                          │                        │
   │  ① 定义工具             │                          │                        │
   │  tool() + Zod Schema    │                          │                        │
   │                         │                          │                        │
   │  ② 创建 MCP Server      │                          │                        │
   │  createSdkMcpServer()    │                          │                        │
   │                         │                          │                        │
   │  ③ 启动 Claude Code     │                          │                        │
   │────────────────────────▶│                          │                        │
   │                         │                          │                        │
   │                         │ ④ 构建 API 请求           │                        │
   │                         │ (包含 tools schema)       │                        │
   │                         │─────────────────────────▶│                        │
   │                         │                          │                        │
   │                         │                          │ ⑤ 推理 + 生成工具调用   │
   │                         │                          │───────────────────────▶│
   │                         │                          │                        │
   │                         │ ⑥ 返回 tool_use          │                        │
   │                         │◀─────────────────────────│                        │
   │                         │                          │                        │
   │                         │ ⑦ 解析工具调用            │                        │
   │                         │ ⑧ 权限检查               │                        │
   │                         │ ⑨ 调用工具处理器          │                        │
   │                         │─────────────────────────▶│                        │
   │                         │                          │                        │
   │                         │                          │     ⑩ 显示问题到终端    │
   │                         │                          │◀───────────────────────│
   │                         │                          │                        │
   │                         │                          │     ⑪ 用户输入 "1"      │
   │                         │                          │◀───────────────────────│
   │                         │                          │                        │
   │                         │ ⑫ 返回工具结果            │                        │
   │                         │◀─────────────────────────│                        │
   │                         │                          │                        │
   │                         │ ⑬ 发送工具结果给 API     │                        │
   │                         │─────────────────────────▶│                        │
   │                         │                          │                        │
   │                         │ ⑭ 模型生成回复           │                        │
   │                         │◀─────────────────────────│                        │
   │                         │                          │                        │
   │ ⑮ 输出响应流             │                          │                        │
   │◀────────────────────────│                          │                        │
   │                         │                          │                        │
```

---

### 简化流程总结

```
1️⃣ 定义阶段
   代码定义工具 + Zod Schema
       ↓
2️⃣ 初始化阶段
   query() 启动 Agent
       ↓
3️⃣ 请求构建阶段
   SDK 把 Schema 转成 API tools format
       ↓
4️⃣ 推理阶段
   Claude 理解工具 + 生成参数
       ↓
5️⃣ 执行阶段
   SDK 调用工具处理器 → 显示问题
       ↓
6️⃣ 交互阶段
   用户在终端输入选择
       ↓
7️⃣ 返回阶段
   工具结果 → SDK → API → Claude → 回复
       ↓
8️⃣ 输出阶段
   响应流输出给用户
```

---

## 运行方式

```bash
node ask-user-question.mjs
```

**预期输出：**

```
Starting SDK with custom AskUser tool...

========================================
💬 Assistant
========================================

[Using tool: mcp__custom-ask__AskUser]

========================================
🤖 Custom AskUser Tool
========================================
What is your favorite programming language?
Options:
  [1] JavaScript - JavaScript
  [2] Python - Python
  [3] TypeScript - TypeScript
  [4] Rust - Rust
========================================

Enter your choice (by number): 1
Selected: JavaScript

========================================
💬 Assistant
========================================

You selected JavaScript as your favorite programming language!

========================================
✅ Result
========================================
"You selected JavaScript as your favorite programming language!"
========================================
```

---

## 依赖安装

```bash
npm install @anthropic-ai/claude-agent-sdk zod
```

---

## 注意事项

1. **工具名称冲突** - 避免使用内置工具的名称（如 `AskUserQuestion`），可能与 SDK 内置功能冲突
2. **权限模式（关键！）** - MCP 自定义工具必须设置 `permissionMode: 'bypassPermissions'`，否则工具会被跳过执行
3. **stdin 占用** - SDK 内部可能占用 stdin，需确保工具运行时 stdin 可用
4. **异步处理** - 工具处理器必须是 async 函数，返回 Promise
5. **Promise 必须** - 工具必须返回 Promise 才能支持异步操作，否则无法等待用户输入
