# Claude Hook 脚本输入输出完整参考

## 概述

本文档详细描述了 Claude Hook 脚本的输入输出规范，按 Hook 事件类型组织。每个 Hook 类型包含：
- **触发时机**：何时触发该 Hook
- **输入字段**：脚本从 stdin 接收的参数
- **输出字段**：脚本通过 stdout 返回的结果
- **退出码含义**：不同退出码的处理方式
- **使用示例**：实际使用示例

---

## 通用输入字段（所有 Hook 通用）

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `session_id` | string | 是 | 当前会话的唯一标识符 | 跟踪会话、关联数据 | `const sessionId = input.session_id;` |
| `transcript_path` | string | 是 | 会话转录文件路径 | 读取会话历史 | `const history = fs.readFileSync(input.transcript_path);` |
| `cwd` | string | 是 | 当前工作目录 | 文件操作、路径解析 | `process.chdir(input.cwd);` |
| `permission_mode` | string | 否 | 当前权限模式 | 权限判断、访问控制 | `if (input.permission_mode === 'safe')` |
| `agent_id` | string | 否 | 子代理标识符 | 区分子代理和主线程 | `if (input.agent_id)` |
| `agent_type` | string | 否 | 代理类型名称 | 识别代理类型 | `if (input.agent_type === 'code-reviewer')` |

---

## 通用输出字段（所有 Hook 通用）

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

## 各 Hook 类型详细说明

### 1. Stop Hook

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

#### 退出码含义
- `0`：stdout/stderr 不显示
- `2`：显示 stderr 给模型并继续对话
- 其他：仅显示 stderr 给用户

---

### 2. PreToolUse Hook

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

### 3. PostToolUse Hook

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

### 4. PostToolUseFailure Hook

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

### 5. PermissionRequest Hook

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

### 6. PermissionDenied Hook

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

### 7. UserPromptSubmit Hook

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

### 8. SessionStart Hook

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

### 9. SessionEnd Hook

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

### 10. Setup Hook

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

### 11. SubagentStart Hook

**触发时机**：子代理启动时

#### 输入字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|------|---------|
| `hook_event_name` | string | 是 | 事件名称 | | |
| `agent_id` | string | 是 | 子代理 ID | 标识子代理 | |
| `agent_type` | string | 是 | 代理类型 | 判断代理角色 | |

#### 输出字段

| 字段 | 类型 | 必填 | 意义 | 用途 | 使用示例 |
|------|------|------|------|---------|
| `continue` | boolean | 否 | 是否继续 | 控制子代理启动 | |
| `hookSpecificOutput.hookEventName` | string | 是 | 事件名称 | `"hookEventName": "SubagentStart"` |
| `hookSpecificOutput.additionalContext` | string | 否 | 附加上下文 | 提供子代理信息 | `"additionalContext": "子代理已启动"` |

---

### 12. SubagentStop Hook

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

### 13. PreCompact Hook

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

### 14. PostCompact Hook

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

### 15. Notification Hook

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

### 16. Elicitation Hook

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

### 17. ElicitationResult Hook

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

### 18. ConfigChange Hook

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

### 19. FileChanged Hook

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

### 20. CwdChanged Hook

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

### 21. WorktreeCreate Hook

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

### 22. WorktreeRemove Hook

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

### 23. TeammateIdle Hook

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

### 24. TaskCreated Hook

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

### 25. TaskCompleted Hook

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

### 26. StopFailure Hook

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

### 27. InstructionsLoaded Hook

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

## 异步 Hook 特殊说明

### 异步模式启动

脚本需要先输出异步标识：

```json
{
  "async": true,
  "asyncTimeout": 60000
}
```

### 异步执行完成

后台执行完成后输出最终结果（使用对应的 Hook 类型输出字段）：

```json
{
  "systemMessage": "异步任务完成",
  "continue": true
}
```

---

## 退出码汇总

| 退出码 | 通用含义 |
|--------|---------|
| `0` | 成功，按默认方式处理 |
| `2` | 特殊处理，显示 stderr 给模型 |
| 其他 | 仅显示 stderr 给用户 |

---

## 脚本读取输入示例

### Python
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

### Bash
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

### Node.js
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

## 注意事项

### stdin 读取时机
- 对于**同步 Hook**：脚本需要在执行期间读取 stdin，读取完成后立即输出结果
- 对于**异步 Hook**：系统会先写入 `jsonInput + '\n'`，然后结束 stdin，脚本需要在后台执行完成前读取输入

### JSON 格式
- 输入始终是有效的 JSON，无需处理解析错误
- 输出必须是有效的 JSON，否则会导致解析错误

### 字符编码
- 使用 UTF-8 编码
- 特殊字符需要正确转义

### 字段存在性
- 某些可选字段可能不存在，使用时需要进行存在性检查
- 建议使用 `input.get('field_name')` 或条件判断来访问可选字段

### 换行符
- 输入末尾会有一个换行符，脚本读取时需要注意处理
- 使用 `cat`、`json.load(sys.stdin)` 等方式会自动处理换行符

### 退出码优先级
- 退出码会覆盖 JSON 中的 `continue` 字段
- 退出码 `2` 会触发特殊处理逻辑

---

## 完整 Hook 事件类型列表

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