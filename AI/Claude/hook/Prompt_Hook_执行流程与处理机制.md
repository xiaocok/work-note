# Prompt Hook 执行流程与处理机制

## 1. Prompt Hook 定义与结构

### 核心定义
Prompt Hook 是 Claude Code 中的一种钩子类型，使用大模型（LLM）来评估条件并返回评估结果。

### 配置结构
```json
{
  "type": "prompt",
  "prompt": "用于 LLM 评估的提示词。使用 $ARGUMENTS 占位符获取钩子输入 JSON。",
  "if": "可选的权限规则语法，用于过滤钩子运行时机",
  "timeout": "可选的超时时间（秒）",
  "model": "可选的使用模型（例如 \"claude-sonnet-4-6\"）",
  "statusMessage": "可选的显示状态消息",
  "once": "可选的布尔值，设置为 true 时钩子仅执行一次后移除"
}
```

## 2. 大模型处理流程

### 1. 提示词构建
- Claude 代码将用户配置的 `prompt` 与实际输入（替换 `$ARGUMENTS` 占位符）组合
- 添加系统提示词，明确要求返回 JSON 格式

### 2. 系统提示词要求
```
你正在 Claude Code 中评估一个钩子。

你的响应必须是一个符合以下模式之一的 JSON 对象：
1. 如果条件满足，返回：{"ok": true}
2. 如果条件不满足，返回：{"ok": false, "reason": "不满足条件的原因"}
```

### 3. 大模型执行
- 大模型接收构建好的提示词
- 分析输入内容，评估条件
- 生成 JSON 格式的响应

### 4. 大模型返回格式
- **条件满足**：`{"ok": true}`
- **条件不满足**：`{"ok": false, "reason": "具体原因"}`

## 3. Claude 处理流程

### 1. 响应解析
- 提取大模型返回的文本内容
- 尝试解析为 JSON 对象
- 验证是否符合预期的 Schema

### 2. 结果处理
- **JSON 解析失败**：返回 `non_blocking_error`，操作继续但记录错误
- **Schema 验证失败**：返回 `non_blocking_error`，操作继续但记录错误
- **条件不满足**：返回 `blocking`，阻止操作继续执行
- **条件满足**：返回 `success`，允许操作继续执行

### 3. 关键处理代码
```typescript
// 解析 JSON
const json = safeParseJSON(fullResponse)
if (!json) {
  return {
    hook,
    outcome: 'non_blocking_error',
    message: createAttachmentMessage({
      type: 'hook_non_blocking_error',
      stderr: 'JSON validation failed',
      stdout: fullResponse,
      exitCode: 1,
    }),
  }
}

// 验证 schema
const parsed = hookResponseSchema().safeParse(json)
if (!parsed.success) {
  return {
    hook,
    outcome: 'non_blocking_error',
    message: createAttachmentMessage({
      type: 'hook_non_blocking_error',
      stderr: `Schema validation failed: ${parsed.error.message}`,
      stdout: fullResponse,
      exitCode: 1,
    }),
  }
}

// 处理评估结果
if (!parsed.data.ok) {
  return {
    hook,
    outcome: 'blocking',
    blockingError: {
      blockingError: `Prompt hook condition was not met: ${parsed.data.reason}`,
      command: hook.prompt,
    },
    preventContinuation: true,
    stopReason: parsed.data.reason,
  }
}

return {
  hook,
  outcome: 'success',
  message: createAttachmentMessage({
    type: 'hook_success',
    content: '',
  }),
}
```

## 4. 完整执行流程

### 触发阶段
1. **事件触发**：特定事件发生（如 `UserPromptSubmit`、`PostToolUse`、`Stop` 等）
2. **Hook 筛选**：根据 `if` 条件筛选符合条件的 Prompt Hook
3. **输入准备**：构建 hook 输入数据

### 执行阶段
1. **提示词处理**：替换 `$ARGUMENTS` 占位符
2. **模型调用**：使用指定模型（默认为小型快速模型）
3. **响应处理**：解析并验证大模型返回的 JSON
4. **结果生成**：根据验证结果生成 hook 执行结果

### 后续阶段
1. **结果汇总**：聚合所有 hook 的执行结果
2. **主流程处理**：根据 hook 执行结果决定是否继续主流程
3. **用户反馈**：向用户显示相关信息（如阻塞原因、推荐内容等）

## 5. 实际应用场景

### 1. 内容安全检查
```json
{
  "type": "prompt",
  "prompt": "检查用户的提示词是否包含有害内容。如果包含，返回 {\"ok\": false, \"reason\": \"检测到有害内容\"}。否则，返回 {\"ok\": true}。\n\n提示词：$ARGUMENTS"
}
```

### 2. 任务完成推荐
```json
{
  "type": "prompt",
  "prompt": "分析对话历史并识别刚刚完成的任务。然后推荐 2-3 个用户最可能想要执行的下一步任务。将响应格式化为包含推荐的 JSON 对象。\n\n对话历史：$ARGUMENTS"
}
```

### 3. 操作验证
```json
{
  "type": "prompt",
  "prompt": "评估执行此操作是否安全。如果安全，返回 {\"ok\": true}。否则，返回 {\"ok\": false, \"reason\": \"安全隐患\"}。\n\n操作：$ARGUMENTS"
}
```

## 6. hooks.json 注册配置

### 完整的 hooks.json 配置示例

```json
{
  "hooks": {
    "UserPromptSubmit": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "检查用户的提示词是否包含有害内容。如果包含，返回 {\"ok\": false, \"reason\": \"检测到有害内容\"}。否则，返回 {\"ok\": true}。\n\n提示词：$ARGUMENTS",
            "statusMessage": "检查内容安全",
            "model": "claude-sonnet-4-6"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "分析对话历史并识别刚刚完成的任务。然后推荐 2-3 个用户最可能想要执行的下一步任务。将响应格式化为包含推荐的 JSON 对象。\n\n对话历史：$ARGUMENTS",
            "statusMessage": "生成任务推荐",
            "model": "claude-sonnet-4-6"
          }
        ]
      }
    ],
    "Stop": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "prompt",
            "prompt": "评估是否可以安全停止当前操作。如果可以，返回 {\"ok\": true}。否则，返回 {\"ok\": false, \"reason\": \"不允许停止的原因\"}。\n\n当前上下文：$ARGUMENTS",
            "statusMessage": "检查是否可以安全停止",
            "model": "claude-sonnet-4-6"
          }
        ]
      }
    ]
  }
}
```

### 注册说明

1. **事件类型**：
   - `UserPromptSubmit`：用户提交提示词时触发
   - `PostToolUse`：工具执行完成后触发
   - `Stop`：用户执行停止操作时触发
   - 其他支持的事件：`PreToolUse`、`SessionStart` 等

2. **Matcher 配置**：
   - 空字符串 `""`：匹配所有情况
   - 具体模式：如 `"Bash(git *)"` 只匹配 git 相关的 Bash 命令

3. **Hook 配置**：
   - `type`：固定为 `"prompt"`
   - `prompt`：大模型评估的提示词
   - `statusMessage`：执行时显示的状态消息
   - `model`：使用的模型（可选）
   - `timeout`：超时时间（可选）
   - `once`：是否只执行一次（可选）

4. **Stop Hook 注册**：
   - 在 `"Stop"` 事件下配置 Prompt Hook
   - 用于评估停止操作的安全性
   - 可以根据当前操作的上下文决定是否允许停止

### 配置位置

hooks.json 文件通常位于以下位置：
- **Windows**：`%APPDATA%\claude-code\settings\hooks.json`
- **macOS/Linux**：`~/.claude-code/settings/hooks.json`

如果文件不存在，可以创建一个新文件并按照上述格式配置。

## 7. $ARGUMENTS 变量详解

### 什么是 $ARGUMENTS

`$ARGUMENTS` 是 Prompt Hook 中的一个特殊占位符，用于在提示词中插入与当前事件相关的上下文数据。当 Hook 执行时，Claude 会将 `$ARGUMENTS` 替换为包含事件相关信息的 JSON 字符串。

### 不同事件中的 $ARGUMENTS 内容

#### 1. UserPromptSubmit 事件

当用户提交提示词时，`$ARGUMENTS` 包含用户提交的提示词内容：

```json
{
  "hook_event_name": "UserPromptSubmit",
  "prompt": "用户提交的提示词内容",
  "permission_mode": "default"
}
```

**使用示例**：
```json
{
  "type": "prompt",
  "prompt": "检查用户的提示词是否包含有害内容：$ARGUMENTS"
}
```

#### 2. PostToolUse 事件

当工具执行完成后，`$ARGUMENTS` 包含工具执行的详细信息：

```json
{
  "hook_event_name": "PostToolUse",
  "tool_name": "Bash",
  "tool_input": "ls -la",
  "tool_output": "total 16\ndrwxr-xr-x  2 user user 4096 Apr 26 10:00 .\ndrwxr-xr-x 10 user user 4096 Apr 26 09:00 ..\n-rw-r--r--  1 user user  123 Apr 26 10:00 file.txt",
  "exit_code": 0,
  "permission_mode": "default"
}
```

**使用示例**：
```json
{
  "type": "prompt",
  "prompt": "分析工具执行结果并生成摘要：$ARGUMENTS"
}
```

#### 3. Stop 事件

当用户执行停止操作时，`$ARGUMENTS` 包含停止操作的上下文信息：

```json
{
  "hook_event_name": "Stop",
  "permission_mode": "default"
}
```

**使用示例**：
```json
{
  "type": "prompt",
  "prompt": "评估是否可以安全停止当前操作：$ARGUMENTS"
}
```

#### 4. PreToolUse 事件

当工具即将执行时，`$ARGUMENTS` 包含工具执行的预信息：

```json
{
  "hook_event_name": "PreToolUse",
  "tool_name": "Bash",
  "tool_input": "rm -rf /",
  "permission_mode": "default"
}
```

**使用示例**：
```json
{
  "type": "prompt",
  "prompt": "评估工具执行的安全性：$ARGUMENTS"
}
```

### 如何使用 $ARGUMENTS

1. **直接引用**：在提示词中直接使用 `$ARGUMENTS` 占位符，Claude 会在执行时替换为实际的 JSON 数据。

2. **在提示词中解析**：由于 `$ARGUMENTS` 是 JSON 格式，你可以在提示词中指导大模型如何解析和使用这些数据。

   **示例**：
   ```json
   {
     "type": "prompt",
     "prompt": "请分析以下 JSON 数据中的工具执行结果，并判断操作是否成功：\n$ARGUMENTS\n\n如果操作成功，返回 {\"ok\": true}，否则返回 {\"ok\": false, \"reason\": \"失败原因\"}"
   }
   ```

3. **提取特定字段**：你可以指导大模型从 JSON 中提取特定字段进行分析。

   **示例**：
   ```json
   {
     "type": "prompt",
     "prompt": "请分析用户提交的提示词，判断是否包含敏感信息：\n\n用户提示词：$ARGUMENTS\n\n如果包含敏感信息，返回 {\"ok\": false, \"reason\": \"检测到敏感信息\"}，否则返回 {\"ok\": true}"
   }
   ```

### 最佳实践

1. **明确说明**：在提示词中明确告诉大模型如何处理 `$ARGUMENTS` 中的数据
2. **结构化输出**：要求大模型返回结构化的 JSON 格式
3. **错误处理**：考虑 `$ARGUMENTS` 可能为空或格式异常的情况
4. **安全性**：不要在提示词中硬编码敏感信息，使用 `$ARGUMENTS` 传递

### 注意事项

- `$ARGUMENTS` 会被替换为 JSON 字符串，因此在提示词中使用时要考虑格式问题
- 不同事件的 `$ARGUMENTS` 结构不同，编写提示词时要针对特定事件的结构
- 对于复杂的 `$ARGUMENTS` 数据，可能需要在提示词中提供更详细的解析指导

## 6. 技术要点

### 1. 模型选择
- 默认为小型快速模型（通常是 Claude Haiku）
- 可通过 `model` 参数指定其他模型，如 "claude-sonnet-4-6"
- 平衡执行速度和评估质量

### 2. 错误处理
- JSON 解析失败 → `non_blocking_error`
- Schema 验证失败 → `non_blocking_error`
- 执行超时 → `cancelled`
- 其他错误 → `non_blocking_error`

### 3. 性能优化
- 使用小型快速模型减少执行时间
- 设置合理的超时时间避免阻塞
- 利用 `if` 条件减少不必要的 hook 执行

## 7. 与其他 Hook 类型的比较

| Hook 类型 | 执行方式 | 核心特点 | 适用场景 |
|----------|---------|---------|--------|
| **Prompt** | 使用 LLM 评估 | 基于自然语言逻辑判断 | 复杂条件评估、语义理解 |
| **Command** | 执行 shell 命令 | 基于脚本执行 | 系统操作、文件处理 |
| **HTTP** | 发送 HTTP 请求 | 基于 API 交互 | 外部服务集成 |
| **Agent** | 执行代理验证 | 基于代理逻辑 | 复杂验证、多步骤检查 |

## 8. 最佳实践

1. **明确的提示词**：确保提示词清晰明确，包含所有必要的信息
2. **结构化输出**：严格遵循 JSON 格式要求，确保大模型返回正确的结构
3. **合理的模型选择**：根据任务复杂度选择适当的模型
4. **错误处理**：考虑各种边界情况，确保 hook 执行的稳定性
5. **性能考虑**：优化提示词长度和复杂度，减少执行时间

## 9. Prompt 类型的 Stop Hook 流程

### 1. 触发时机
Stop Hook 在用户执行停止操作时触发，例如：
- 在 REPL 中按下 Ctrl+C
- 使用 `/stop` 命令
- 其他停止当前操作的方式

### 2. 执行流程

#### 步骤 1：触发 Stop Hook
当用户执行停止操作时，系统会调用 `executeStopHooks` 函数（如果存在）。

#### 步骤 2：执行 Prompt 类型的 Stop Hook
与其他 Prompt Hook 类似，Prompt 类型的 Stop Hook 会：
1. 构建提示词（替换 `$ARGUMENTS` 占位符）
2. 调用大模型进行评估
3. 处理大模型返回的结果

#### 步骤 3：大模型评估与返回
大模型会根据提示词返回 JSON 格式的评估结果：
- `{"ok": true}` → 条件满足，允许停止操作
- `{"ok": false, "reason": "具体原因"}` → 条件不满足，阻止停止操作

#### 步骤 4：Claude 处理结果
Claude 代码会根据大模型的返回执行以下操作：

##### 情况 1：大模型返回 `{"ok": true}`
- `outcome: 'success'`
- 继续执行停止操作

##### 情况 2：大模型返回 `{"ok": false, "reason": "..."}`
- `outcome: 'blocking'`
- 阻止停止操作，显示错误消息

##### 情况 3：大模型返回格式错误
- `outcome: 'non_blocking_error'`
- 继续执行停止操作，但记录错误

### 3. 关键代码分析

#### Stop Hook 的执行
在 `src/utils/hooks.ts` 中，`executeStopHooks` 函数负责执行 Stop 事件的 hooks：

```typescript
export async function* executeStopHooks(
  toolUseContext: ToolUseContext,
  requestPrompt?: (
    sourceName: string,
    toolInputSummary?: string | null,
  ) => (request: PromptRequest) => Promise<PromptResponse>,
): AsyncGenerator<AggregatedHookResult> {
  const appState = toolUseContext.getAppState()
  const sessionId = toolUseContext.agentId ?? getSessionId()
  
  if (!hasHookForEvent('Stop', appState, sessionId)) {
    return
  }

  const hookInput: StopHookInput = {
    ...createBaseHookInput(appState.toolPermissionContext.mode),
    hook_event_name: 'Stop',
  }

  yield* executeHooks({
    hookInput,
    toolUseID: randomUUID(),
    signal: toolUseContext.abortController.signal,
    timeoutMs: TOOL_HOOK_EXECUTION_TIMEOUT_MS,
    toolUseContext,
    requestPrompt,
  })
}
```

### 4. 大模型调用的必要性

**是的，Prompt 类型的 Stop Hook 会调用大模型**，即使是停止操作。这是因为：
1. Stop Hook 的目的是评估是否应该允许停止操作
2. 这种评估可能需要复杂的逻辑或上下文理解，适合由大模型处理
3. 与其他 Prompt Hook 一样，Stop Hook 依赖大模型的评估结果来决定是否阻止操作

### 5. 执行示例

#### 示例 1：允许停止

**配置的 Prompt Hook**：
```json
{
  "type": "prompt",
  "prompt": "评估是否可以安全停止当前操作。如果可以，返回 {\"ok\": true}。否则，返回 {\"ok\": false, \"reason\": \"不允许停止的原因\"}。\n\n当前上下文：$ARGUMENTS",
  "statusMessage": "检查是否可以安全停止"
}
```

**执行流程**：
1. 用户执行停止操作
2. 系统触发 Stop Hook
3. 大模型评估后返回 `{"ok": true}`
4. Claude 处理为 `outcome: 'success'`
5. 停止操作继续执行

#### 示例 2：阻止停止

**配置的 Prompt Hook**：
```json
{
  "type": "prompt",
  "prompt": "检查当前操作是否处于不应被中断的关键状态。如果是，返回 {\"ok\": false, \"reason\": \"操作处于关键状态\"}。否则，返回 {\"ok\": true}。\n\n当前上下文：$ARGUMENTS",
  "statusMessage": "检查操作状态"
}
```

**执行流程**：
1. 用户执行停止操作
2. 系统触发 Stop Hook
3. 大模型评估后返回 `{"ok": false, "reason": "操作处于关键状态"}`
4. Claude 处理为 `outcome: 'blocking'`
5. 停止操作被阻止，显示错误消息

### 6. 后续流程

#### 停止操作成功
如果 Stop Hook 执行成功（大模型返回 `{"ok": true}`），后续流程：
1. 停止当前正在执行的操作
2. 清理相关资源
3. 返回停止成功的消息给用户

#### 停止操作被阻止
如果 Stop Hook 执行失败（大模型返回 `{"ok": false, "reason": "..."}`），后续流程：
1. 显示阻止消息给用户
2. 继续当前操作
3. 不执行停止操作

## 10. 总结

Prompt Hook 是 Claude Code 中一种强大的机制，利用大模型的自然语言理解能力来评估复杂条件。它的工作流程是：

1. **触发事件** → 2. **构建提示词** → 3. **大模型评估** → 4. **Claude 处理响应** → 5. **决定后续操作**

这种设计允许用户通过配置 Prompt Hook 来自定义 Claude Code 的行为，实现从内容安全检查到任务推荐的各种功能，为系统提供了灵活的扩展能力。

Prompt Hook 的核心价值在于将大模型的智能判断与系统的执行逻辑相结合，既利用了大模型的语义理解能力，又保持了系统的稳定性和可预测性。

特别是 Prompt 类型的 Stop Hook，它允许用户在执行停止操作时进行智能评估，确保只有在适当的情况下才允许停止，从而保护正在执行的关键操作。