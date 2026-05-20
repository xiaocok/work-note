# Claude Code Hook 完整参考文档

---

## 目录

1. [概述](#1-概述)
2. [通用输入字段](#2-通用输入字段)
3. [通用输出字段](#3-通用输出字段)
4. [Matcher 支持情况](#4-matcher-支持情况)
5. [各 Hook 类型详细说明](#5-各-hook-类型详细说明)
6. [Hook 上下文加载机制](#6-hook-上下文加载机制)
7. [异步 Hook 详解](#7-异步-hook-详解)
8. [执行机制与并发处理](#8-执行机制与并发处理)
9. [脚本读取输入示例](#9-脚本读取输入示例)
10. [注意事项](#10-注意事项)
11. [完整 Hook 事件类型列表](#11-完整-hook-事件类型列表)

---

## 1. 概述

本文档详细描述了 Claude Hook 脚本的输入输出规范，按 Hook 事件类型组织。每个 Hook 类型包含：
- **触发时机**：何时触发该 Hook
- **输入字段**：脚本从 stdin 接收的参数
- **输出字段**：脚本通过 stdout 返回的结果
- **退出码含义**：不同退出码的处理方式
- **使用示例**：实际使用示例

---

## 2. 通用输入字段（所有 Hook 通用）

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `session_id` | string | 是 | 当前会话的唯一标识符 | 跟踪会话、关联数据 | `const sessionId = input.session_id;` |
| `transcript_path` | string | 是 | 会话转录文件路径 | 读取会话历史 | `const history = fs.readFileSync(input.transcript_path);` |
| `cwd` | string | 是 | 当前工作目录 | 文件操作、路径解析 | `process.chdir(input.cwd);` |
| `permission_mode` | string | 否 | 当前权限模式 | 权限判断、访问控制 | `if (input.permission_mode === 'safe')` |
| `agent_id` | string | 否 | 子代理标识符 | 区分子代理和主线程 | `if (input.agent_id)` |
| `agent_type` | string | 否 | 代理类型名称 | 识别代理类型 | `if (input.agent_type === 'code-reviewer')` |

---

## 3. 通用输出字段（所有 Hook 通用）

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否继续执行后续流程 | 控制流程走向 | `"continue": false` 阻止执行 |
| `suppressOutput` | boolean | 否 | 是否隐藏 stdout 从 transcript | 控制输出显示 | `"suppressOutput": true` |
| `stopReason` | string | 否 | 当 `continue: false` 时的停止原因 | 解释为何阻止执行 | `"stopReason": "安全检查失败"` |
| `decision` | string | 否 | 权限决策：`approve` 或 `block` | 批准或拒绝操作 | `"decision": "block"` |
| `systemMessage` | string | 否 | 显示给用户的系统消息 | 通知用户、显示结果 | `"systemMessage": "任务完成"` |
| `reason` | string | 否 | 决策原因 | 解释决策依据 | `"reason": "权限不足"` |
| `additionalContext` | string | 否 | 额外上下文信息 | 向模型传递附加信息 | `"additionalContext": "分析结果..."` |
| `hookSpecificOutput` | object | 否 | 特定事件输出 | 传递事件特定数据 | 见各事件类型 |
| `async` | boolean | 否 | 是否异步执行 | 后台执行长时间任务 | `"async": true` |
| `asyncTimeout` | number | 否 | 异步超时时间（毫秒） | 设置异步执行超时 | `"asyncTimeout": 60000` |

---

## 4. Matcher 支持情况

### 4.1 概述

**不是所有 Hook 都支持 matcher 过滤。** matcher 字段用于过滤 hook 何时触发，但某些事件类型不支持此功能。

### 4.2 不支持 matcher 的 Hook 事件

以下 Hook 事件**不支持 matcher**，总是在每次出现时触发。如果添加 `matcher` 字段，它会被**静默忽略**：

| 事件类型 | 说明 |
|---------|------|
| `Stop` | 每次 Claude 完成响应时触发 |
| `UserPromptSubmit` | 每次用户提交提示时触发 |
| `PostToolBatch` | 每批工具调用完成后触发 |
| `TeammateIdle` | 每次 teammate 空闲时触发 |
| `TaskCreated` | 每次任务创建时触发 |
| `TaskCompleted` | 每次任务完成时触发 |
| `WorktreeCreate` | 每次 worktree 创建时触发 |
| `WorktreeRemove` | 每次 worktree 移除时触发 |
| `CwdChanged` | 每次工作目录改变时触发 |

### 4.3 支持 matcher 的 Hook 事件及匹配内容

| 事件类型 | matcher 匹配的内容 | 示例 matcher 值 |
|---------|-------------------|----------------|
| `PreToolUse` | 工具名称 | `Bash`, `Edit\|Write`, `mcp__.*` |
| `PostToolUse` | 工具名称 | 同上 |
| `PostToolUseFailure` | 工具名称 | 同上 |
| `PermissionRequest` | 工具名称 | 同上 |
| `PermissionDenied` | 工具名称 | 同上 |
| `SessionStart` | 会话如何启动 | `startup`, `resume`, `clear`, `compact` |
| `SessionEnd` | 会话为何结束 | `clear`, `resume`, `logout`, `prompt_input_exit` |
| `Setup` | 哪个 CLI 标志触发 | `init`, `maintenance` |
| `Notification` | 通知类型 | `permission_prompt`, `idle_prompt`, `auth_success`, `elicitation_dialog` |
| `SubagentStart` | 代理类型 | `general-purpose`, `Explore`, `Plan` 或自定义代理名称 |
| `SubagentStop` | 代理类型 | 同上 |
| `PreCompact` | 触发压缩的原因 | `manual`, `auto` |
| `PostCompact` | 触发压缩的原因 | 同上 |
| `ConfigChange` | 配置源 | `user_settings`, `project_settings`, `local_settings`, `policy_settings`, `skills` |
| `FileChanged` | 文字文件名（非正则） | `.envrc\|.env` |
| `StopFailure` | 错误类型 | `rate_limit`, `authentication_failed`, `oauth_org_not_allowed`, `billing_error`, `invalid_request`, `server_error`, `max_output_tokens`, `unknown` |
| `InstructionsLoaded` | 加载原因 | `session_start`, `nested_traversal`, `path_glob_match`, `include`, `compact` |
| `UserPromptExpansion` | 命令名称 | skill 或命令名称 |
| `Elicitation` | MCP 服务器名称 | 配置的 MCP 服务器名称 |
| `ElicitationResult` | MCP 服务器名称 | 同上 |

### 4.4 matcher 语法说明

matcher 字段是一个**正则表达式字符串**，评估方式取决于内容：

| matcher 值 | 评估方式 | 示例 |
|-----------|---------|------|
| `"*"`, `""` 或省略 | 匹配所有 | 在事件的每次出现时触发 |
| 仅字母、数字、`_` 和 `\|` | 精确字符串或 `\|` 分隔的列表 | `Bash` 仅匹配 Bash；`Edit\|Write` 匹配任一 |
| 包含其他字符 | JavaScript 正则表达式 | `^Notebook` 匹配以 Notebook 开头的工具 |

**注意**：`*` 和 `.*` 的区别：
- `*` 匹配**前面的字符** 0 次或多次（如 `Bash*` 匹配 `Bas`, `Bash`, `Bashh`）
- `.*` 匹配**任意字符** 0 次或多次（如 `Bash.*` 匹配 `Bash`, `BashCommand`）

### 4.5 MCP 工具匹配

MCP 工具遵循命名模式 `mcp__<server>__<tool>`：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "mcp__memory__.*",
        "hooks": [{ "type": "command", "command": "./log-memory.sh" }]
      }
    ]
  }
}
```

- `mcp__memory__.*` 匹配来自 `memory` 服务器的所有工具
- `mcp__.*__write.*` 匹配来自任何服务器的名称以 `write` 开头的工具

### 4.6 使用示例

#### 支持 matcher 的 hook
```python
# PreToolUse - 可以针对特定工具
HookMatcher(matcher="Bash", hooks=[my_callback])  # 只匹配 Bash 工具
HookMatcher(matcher="Edit|Write", hooks=[my_callback])  # 匹配 Edit 或 Write
```

#### 不支持 matcher 的 hook
```python
# Stop - matcher 会被静默忽略
HookMatcher(matcher=".*", hooks=[my_callback])  # matcher 无意义
HookMatcher(hooks=[my_callback])  # 推荐写法：不写 matcher
```

---

## 5. 各 Hook 类型详细说明

### 5.1 Stop Hook

**触发时机**：Claude 即将结束响应时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | 标识事件类型 | |
| `stop_hook_active` | boolean | 是 | Stop hook 是否激活 | 判断是否需要处理 | `if (input.stop_hook_active)` |
| `last_assistant_message` | string | 否 | 最后一条助手消息 | 分析对话内容、生成总结 | `const summary = generateSummary(input.last_assistant_message);` |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否继续 | 通常设为 true | `"continue": true` |
| `systemMessage` | string | 否 | 会话总结消息 | 向用户展示总结 | `"systemMessage": "本次会话已完成"` |
| `suppressOutput` | boolean | 否 | 是否隐藏输出 | 控制 transcript 显示 | `"suppressOutput": true` |
| `decision` | string | 否 | 权限决策 | `approve` 或 `block`，阻止后续操作 | `"decision": "block"` |
| `reason` | string | 否 | 决策原因 | 解释决策依据 | `"reason": "需要用户确认"` |
| `stopReason` | string | 否 | 停止原因 | 当 `continue: false` 时的停止说明 | `"stopReason": "任务已完成"` |

#### 输出字段发送给大模型的情况

根据 Claude Code 源码分析，Stop Hook 返回的字段中，**以下字段会发送给大模型**：

| 字段 | 是否发送给大模型 | 说明 |
|------|----------------|------|
| `continue: false` + `stopReason` | ✅ **是** | 通过 `hook_stopped_continuation` attachment 发送给模型，包装为 `<system-reminder>` 消息 |
| `decision: "block"` + `reason` | ✅ **是** | 通过 `hook_blocking_error` attachment 发送给模型，包装为 `<system-reminder>` 消息 |
| `systemMessage` | ❌ **否** | 被包装为 `hook_system_message` attachment，但在 `normalizeAttachmentForAPI` 中被过滤掉，不会发送给模型 |
| `suppressOutput` | ❌ **否** | 仅控制 stdout 是否在 transcript 中显示，不会发送给模型 |
| `continue: true` | ❌ **否** | 仅控制流程走向，不会发送给模型 |
| `additionalContext` | ❌ **否** | Stop Hook 不支持返回此字段（schema 中未定义） |

**重要说明**：
- 当 `continue: false` 时，必须提供 `stopReason`，该消息会作为系统提醒发送给大模型
- 当 `decision: "block"` 时，建议提供 `reason`，该消息会作为阻塞错误发送给大模型
- `systemMessage` 虽然文档中列出，但实际上**不会发送给大模型**，仅作为内部处理使用

#### 退出码含义
- `0`：stdout/stderr 不显示
- `2`：显示 stderr 给模型并继续对话
- 其他：仅显示 stderr 给用户

---

### 5.2 PreToolUse Hook

**触发时机**：工具执行前

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | 标识事件类型 | |
| `tool_name` | string | 是 | 工具名称 | 判断工具类型、应用规则 | `if (input.tool_name === 'Bash')` |
| `tool_input` | unknown | 是 | 工具输入参数 | 检查输入、修改参数 | `const filePath = input.tool_input.path;` |
| `tool_use_id` | string | 是 | 工具调用 ID | 跟踪工具调用 | `const id = input.tool_use_id;` |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否允许工具执行 | 阻止危险操作 | `"continue": false` |
| `systemMessage` | string | 否 | 阻止时的消息 | 解释阻止原因 | `"systemMessage": "危险操作已阻止"` |
| `hookSpecificOutput.hookEventName` | string | 是 | 事件名称 | `"hookEventName": "PreToolUse"` |
| `hookSpecificOutput.permissionDecision` | string | 否 | 权限决策 | `allow` 或 `deny` | `"permissionDecision": "deny"` |
| `hookSpecificOutput.permissionDecisionReason` | string | 否 | 权限决策原因 | 解释权限决策 | `"permissionDecisionReason": "安全风险"` |
| `hookSpecificOutput.updatedInput` | object | 否 | 修改后的输入 | 安全过滤、参数调整 | `"updatedInput": { "path": "/safe/file" }` |
| `hookSpecificOutput.additionalContext` | string | 否 | 附加上下文 | 向模型传递信息 | `"additionalContext": "已验证安全"` |

#### 退出码含义
- `0`：继续执行工具调用
- `2`：阻止工具调用并显示 stderr 给模型
- 其他：显示 stderr 给用户但继续执行

---

### 5.3 PostToolUse Hook

**触发时机**：工具执行成功后

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `tool_name` | string | 是 | 工具名称 | 判断工具类型 | |
| `tool_input` | unknown | 是 | 工具输入 | 分析输入输出关系 | |
| `tool_response` | unknown | 是 | 工具响应 | 处理工具执行结果 | `const result = input.tool_response;` |
| `tool_use_id` | string | 是 | 工具调用 ID | 关联调用记录 | |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否继续 | 控制后续流程 | |
| `hookSpecificOutput.hookEventName` | string | 是 | 事件名称 | `"hookEventName": "PostToolUse"` |
| `hookSpecificOutput.additionalContext` | string | 否 | 附加上下文 | 向模型传递结果分析 | `"additionalContext": "分析结果已保存"` |
| `hookSpecificOutput.updatedMCPToolOutput` | unknown | 否 | 更新后的工具输出 | 修改工具响应 | `"updatedMCPToolOutput": { "data": "modified" }` |

#### 退出码含义
- `0`：stdout 在 transcript 模式下显示
- `2`：立即显示 stderr 给模型
- 其他：仅显示 stderr 给用户

---

### 5.4 PostToolUseFailure Hook

**触发时机**：工具执行失败后

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `tool_name` | string | 是 | 工具名称 | | |
| `tool_input` | unknown | 是 | 工具输入 | | |
| `tool_use_id` | string | 是 | 工具调用 ID | | |
| `error` | string | 是 | 错误消息 | 分析错误原因 | `const errorMsg = input.error;` |
| `is_interrupt` | boolean | 否 | 是否被用户中断 | 判断中断原因 | `if (input.is_interrupt)` |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否继续 | 决定是否重试 | `"continue": true` |
| `systemMessage` | string | 否 | 错误处理消息 | 通知用户错误情况 | |
| `hookSpecificOutput.hookEventName` | string | 是 | 事件名称 | `"hookEventName": "PostToolUseFailure"` |
| `hookSpecificOutput.additionalContext` | string | 否 | 附加上下文 | 提供错误分析 | `"additionalContext": "建议检查网络连接"` |

---

### 5.5 PermissionRequest Hook

**触发时机**：显示权限对话框时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `tool_name` | string | 是 | 工具名称 | 判断权限需求 | |
| `tool_input` | unknown | 是 | 工具输入 | 评估风险 | |
| `permission_suggestions` | array | 否 | 权限建议 | 获取系统建议 | `input.permission_suggestions[0]` |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hookSpecificOutput.hookEventName` | string | 是 | 事件名称 | `"hookEventName": "PermissionRequest"` |
| `hookSpecificOutput.decision.behavior` | string | 是 | 权限决策 | `allow` 或 `deny` | `"behavior": "allow"` |
| `hookSpecificOutput.decision.updatedInput` | object | 否 | 修改后输入 | 安全过滤 | `"updatedInput": { "safeParam": "value" }` |
| `hookSpecificOutput.decision.updatedPermissions` | array | 否 | 更新后的权限列表 | 修改权限 | `"updatedPermissions": [...]` |
| `hookSpecificOutput.decision.message` | string | 否 | 决策消息（deny 时） | 解释拒绝原因 | `"message": "权限不足"` |
| `hookSpecificOutput.decision.interrupt` | boolean | 否 | 是否中断（deny 时） | 立即停止操作 | `"interrupt": true` |

---

### 5.6 PermissionDenied Hook

**触发时机**：自动模式拒绝工具调用时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `tool_name` | string | 是 | 工具名称 | | |
| `tool_input` | unknown | 是 | 工具输入 | | |
| `tool_use_id` | string | 是 | 工具调用 ID | | |
| `reason` | string | 是 | 拒绝原因 | 分析拒绝理由 | `const reason = input.reason;` |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hookSpecificOutput.hookEventName` | string | 是 | 事件名称 | `"hookEventName": "PermissionDenied"` |
| `hookSpecificOutput.retry` | boolean | 否 | 是否允许重试 | 允许模型重新尝试 | `"retry": true` |

---

### 5.7 UserPromptSubmit Hook

**触发时机**：用户提交提示词时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `prompt` | string | 是 | 用户原始提示词 | 审查、修改提示词 | `const userInput = input.prompt;` |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否继续处理 | 阻止不当请求 | `"continue": false` |
| `systemMessage` | string | 否 | 处理消息 | 通知用户处理结果 | |
| `hookSpecificOutput.hookEventName` | string | 是 | 事件名称 | `"hookEventName": "UserPromptSubmit"` |
| `hookSpecificOutput.additionalContext` | string | 否 | 附加上下文 | 向模型传递信息 | `"additionalContext": "已过滤敏感内容"` |

#### 退出码含义
- `0`：stdout 显示给 Claude
- `2`：阻止处理，擦除原始提示词
- 其他：仅显示 stderr 给用户

---

### 5.8 SessionStart Hook

**触发时机**：新会话启动时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `source` | string | 是 | 启动来源 | 判断启动方式 | `if (input.source === 'startup')` |
| `agent_type` | string | 否 | 代理类型 | 判断代理类型 | `if (input.agent_type)` |
| `model` | string | 否 | 使用的模型 | 获取模型信息 | `const model = input.model;` |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否继续 | 控制会话启动 | |
| `systemMessage` | string | 否 | 欢迎消息 | 自定义欢迎语 | `"systemMessage": "欢迎回来！"` |
| `hookSpecificOutput.hookEventName` | string | 是 | 事件名称 | `"hookEventName": "SessionStart"` |
| `hookSpecificOutput.additionalContext` | string | 否 | 附加上下文 | 提供会话初始化信息 | `"additionalContext": "项目已加载"` |
| `hookSpecificOutput.initialUserMessage` | string | 否 | 初始用户消息 | 设置初始提示词 | `"initialUserMessage": "请开始你的任务"` |
| `hookSpecificOutput.watchPaths` | array | 否 | 监视路径 | 注册文件监视器 | `"watchPaths": ["/project"]` |

---

### 5.9 SessionEnd Hook

**触发时机**：会话结束时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `reason` | string | 是 | 结束原因 | 判断结束类型 | `if (input.reason === 'logout')` |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否继续 | 确认结束 | |
| `systemMessage` | string | 否 | 结束消息 | 自定义结束提示 | `"systemMessage": "会话已结束"` |

---

### 5.10 Setup Hook

**触发时机**：仓库初始化或维护时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `trigger` | string | 是 | 触发类型 | `init` 或 `maintenance` | `if (input.trigger === 'init')` |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否继续 | 控制初始化流程 | |
| `hookSpecificOutput.hookEventName` | string | 是 | 事件名称 | `"hookEventName": "Setup"` |
| `hookSpecificOutput.additionalContext` | string | 否 | 附加上下文 | 提供初始化信息 | `"additionalContext": "依赖已安装"` |

---

### 5.11 SubagentStart Hook

**触发时机**：子代理启动时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `agent_id` | string | 是 | 子代理 ID | 标识子代理 | |
| `agent_type` | string | 是 | 代理类型 | 判断代理角色 | |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否继续 | 控制子代理启动 | |
| `hookSpecificOutput.hookEventName` | string | 是 | 事件名称 | `"hookEventName": "SubagentStart"` |
| `hookSpecificOutput.additionalContext` | string | 否 | 附加上下文 | 提供子代理信息 | `"additionalContext": "子代理已启动"` |

---

### 5.12 SubagentStop Hook

**触发时机**：子代理结束响应时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `stop_hook_active` | boolean | 是 | Stop hook 是否激活 | | |
| `agent_id` | string | 是 | 子代理 ID | 标识子代理 | |
| `agent_transcript_path` | string | 是 | 子代理转录路径 | 读取子代理对话 | |
| `agent_type` | string | 是 | 代理类型 | | |
| `last_assistant_message` | string | 否 | 最后消息 | 分析子代理输出 | |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否继续 | 控制子代理结束 | |
| `systemMessage` | string | 否 | 消息 | 发送给子代理 | `"systemMessage": "任务完成"` |

---

### 5.13 PreCompact Hook

**触发时机**：对话压缩前

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `trigger` | string | 是 | 触发类型 | `manual` 或 `auto` | |
| `custom_instructions` | string | 否 | 自定义压缩指令 | 获取现有指令 | `const instructions = input.custom_instructions;` |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否允许压缩 | 阻止压缩 | `"continue": false` |
| `systemMessage` | string | 否 | 压缩消息 | 自定义压缩提示 | `"systemMessage": "压缩已阻止"` |

#### 退出码含义
- `0`：stdout 作为自定义压缩指令
- `2`：阻止压缩
- 其他：继续压缩但显示 stderr

---

### 5.14 PostCompact Hook

**触发时机**：对话压缩后

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `trigger` | string | 是 | 触发类型 | `manual` 或 `auto` | |
| `compact_summary` | string | 是 | 压缩摘要 | 分析压缩结果 | `const summary = input.compact_summary;` |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否继续 | | |
| `systemMessage` | string | 否 | 压缩完成消息 | 通知用户压缩结果 | `"systemMessage": "对话已压缩"` |

---

### 5.15 Notification Hook

**触发时机**：发送通知时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `message` | string | 是 | 通知消息 | 获取通知内容 | |
| `title` | string | 否 | 通知标题 | 获取通知标题 | `const title = input.title;` |
| `notification_type` | string | 是 | 通知类型 | 判断通知种类 | |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hookSpecificOutput.hookEventName` | string | 是 | 事件名称 | `"hookEventName": "Notification"` |
| `hookSpecificOutput.additionalContext` | string | 否 | 附加上下文 | 提供额外信息 | `"additionalContext": "通知已处理"` |

---

### 5.16 Elicitation Hook

**触发时机**：MCP 服务器请求用户输入时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `mcp_server_name` | string | 是 | MCP 服务器名称 | 识别来源 | |
| `message` | string | 是 | 请求消息 | 获取请求内容 | |
| `mode` | string | 否 | 交互模式 | `form` 或 `url` | `if (input.mode === 'form')` |
| `url` | string | 否 | URL 模式的地址 | 获取 URL | `const url = input.url;` |
| `elicitation_id` | string | 否 | 请求 ID | 关联请求 | `const id = input.elicitation_id;` |
| `requested_schema` | object | 否 | 请求的 schema | 了解所需输入格式 | |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hookSpecificOutput.hookEventName` | string | 是 | 事件名称 | `"hookEventName": "Elicitation"` |
| `hookSpecificOutput.action` | string | 否 | 操作 | `accept`、`decline`、`cancel` | `"action": "accept"` |
| `hookSpecificOutput.content` | object | 否 | 响应内容 | 自动填写响应 | `"content": { "answer": "yes" }` |

---

### 5.17 ElicitationResult Hook

**触发时机**：用户响应后

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `mcp_server_name` | string | 是 | MCP 服务器名称 | 识别来源 | |
| `elicitation_id` | string | 否 | 请求 ID | 关联请求 | |
| `mode` | string | 否 | 交互模式 | `form` 或 `url` | |
| `action` | string | 是 | 用户操作 | 判断用户选择 | |
| `content` | object | 否 | 用户输入内容 | 获取用户响应 | |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hookSpecificOutput.hookEventName` | string | 是 | 事件名称 | `"hookEventName": "ElicitationResult"` |
| `hookSpecificOutput.action` | string | 否 | 覆盖操作 | 修改用户操作 | `"action": "decline"` |
| `hookSpecificOutput.content` | object | 否 | 覆盖内容 | 修改用户输入 | `"content": { "modified": "data" }` |

---

### 5.18 ConfigChange Hook

**触发时机**：配置文件变化时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `source` | string | 是 | 配置来源 | 判断变更来源 | |
| `file_path` | string | 否 | 文件路径 | 识别变更文件 | `const path = input.file_path;` |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否允许变更 | 阻止不安全变更 | `"continue": false` |
| `systemMessage` | string | 否 | 变更消息 | 通知用户变更情况 | `"systemMessage": "配置变更已阻止"` |

---

### 5.19 FileChanged Hook

**触发时机**：文件发生变化时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `file_path` | string | 是 | 文件路径 | 识别变化文件 | |
| `event` | string | 是 | 事件类型 | `change`、`add`、`unlink` | |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hookSpecificOutput.hookEventName` | string | 是 | 事件名称 | `"hookEventName": "FileChanged"` |
| `hookSpecificOutput.watchPaths` | array | 否 | 监视路径 | 动态更新监视列表 | `"watchPaths": ["/new/path"]` |

---

### 5.20 CwdChanged Hook

**触发时机**：工作目录变化时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `old_cwd` | string | 是 | 旧目录 | 比较目录变化 | |
| `new_cwd` | string | 是 | 新目录 | 获取新工作目录 | |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hookSpecificOutput.hookEventName` | string | 是 | 事件名称 | `"hookEventName": "CwdChanged"` |
| `hookSpecificOutput.watchPaths` | array | 否 | 监视路径 | 更新监视列表 | `"watchPaths": ["/new/cwd"]` |

---

### 5.21 WorktreeCreate Hook

**触发时机**：创建隔离工作树时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `name` | string | 是 | 建议的工作树名称 | 获取工作树标识 | |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| stdout | string | 是 | 工作树路径 | 输出创建的工作树目录绝对路径 | `/path/to/worktree` |
| `hookSpecificOutput.hookEventName` | string | 是 | 事件名称 | `"hookEventName": "WorktreeCreate"` |
| `hookSpecificOutput.worktreePath` | string | 是 | 工作树路径 | 提供创建的工作树路径 | `"worktreePath": "/path/to/worktree"` |

---

### 5.22 WorktreeRemove Hook

**触发时机**：删除工作树时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `worktree_path` | string | 是 | 工作树路径 | 识别要删除的工作树 | |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否继续删除 | 确认删除操作 | `"continue": true` |

---

### 5.23 TeammateIdle Hook

**触发时机**：队友即将进入空闲状态时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `teammate_name` | string | 是 | 队友名称 | 识别队友 | |
| `team_name` | string | 是 | 团队名称 | 识别团队 | |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否允许进入空闲状态 | 阻止空闲 | `"continue": false` |
| `systemMessage` | string | 否 | 消息 | 发送给队友 | `"systemMessage": "请继续工作"` |

---

### 5.24 TaskCreated Hook

**触发时机**：任务创建时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `task_id` | string | 是 | 任务 ID | 识别任务 | |
| `task_subject` | string | 是 | 任务主题 | 获取任务标题 | |
| `task_description` | string | 否 | 任务描述 | 获取任务详情 | |
| `teammate_name` | string | 否 | 创建任务的队友 | 识别创建者 | |
| `team_name` | string | 否 | 团队名称 | 识别团队 | |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否允许创建任务 | 阻止任务创建 | `"continue": false` |
| `systemMessage` | string | 否 | 消息 | 通知用户 | `"systemMessage": "任务创建已阻止"` |

---

### 5.25 TaskCompleted Hook

**触发时机**：任务标记为完成时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `task_id` | string | 是 | 任务 ID | 识别任务 | |
| `task_subject` | string | 是 | 任务主题 | 获取任务标题 | |
| `task_description` | string | 否 | 任务描述 | 获取任务详情 | |
| `teammate_name` | string | 否 | 完成任务的队友 | 识别完成者 | |
| `team_name` | string | 否 | 团队名称 | 识别团队 | |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否允许标记完成 | 阻止完成 | `"continue": false` |
| `systemMessage` | string | 否 | 消息 | 通知用户 | `"systemMessage": "任务完成已阻止"` |

---

### 5.26 StopFailure Hook

**触发时机**：API 错误导致会话结束时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `error` | string | 是 | 错误类型 | 判断错误原因 | |
| `error_details` | string | 否 | 错误详情 | 获取详细信息 | `const details = input.error_details;` |
| `last_assistant_message` | string | 否 | 最后消息 | 获取上下文 | |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| 无特定字段 | - | - | 输出被忽略 | fire-and-forget | - |

---

### 5.27 InstructionsLoaded Hook

**触发时机**：加载指令文件时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `file_path` | string | 是 | 文件路径 | 识别加载的文件 | |
| `memory_type` | string | 是 | 内存类型 | 判断文件类型 | |
| `load_reason` | string | 是 | 加载原因 | 判断加载触发方式 | |
| `globs` | array | 否 | 匹配的路径模式 | 获取匹配模式 | `const patterns = input.globs;` |
| `trigger_file_path` | string | 否 | 触发加载的文件路径 | 识别触发源 | `const trigger = input.trigger_file_path;` |
| `parent_file_path` | string | 否 | 包含此文件的父文件路径 | 识别包含关系 | `const parent = input.parent_file_path;` |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| 无特定字段 | - | - | 仅用于可观测性 | 不支持阻塞 | - |

---

## 6. Hook 上下文加载机制

### 6.1 Hook 返回的信息会加载到上下文中吗？

**是的，但处理方式取决于字段类型和 hook 执行模式。**

Hook 返回的 `additionalContext` 和 `systemMessage` 在 `processHookJSONOutput` 函数中被提取：

```typescript
// 来自 hooks.ts:621-628
case 'PostToolUse':
  result.additionalContext = json.hookSpecificOutput.additionalContext
  break
case 'UserPromptSubmit':
  result.additionalContext = json.hookSpecificOutput.additionalContext
  break
// ... 其他 hook 类型类似
```

这些返回值随后在 `processUserInput` 中被处理：

```typescript
// 来自 processUserInput.ts:227-239
if (hookResult.additionalContexts && hookResult.additionalContexts.length > 0) {
  result.messages.push(
    createAttachmentMessage({
      type: 'hook_additional_context',
      content: hookResult.additionalContexts.map(applyTruncation),
      // ...
    }),
  )
}
```

### 6.2 后续发送信息给大模型会将这些信息持续的发送给大模型吗？

**是的，hook 返回的信息会被构建成消息对象，并在后续 API 请求中发送给模型。**

#### 处理流程

```
Hook 执行 → 提取 additionalContext/systemMessage
    ↓
构建 attachment message (hook_additional_context / hook_system_message)
    ↓
在 processUserInput 中添加到 messages 数组
    ↓
mergeUserMessagesAndToolResults 合并到用户消息
    ↓
normalizeAttachmentForAPI 转换格式
    ↓
发送给大模型
```

#### systemMessage 的特殊处理

**同步 hook 的 `systemMessage`**：
- 被包装成 `hook_system_message` 类型的 attachment message
- 在 `normalizeAttachmentForAPI` 函数中被**过滤掉**，**不会发送给模型**

**异步 hook 的 `systemMessage`**：
- 通过 `enqueuePendingNotification` 作为 `task-notification` 加入队列
- 会被包装成带有 `<system-reminder>` 标签的用户消息
- **会发送给模型**

### 6.3 不同的 hook 或返回不同的字段，会影响效果吗？

**是的，效果完全不同。**

#### 各字段的影响

| 字段 | 作用 | 是否发送给模型 |
|------|------|---------------|
| `additionalContext` | 作为 `hook_additional_context` attachment message，包装成 `<system-reminder>` 标签的用户消息 | ✅ 是 |
| `systemMessage` (同步) | 作为 `hook_system_message` attachment message | ❌ **被过滤掉** |
| `systemMessage` (异步) | 作为 `task-notification` 发送 | ✅ 是 |
| `updatedInput` | 修改工具调用的输入参数 | ❌ 不发模型，影响执行 |
| `updatedMCPToolOutput` | 修改 MCP 工具的输出 | ❌ 不发模型，影响执行 |
| `decision` (PermissionRequest) | 控制权限决策 | ❌ 不发模型，影响执行 |
| `preventContinuation` | 阻止操作继续执行 | ❌ 不发模型，中断流程 |

#### 不同 hook 类型的 additionalContext 处理

所有支持 `additionalContext` 的 hook 类型：

- `PreToolUse` / `PostToolUse` / `PostToolUseFailure`
- `UserPromptSubmit`
- `SessionStart` / `Setup`
- `SubagentStart`
- `PermissionRequest` / `PermissionDenied`

它们的 `additionalContext` 都会被收集并发送给模型。

#### 异步 Hook 的特殊行为

```typescript
// 异步 hook 完成时检查响应
const lines = stdout.split('\n')
for (const line of lines) {
  if (line.trim().startsWith('{')) {
    const parsed = jsonParse(line.trim())
    if (!('async' in parsed)) {
      // 这是最终响应
      response = parsed
      break
    }
  }
}
```

异步 hook 的结果会在**下一轮对话**时生效（用户发送新消息或模型准备下一轮生成）。

---

## 7. 异步 Hook 详解

### 7.1 同步 vs 异步对比

| 维度 | 同步 Hook（默认） | 异步 Hook（`async: true`） |
|------|-----------------|--------------------------|
| 阻塞主流程 | ✅ 是（必须等完成） | ❌ 否（后台跑，立即继续） |
| 支持决策 | ✅ 是（可 deny/block） | ❌ 否（决策字段无效） |
| 内容生效时机 | ✅ 当前轮次立即生效 | ❌ 下一轮对话才生效 |
| 支持类型 | command/prompt/agent/http | ✅ 仅 command |
| 适用场景 | 权限、阻断、实时修改 | 日志、通知、后台任务 |

### 7.2 异步配置方式

**方式一：配置文件中设置**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": { "tool": "Write|Edit|Bash" },
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/audit-log.sh",
            "async": true,
            "timeout": 120
          }
        ]
      }
    ]
  }
}
```

**方式二：脚本返回值设置**（优先级更高）
```bash
#!/bin/bash
echo '{"async": true, "asyncTimeout": 60000}'
sleep 10
echo '{"systemMessage": "审计日志已记录"}'
```

### 7.3 systemMessage 注入机制

**Hook 返回的 `systemMessage` 并不是作为系统提示词注入的，而是作为带有 `<system-reminder>` 标签的用户消息注入到对话中的。**

- **同步 hook**：`systemMessage` 被包装成 `hook_system_message` 类型的 attachment message，在 `normalizeAttachmentForAPI` 函数中被过滤掉，**不会发送给模型**。
- **异步 hook**：`systemMessage` 通过 `enqueuePendingNotification` 作为 `task-notification` 被加入消息队列，被包装成带有 `<system-reminder>` 标签的用户消息，**作为用户输入的一部分发送给模型**。

### 7.4 异步模式启动

脚本需要先输出异步标识：
```json
{
  "async": true,
  "asyncTimeout": 60000
}
```

### 7.5 异步执行完成

后台执行完成后输出最终结果：
```json
{
  "systemMessage": "异步任务完成",
  "continue": true
}
```

---

## 8. 执行机制与并发处理

### 8.1 注册来源

Hook 从四个来源汇聚，在 `getHooksConfig()` 中合并为一个数组：

| 来源 | 存储位置 | 生命周期 |
|------|----------|----------|
| **settings 文件** | `userSettings` / `projectSettings` / `localSettings` | 持久化 |
| **全局注册 hook** | `STATE.registeredHooks` | 进程级别 |
| **会话 hook** | `appState.sessionHooks` (Map) | Agent/Skill 生命周期 |
| **函数 hook** | `appState.sessionHooks` (FunctionHook) | 会话级别 |

### 8.2 执行模型

**REPL 上下文**（`executeHooks()`）：
- 所有匹配的 hook 转为 async generator 数组
- 通过 `all()` 函数并发消费这些 generator
- 使用 `Promise.race()` 轮询哪个先 yield 出值
- 先完成的 hook 先被消费，不保证顺序

**REPL 外上下文**（`executeHooksOutsideREPL()`）：
- 同样是并行执行，使用 `Promise.all`

### 8.3 独立超时控制

每个 hook 有独立的超时：
- 可通过配置设置 `timeout`（秒）
- 默认超时：`TOOL_HOOK_EXECUTION_TIMEOUT_MS`（15秒）
- 一个 hook 超时完全不影响其他 hook

### 8.4 结果聚合与干扰分析

#### 不会相互干扰的方面

| 方面 | 原因 |
|------|------|
| **执行超时** | 每个 hook 有独立的 AbortSignal |
| **进程隔离** | command hooks 各自启动独立子进程 |
| **JSON 输出解析** | 每个 hook 的 stdout 独立解析 |
| **后台异步** | `AsyncHookRegistry` 按 `processId` 独立管理 |
| **执行错误** | 单个 hook 异常被隔离 |

#### 可能相互干扰的方面

| 方面 | 行为 | 风险等级 |
|------|------|----------|
| **`permissionBehavior`** | `deny > ask > allow` 优先级规则 | 高 |
| **`preventContinuation`** | 任一 hook 设置即生效，其他被覆盖 | 中 |
| **`updatedInput`** | 仅最后一轮生效 | 中 |
| **`blockingError`** | 全部收集，多个都会传递 | 低 |
| **`additionalContexts`** | 全部合并，不丢失 | 低 |
| **`watchPaths`** | 全部合并 | 低 |
| **`initialUserMessage`** | 取最后一个设置的 | 低 |

### 8.5 permissionBehavior 优先级规则

```typescript
case 'deny':  // deny 总是优先
  permissionBehavior = 'deny'; break;
case 'ask':   // 仅当未设置 deny
  if (permissionBehavior !== 'deny') permissionBehavior = 'ask'; break;
case 'allow': // 仅当未设置其他
  if (!permissionBehavior) permissionBehavior = 'allow'; break;
```

---

## 9. 脚本读取输入示例

### 9.1 Python
```python
#!/usr/bin/env python3
import sys
import json

# 从 stdin 读取 JSON 输入
input_data = json.load(sys.stdin)

session_id = input_data['session_id']
cwd = input_data['cwd']
hook_event = input_data['hook_event_name']

print(f"Session ID: {session_id}")
print(f"CWD: {cwd}")
print(f"Hook Event: {hook_event}")

# 输出 JSON 结果
print(json.dumps({"continue": True}))
```

### 9.2 Bash
```bash
#!/bin/bash
# 从 stdin 读取 JSON 输入
INPUT=$(cat)

# 使用 jq 解析字段
SESSION_ID=$(echo "$INPUT" | jq -r '.session_id')
HOOK_EVENT=$(echo "$INPUT" | jq -r '.hook_event_name')

echo "Session ID: $SESSION_ID"
echo "Hook Event: $HOOK_EVENT"

# 输出 JSON 结果
echo '{"continue": true}'
```

### 9.3 Node.js
```javascript
#!/usr/bin/env node
const readline = require('readline');

const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout,
  terminal: false
});

let input = '';
rl.on('line', (line) => {
  input += line;
});

rl.on('close', () => {
  const data = JSON.parse(input);
  console.log('Session ID:', data.session_id);
  console.log('Hook Event:', data.hook_event_name);
  
  // 输出 JSON 结果
  console.log(JSON.stringify({ continue: true }));
});
```

---

## 10. 注意事项

### 10.1 stdin 读取时机
- 对于**同步 Hook**：脚本需要在执行期间读取 stdin，读取完成后立即输出结果
- 对于**异步 Hook**：系统会先写入 `jsonInput + '\n'`，然后结束 stdin，脚本需要在后台执行完成前读取输入

### 10.2 JSON 格式
- 输入始终是有效的 JSON，无需处理解析错误
- 输出必须是有效的 JSON，否则会导致解析错误

### 10.3 字符编码
- 使用 UTF-8 编码
- 特殊字符需要正确转义

### 10.4 字段存在性
- 某些可选字段可能不存在，使用时需要进行存在性检查
- 建议使用 `input.get('field_name')` 或条件判断来访问可选字段

### 10.5 换行符
- 输入末尾会有一个换行符，脚本读取时需要注意处理
- 使用 `cat`、`json.load(sys.stdin)` 等方式会自动处理换行符

### 10.6 退出码优先级
- 退出码会覆盖 JSON 中的 `continue` 字段
- 退出码 `2` 会触发特殊处理逻辑

### 10.7 systemMessage 的特殊行为
- **同步 Hook**：`systemMessage` 不会发送给大模型，仅作为内部处理
- **异步 Hook**：`systemMessage` 会通过 `task-notification` 发送给大模型

---

## 11. 完整 Hook 事件类型列表

| 序号 | 事件类型 | 触发时机 |
|------|---------|---------|
| 1 | `PreToolUse` | 工具执行前 |
| 2 | `PostToolUse` | 工具执行成功后 |
| 3 | `PostToolUseFailure` | 工具执行失败后 |
| 4 | `PermissionRequest` | 显示权限对话框时 |
| 5 | `PermissionDenied` | 自动模式拒绝工具调用时 |
| 6 | `UserPromptSubmit` | 用户提交提示词时 |
| 7 | `SessionStart` | 新会话启动时 |
| 8 | `SessionEnd` | 会话结束时 |
| 9 | `Stop` | Claude 即将结束响应时 |
| 10 | `StopFailure` | API 错误导致会话结束时 |
| 11 | `SubagentStart` | 子代理启动时 |
| 12 | `SubagentStop` | 子代理即将结束响应时 |
| 13 | `PreCompact` | 对话压缩前 |
| 14 | `PostCompact` | 对话压缩后 |
| 15 | `Notification` | 发送通知时 |
| 16 | `Setup` | 仓库初始化或维护时 |
| 17 | `TeammateIdle` | 队友即将进入空闲状态时 |
| 18 | `TaskCreated` | 任务创建时 |
| 19 | `TaskCompleted` | 任务标记为完成时 |
| 20 | `Elicitation` | MCP 服务器请求用户输入时 |
| 21 | `ElicitationResult` | 用户响应用户输入请求后 |
| 22 | `ConfigChange` | 配置文件发生变化时 |
| 23 | `InstructionsLoaded` | 加载指令文件时 |
| 24 | `WorktreeCreate` | 创建隔离工作树时 |
| 25 | `WorktreeRemove` | 删除工作树时 |
| 26 | `CwdChanged` | 工作目录发生变化时 |
| 27 | `FileChanged` | 监听的文件发生变化时 |

---

## 附录 A：支持 additionalContext 的 Hook 类型

以下 Hook 类型支持返回 `additionalContext` 字段，该字段会作为 `hook_additional_context` attachment 发送给大模型：

| Hook 类型 | 支持 additionalContext |
|---------|----------------------|
| `PreToolUse` | ✅ |
| `PostToolUse` | ✅ |
| `PostToolUseFailure` | ✅ |
| `UserPromptSubmit` | ✅ |
| `SessionStart` | ✅ |
| `Setup` | ✅ |
| `SubagentStart` | ✅ |
| `Notification` | ✅ |

以下 Hook 类型**不支持**返回 `additionalContext`：

| Hook 类型 | 原因 |
|---------|------|
| `Stop` | schema 中未定义 |
| `SubagentStop` | schema 中未定义 |
| `SessionEnd` | schema 中未定义 |
| `PreCompact` | schema 中未定义 |
| `PostCompact` | schema 中未定义 |
| `PermissionRequest` | 使用 decision 字段控制 |
| `PermissionDenied` | 使用 retry 字段控制 |
| `StopFailure` | fire-and-forget，不处理输出 |

---

## 附录 B：systemMessage 的处理差异

| Hook 执行模式 | systemMessage 处理方式 |
|-------------|---------------------|
| 同步 Hook | `hook_system_message` → 被 `normalizeAttachmentForAPI` 过滤 → **不发送** |
| 异步 Hook | `enqueuePendingNotification` → `task-notification` → **发送给模型** |

---

**文档版本**: 1.3  
**更新日期**: 2026-05-11  
**更新内容**:
- 新增第4章 "Matcher 支持情况"，详细说明哪些 Hook 支持/不支持 matcher
- 补充各事件的 matcher 匹配内容对照表
- 说明 `*` 与 `.*` 在正则表达式中的区别
- 补充 Stop Hook 输出字段发送给大模型的详细说明
- 新增第6章 "Hook 上下文加载机制"，解答关于上下文加载和字段效果的常见问题
- 新增附录说明支持 additionalContext 的 Hook 类型
- 新增附录说明 systemMessage 的处理差异
