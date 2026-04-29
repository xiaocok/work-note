

systemMessage

Hook 返回的 systemMessage 并不是作为系统提示词注入的，而是作为带有 <system-reminder> 标签的用户消息注入到对话中的 。



对于同步 hook， systemMessage 被包装成 hook_system_message 类型的 attachment message：

同步 hook 的 systemMessage ： hook_system_message 类型的 attachment message 在 normalizeAttachmentForAPI 函数中被过滤掉（返回 [] ），不会发送给模型。



对于异步 hook（特别是 asyncRewake 类型）， systemMessage 通过 enqueuePendingNotification 作为 task-notification 被加入消息队列：

异步 hook 的 systemMessage ： task-notification 类型的消息被包装成带有 <system-reminder> 标签的用户消息，作为用户输入的一部分发送给模型。



Hook 的异步标识是在脚本执行时通过返回值设置的，而不是在注册时的 hooks.json 中配置的 。

### 2. 脚本执行时的返回值
异步标识是在 Hook 脚本执行时，通过返回包含 async: true 的 JSON 对象来设置的：

```json
{
  "async": true,
  "asyncTimeout": 60000
}
```



# 异步 Hook 的执行与输出机制
## 详细执行流程
### 1. 第一次输出：异步标识
- 脚本开始执行，立即输出包含 async: true 的 JSON：
  ```
  {
    "async": true,
    "asyncTimeout": 60000
  }
  ```
- 系统捕获到这个输出，识别为异步 Hook，启动后台执行
### 2. 脚本继续执行
- 脚本不会因为输出了异步标识而终止，而是继续执行后续的任务：
  - 例如，发起网络请求、处理数据、调用外部 API 等
  - 执行耗时操作，如文件处理、数据分析等
### 3. 最终输出：Hook 规范内容
- 脚本执行完成后，输出符合 Hook 规范的内容：
  - 可以是包含 systemMessage 的 JSON：
    ```
    {
      "systemMessage": "异步任务执行完成，分析结果已生成",
      "continue": true
    }
    ```
  - 也可以是普通文本：
    ```
    异步任务执行完成，分析结果已生成
    ```
- 系统捕获到这个最终输出，作为通知发送给大模型
## 系统处理机制
在 executeAsyncHook 函数中，系统会：

1. 首次执行 ：运行脚本并检查是否返回 async: true
2. 后台执行 ：如果是异步，在后台继续执行脚本直到完成
3. 捕获最终输出 ：读取脚本的最终 stdout 和 stderr
4. 生成通知 ：将最终输出包装成通知发送给大模型
## 脚本编写示例
```
// async-hook.js
// 1. 首先输出异步标识
console.log(JSON.stringify({
  "async": true,
  "asyncTimeout": 60000
}));

// 2. 继续执行耗时任务
setTimeout(() => {
  // 3. 执行完成后输出最终结果
  console.log(JSON.stringify({
    "systemMessage": "异步分析完成，发现以下问题：\n1. 代码中存在未使用
    的变量\n2. 函数命名不够规范",
    "continue": true
  }));
}, 5000);
```
## 注意事项
1. 脚本执行环境 ：异步脚本需要在后台持续运行，直到任务完成
2. 输出顺序 ：必须先输出异步标识，再输出最终结果
3. 超时设置 ：建议设置合理的 asyncTimeout ，防止脚本无限执行
4. 错误处理 ：如果脚本执行出错，错误信息会被作为通知发送给大模型
## 结论
异步 Hook 的工作原理是：脚本先输出异步标识，让系统知道它需要在后台执行，然后脚本继续执行耗时任务，完成后输出最终结果，系统将这个结果作为通知发送给大模型。这种设计允许 Hook 执行长时间运行的任务，而不会阻塞主流程。





# Claude Code 异步 Hook 完整指南

### 一、什么时候适合用异步 Hook（`async: true`）

**核心原则**：只在**不影响主流程、不需要阻塞决策、只做副作用**的场景用。

#### ✅ 最佳适用场景

1. **日志 / 审计 / 监控**（最常用）

   - 记录工具调用、文件修改、命令执行到本地 / 远程日志系统
   - 发送操作到监控 / 告警服务（如 Prometheus、Slack、企业微信）

   

2. **后台任务 / 非实时处理**

   - 代码写完后异步跑单元测试、代码扫描、格式化
   - 自动备份、数据同步、构建产物上传

   

3. **通知 / 提醒**

   - 任务完成、关键操作、错误发生时发通知
   - 不阻塞主对话，后台触发即可

   

4. **数据采集 / 分析**

   - 收集会话数据、代码质量指标、用户行为
   - 不影响模型思考与执行

   

#### ❌ 绝对不适合的场景（必须同步）

- **权限控制**（`PreToolUse` 的 `permissionDecision`）
- **流程阻断 / 决策**（`Stop` 的 `decision`、`continueSession`）
- **需要立即修改上下文 / 提示词**（`systemMessage`/`additionalContext` 要立即生效）
- **依赖 Hook 结果才能继续的逻辑**Anthropic

------

### 二、异步 Hook 怎么配置（仅支持 `command` 类型）

在 `hooks.json` 中给 `command` 类型 Hook 加 `"async": true` 即可。

json









```
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": { "tool": "Write|Edit|Bash" },
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/audit-log.sh",
            "async": true, // ✅ 核心配置
            "timeout": 120 // 后台超时（秒）
          }
        ]
      }
    ]
  }
}
```

#### 关键限制

- **仅 `type: "command"` 支持 `async`**；`prompt`/`agent`/`http` 不支持
- 异步 Hook **不能返回决策类字段**：`permissionDecision`/`decision`/`continue` 等会被忽略Anthropic
- 超时是**后台执行超时**，不阻塞主流程

------

### 三、异步 Hook 如何把内容传给大模型

#### 1. 传输机制（官方设计）

- 异步 Hook 在**后台独立进程**执行，**不阻塞主会话**

- 执行完成后，若输出 **合法 JSON** 且包含：

  - `additionalContext`
  - `systemMessage`
  - `tool_output`（`PostToolUse`）

  

- 这些内容会被**暂存**，在**下一轮对话（用户发新消息 / 模型下一次思考）** 时，**合并注入到模型上下文**。

#### 2. 异步脚本输出示例（`audit-log.sh`）

bash



运行







```
#!/bin/bash
# 读取 Claude 传入的事件 JSON（stdin）
input=$(cat)

# 后台处理：写日志、发通知、跑测试等
echo "$(date) | $input" >> .claude/async-hook.log

# 若需要给模型传上下文，输出 JSON（仅在下一轮生效）
cat <<EOF
{
  "additionalContext": "【异步日志】已记录工具调用：$(echo $input | jq -r '.tool_name')",
  "systemMessage": "注意：所有操作已异步审计，请勿重复执行危险命令"
}
EOF
```

#### 3. 传输规则

- 只有**完成并正常退出（exit 0）** 的异步 Hook 输出才会被收集
- 多个异步 Hook 的同名字段（如 `additionalContext`）**按完成顺序合并 / 覆盖**（无优先级）
- 输出非 JSON、超时、报错的异步 Hook：**内容丢弃，不影响主流程**

------

### 四、大模型如何接收与处理异步 Hook

#### 1. 接收时机

- 模型**不会在 Hook 触发的当前轮次**接收异步内容

- 仅在：

  - 用户发送**新消息**
  - 模型完成当前思考 / 工具调用，准备**下一轮生成**

  

- 才会把**所有已完成的异步 Hook 输出**合并到上下文。

#### 2. 处理流程（时序图）

plaintext









```
用户发消息 → 模型思考 → 触发 PreToolUse（同步，可阻断）
→ 执行工具 → 触发 PostToolUse（异步，后台跑）
→ 模型立即返回结果给用户（不等待异步 Hook）
→ 用户发下一条消息 / 模型下一轮思考
→ 合并所有已完成异步 Hook 的 additionalContext/systemMessage
→ 模型基于“原上下文 + 异步注入内容”继续生成
```

#### 3. 优先级与合并规则

- 异步 Hook 内容**优先级低于**：

  - 同步 Hook 的 `systemMessage`/`additionalContext`
  - Skill 里的固定提示词
  - 用户当前轮次的消息

  

- 合并方式：

  - `additionalContext`：**追加**（多个异步 Hook 会拼接）
  - `systemMessage`：**覆盖**（最后完成的异步 Hook 生效）
  - 其他字段：按事件类型规则合并（如 `tool_output`）

  

------

### 五、异步 vs 同步 Hook 核心区别（速查表）

表格







|     维度     |     同步 Hook（默认）     | 异步 Hook（`async: true`） |
| :----------: | :-----------------------: | :------------------------: |
|  阻塞主流程  |    ✅ 是（必须等完成）     |  ❌ 否（后台跑，立即继续）  |
|   支持决策   |   ✅ 是（可 deny/block）   |    ❌ 否（决策字段无效）    |
| 内容生效时机 |    ✅ 当前轮次立即生效     |     ❌ 下一轮对话才生效     |
|   支持类型   | command/prompt/agent/http |        ✅ 仅 command        |
|   适用场景   |   权限、阻断、实时修改    |    日志、通知、后台任务    |
|   冲突处理   |  并行 + 覆盖（不可预测）  |   完成顺序 + 合并 / 覆盖   |

------

### 六、实战建议（避坑）

1. **异步只做 “副作用”**：日志、通知、备份、测试，**不做决策**
2. **需要实时影响模型 → 必须同步**：权限检查、关键阻断、即时提示
3. **异步输出尽量用 `additionalContext`（追加）**，少用 `systemMessage`（覆盖）
4. **调试**：用 `claude --debug` 查看异步 Hook 启动 / 完成 / 输出日志
5. **超时设置**：后台任务设合理 `timeout`（如测试 120s，日志 30s）



## 异步 Hook 的配置 

------

# 一、精准配置位置

`async: true`**只写在这里**：

在你注册 `command` 类型 Hook 的**配置对象中**，和 `type`/`command` 平级。

### 正确配置示例

```json
// .claude/hooks.json 或 .claude/settings.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "AskUserQuestion|Bash",
        "hooks": [
          {
            "type": "command",       // 固定
            "command": "./hook.sh",  // 你的脚本路径
            "async": true,           // ✅ 异步配置：只在这里！
            "timeout": 60            // 可选：后台超时秒数
          }
        ]
      }
    ]
  }
}
```

------

# 二、脚本侧：**完全不用改！**

你的 hook 脚本（sh/node/python）**和同步脚本写法一模一样**：

1. 读取 STDIN 输入
2. 执行业务逻辑
3. 输出标准 JSON
4. **不需要加任何异步标识、字段、参数**

### 脚本示例（同步 / 异步 完全通用）

```bash
#!/bin/bash
# 读取输入
input=$(cat)

# 执行业务逻辑（日志/通知/上报）
echo "Hook 执行: $input" >> hook.log

# 输出标准JSON（和同步完全一样！无异步字段）
jq -n '{
  hookSpecificOutput: { hookEventName: "PostToolUse", additionalContext: "操作已记录" }
}'
```

------

# 四、关键规则（官方硬性规定）

1. **只有 `type: "command"` 支持 `async: true`**
   - `prompt`/`agent`/`http` 类型 Hook **不支持异步**
2. **`async: true` 是给 Claude Code 引擎看的**
   - 告诉引擎：**别等这个脚本跑完，直接继续主流程**
3. **脚本完全感知不到自己是同步还是异步**
   - 执行逻辑、输入、输出 **完全无区别**

------

# 五、同步 vs 异步 配置对比（一眼看懂）

|       模式       |        配置位置        |        脚本写法        |      主流程      |
| :--------------: | :--------------------: | :--------------------: | :--------------: |
| **同步（默认）** |      不写 `async`      |        标准脚本        |   等待脚本完成   |
|     **异步**     | 配置里写 `async: true` | **完全一样的标准脚本** | 不等待，后台运行 |

------

# 六、最终一句话总结

## 异步 = 配置文件里加 `async: true`

## 脚本 = 啥都不用改，和同步一模一样 

这就是 Claude Code 异步 Hook 的**唯一正确规则**。





# 异步 Hook 配置位置的确认
## 核心发现
根据代码检查，异步 Hook 的配置有两种方式 ：

1. 在配置文件中直接设置 （如你提供的示例）
2. 在脚本执行时通过返回值设置 （之前讨论的方式）
## 代码证据
### 1. 配置文件中的异步设置
在 hooks.ts 文件中，系统会检查配置中的 async 字段：

```
function getHookDefinition(hookConfig: any): HookDefinition {
  // ...
  const async = hookConfig.async;
  const timeout = hookConfig.timeout;
  // ...
  return {
    // ...
    async,
    timeout,
    // ...
  };
}
```
### 2. 脚本返回的异步设置
同时，系统也会检查脚本返回的 async 字段：

```
// 执行脚本获取返回值
const { stdout, stderr } = await runCommand({
  command: scriptPath,
  // ...
});

// 解析返回值
const syncJson = parseJSON(stdout);

// 检查是否为异步 Hook
if (syncJson.async === true) {
  // 处理异步 Hook
  const asyncTimeout = syncJson.asyncTimeout || 
  DEFAULT_ASYNC_HOOK_TIMEOUT;
  // 启动异步执行...
}
```
### 3. 优先级
如果配置文件和脚本返回都设置了 async ， 脚本返回的 async 字段优先级更高 ，会覆盖配置文件中的设置。

## 两种配置方式的区别
配置方式 位置 优先级 灵活性 配置文件 hooks.json 或类似配置文件 低 固定配置，适用于所有执行 脚本返回 脚本执行时的返回值 高 动态配置，可根据执行时的情况决定

## 示例说明
### 配置文件方式
```
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": { "tool": "Write|Edit|Bash" },
        "hooks": [
          {
            "type": "command",
            "command": "./scripts/audit-log.sh",
            "async": true, // 配置文件中设置
            "timeout": 120 // 后台超时（秒）
          }
        ]
      }
    ]
  }
}
```
### 脚本返回方式
```
// audit-log.sh
#!/bin/bash
# 脚本返回方式设置
echo '{"async": true, "asyncTimeout": 120000}'

# 执行耗时任务
sleep 10
echo '{"systemMessage": "审计日志已记录"}'
```
## 结论
异步 Hook 的配置可以在两个地方设置：

1. 配置文件中 ：通过 async 字段设置
2. 脚本返回值中 ：通过返回包含 async: true 的 JSON 设置
脚本返回值的优先级更高 ，如果脚本返回了 async 字段，会覆盖配置文件中的设置。

这种设计提供了灵活性，允许 Hook 根据执行时的具体情况动态决定是否异步执行。





# 异步 Hook 模板（含日志 + 通知 + 上下文注入），并标注每一步的作用

严格遵循 Claude Code 官方规范，包含 **本地日志记录 + 外部通知 + 上下文注入** 三大核心功能，**异步执行不阻塞主流程**，逐行标注作用，复制即用！

## 适用场景

- 工具执行后**后台记录日志**
- 发送操作通知（企业微信 / 钉钉 / Slack）
- 异步给模型注入上下文信息
- 所有**不影响主流程**的副作用操作

------

## 一、核心配置文件

### 文件路径：`你的项目/.claude/settings.json`

```json
{
  "hooks": {
    // 监听 工具执行完成 事件
    "PostToolUse": [
      {
        "matcher": "*",        // * = 匹配所有工具（Bash/Write/AskUserQuestion等）
        "hooks": [
          {
            "type": "command",
            "command": ".claude/async-hook.sh",  // 钩子脚本路径
            "async": true,           // ✅ 核心：开启异步执行（后台运行）
            "timeout": 60            // 异步超时时间（秒），防止卡死
          }
        ]
      }
    ]
  }
}
```

------

## 二、异步钩子脚本（核心）

### 文件路径：`你的项目/.claude/async-hook.sh`

```bash
#!/bin/bash
set -euo pipefail

# ====================== 步骤1：读取 Claude 传入的输入数据 ======================
# Claude 会把工具执行的完整信息（工具名、参数、结果）通过 STDIN 传给脚本
HOOK_INPUT=$(cat)

# ====================== 步骤2：定义基础变量 ======================
LOG_FILE=".claude/hook-async.log"  # 日志存储路径
TIMESTAMP=$(date "+%Y-%m-%d %H:%M:%S")

# ====================== 步骤3：本地日志记录（持久化存储） ======================
echo "【${TIMESTAMP}】异步Hook执行 | 输入数据：${HOOK_INPUT}" >> "${LOG_FILE}"

# ====================== 步骤4：发送外部通知（企业微信/钉钉/Slack，可选） ======================
# 示例：企业微信机器人通知（替换你的WebHook地址即可使用）
WECHAT_WEBHOOK="https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=你的密钥"
# 解析工具名称和结果
TOOL_NAME=$(echo "${HOOK_INPUT}" | jq -r '.tool_name // "未知工具"')
TOOL_SUCCESS=$(echo "${HOOK_INPUT}" | jq -r '.success // false')

# 发送通知（后台执行，不阻塞脚本）
curl -s -X POST "${WECHAT_WEBHOOK}" \
  -H "Content-Type: application/json" \
  -d "{
    \"msgtype\": \"text\",
    \"text\": {
      \"content\": \"【Claude异步通知】\n时间：${TIMESTAMP}\n工具：${TOOL_NAME}\n状态：${TOOL_SUCCESS}\n\"
    }
  }" > /dev/null 2>&1 &

# ====================== 步骤5：构造标准输出 → 给大模型注入上下文 ======================
# 异步Hook输出规则：仅支持 additionalContext/systemMessage，下一轮生效
echo "${HOOK_INPUT}" | jq -n --arg ctx "【异步注入】${TIMESTAMP} | 工具 ${TOOL_NAME} 执行完成，状态：${TOOL_SUCCESS}" '
{
  # 给模型注入上下文（用户不可见，纯数据补充）
  "additionalContext": $ctx,
  # 固定必须输出的结构
  "hookSpecificOutput": {
    "hookEventName": "PostToolUse"
  }
}'
```

------

## 三、一键启用（必须执行）

给脚本添加**执行权限**，否则 Hook 无法运行：

```bash
chmod +x .claude/async-hook.sh
```

------

## 四、完整工作流程（异步机制）

1. **工具执行完成** → 触发 `PostToolUse`
2. Claude **后台启动脚本**，不等待执行完成，直接继续主流程
3. 脚本**后台执行**：写日志 + 发通知
4. 脚本执行完成 → 输出 `additionalContext`
5. **下一轮对话** → 模型自动读取注入的上下文

------

## 五、自定义修改指南

### 1. 修改监听的工具

```json
// 只监听 AskUserQuestion 和 Bash 工具
"matcher": "AskUserQuestion|Bash"
```

### 2. 关闭通知

直接删除脚本中 **步骤 4** 的所有代码即可

### 3. 修改注入的上下文

修改 `additionalContext` 对应的字符串内容

### 4. 切换为同步执行

删除配置中的 `"async": true` 即可

------

## 六、关键特性

✅ **完全异步**：不阻塞 Claude 主流程

✅ **标准输出**：符合官方格式，无报错

✅ **三合一功能**：日志 + 通知 + 上下文注入

✅ **无冲突**：异步 Hook 仅做副作用，不影响决策

✅ **可调试**：日志文件 `.claude/hook-async.log` 查看执行记录

------

## 七、调试方法

启动调试模式，查看异步 Hook 执行日志：

```bash
claude --debug
```







# Stop Hooks 可返回的信息

Stop hooks 作为同步 hook 的一种，可以返回以下信息：

## 通用字段
字段名 类型 描述 continue boolean (可选) Claude 是否应该在 hook 执行后继续（默认为 true） suppressOutput boolean (可选) 是否隐藏 stdout 从 transcript 中（默认为 false） stopReason string (可选) 当 continue 为 false 时显示的消息 decision 'approve' | 'block' (可选) 决策 reason string (可选) 决策的解释 systemMessage string (可选) 显示给用户的警告消息

## 特定场景字段
虽然 hookSpecificOutput 中没有为 Stop 事件定义特定结构，但 stop hooks 仍然可以返回以下与其他事件共享的字段：

字段名 类型 描述 additionalContext string (可选) 提供额外的上下文信息 updatedInput Record<string, unknown> (可选) 更新工具输入 updatedMCPToolOutput unknown (可选) 更新 MCP 工具的输出

## 示例输出
### 基本示例
```
{
  "continue": true,
  "systemMessage": "Task completed successfully"
}
```
### 阻止继续的示例
```
{
  "continue": false,
  "stopReason": "Task failed due to validation error",
  "reason": "The task did not pass all validation checks",
  "systemMessage": "Task validation failed"
}
```
### 提供额外上下文的示例
```
{
  "continue": true,
  "systemMessage": "Task completed",
  "hookSpecificOutput": {
    "hookEventName": "Stop",
    "additionalContext": "Task took 10 seconds to complete"
  }
}
```
## 处理机制
当 stop hook 返回这些信息时：

- continue: false 会阻止 Claude 继续执行
- stopReason 会显示给用户
- systemMessage 会被包装成系统提醒
- additionalContext 会被添加到对话上下文中
这些返回值允许 stop hooks 对任务执行结果进行总结、提供额外信息，甚至在必要时阻止 Claude 继续执行。