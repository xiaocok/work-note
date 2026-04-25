# Claude Code Hook 多注册执行机制分析

## 核心结论

**多个相同类型的 hook 是并发执行的，全部同时启动，通过 `Promise.race` 轮询逐个收集结果。它们在进程/超时层面互不干扰，但在逻辑结果（如 permission 决策）上可能存在冲突。**

---

## 1. 注册来源

Hook 从四个来源汇聚，在 `getHooksConfig()` 中合并为一个数组：

| 来源 | 存储位置 | 生命周期 |
|------|----------|----------|
| **settings 文件** | `userSettings` / `projectSettings` / `localSettings` | 持久化 |
| **全局注册 hook** | `STATE.registeredHooks` (src/bootstrap/state.ts) | 进程级别 |
| **会话 hook** | `appState.sessionHooks` (Map) | Agent/Skill 生命周期 |
| **函数 hook** | `appState.sessionHooks` (FunctionHook) | 会话级别，内存回调 |

去重策略：同源（同一 plugin/skill）内，相同的 `command`/`prompt`/`url` 会去重，保留最后一个。

---

## 2. 执行模型

### 2.1 REPL 上下文 (`executeHooks()`)

```typescript
// src/utils/hooks.ts:2143 — 所有匹配 hook 转为 async generator 数组
const hookPromises = matchingHooks.map(async function* ({ hook, ... }, hookIndex) {
  // 每个 hook 有独立的 AbortSignal + 独立的超时
  // 然后执行对应类型（command/prompt/agent/http/callback/function）
})

// src/utils/hooks.ts:2744 — 通过 all() 并发消费
for await (const result of all(hookPromises)) {
  // 按完成顺序逐个收集结果
}
```

`all()` 函数 (`src/utils/generators.ts`) ：
- 并发启动所有 generator，无上限（`concurrencyCap = Infinity`）
- 通过 `Promise.race()` 轮询哪个先 yield
- 先完成的 hook 先被消费，**不保证顺序**

### 2.2 REPL 外上下文 (`executeHooksOutsideREPL()`)

```typescript
// 同样是并行执行
const hookPromises = matchingHooks.map(async (hook, hookIndex) => { ... })
return await Promise.all(hookPromises)
```

---

## 3. 超时控制

每个 hook 有独立的超时（`hooks.ts:2195`）：

```typescript
const commandTimeoutMs = hook.timeout ? hook.timeout * 1000 : timeoutMs
const { signal: abortSignal, cleanup } = createCombinedAbortSignal(signal, {
  timeoutMs: commandTimeoutMs,
})
```

- 可通过配置设置 `timeout`（秒）
- 默认超时：`TOOL_HOOK_EXECUTION_TIMEOUT_MS`（15秒）
- **一个 hook 超时完全不影响其他 hook**

---

## 4. 结果聚合与干扰分析

### 4.1 不会相互干扰的方面

| 方面 | 原因 |
|------|------|
| **执行超时** | 每个 hook 有独立的 AbortSignal |
| **进程隔离** | command hooks 各自启动独立子进程 |
| **JSON 输出解析** | 每个 hook 的 stdout 独立解析 |
| **后台异步** | `AsyncHookRegistry` 按 `processId` 独立管理 |
| **执行错误** | 单个 hook 异常被 `all()` 的 `Promise.race` 隔离 |

### 4.2 可能相互干扰的方面

| 方面 | 行为 | 风险等级 |
|------|------|----------|
| **`permissionBehavior`** | `deny > ask > allow` 优先级规则，多个 hook 返回不同值会冲突 | 高 |
| **`preventContinuation`** | 任一 hook 设置即生效，其他被覆盖 | 中 |
| **`updatedInput`** | 仅最后一轮生效 | 中 |
| **`blockingError`** | 全部收集，多个 blocking errors 都会传递 | 低 |
| **`additionalContexts`** | 全部合并，不丢失 | 低 |
| **`watchPaths`** | 全部合并 | 低 |
| **`initialUserMessage`** | 取最后一个设置的 | 低 |

### 4.3 permissionBehavior 优先级规则（hooks.ts:2821-2847）

```typescript
case 'deny':  // deny 总是优先
  permissionBehavior = 'deny'; break;
case 'ask':   // 仅当未设置 deny
  if (permissionBehavior !== 'deny') permissionBehavior = 'ask'; break;
case 'allow': // 仅当未设置其他
  if (!permissionBehavior) permissionBehavior = 'allow'; break;
```

---

## 5. 执行流程

```
executeHooks() 被调用
  |
  v
getMatchingHooks() -- 从 4 个来源合并匹配的 hooks
  |
  v
去重（同源内相同命令保留一个）
  |
  v
安全检查（workspace trust、managed-only、bare mode）
  |
  v
为每个 hook 创建独立的 async generator
  |
  v
all() 函数 -- 所有 generator 并发启动
  |
  +-- Hook A -- 独立子进程 -- 解析 JSON
  +-- Hook B -- 独立子进程 -- 解析 JSON
  +-- Hook C -- 独立子进程 -- 解析 JSON
  |
  v
按完成顺序逐个 yield 结果到主循环
  |
  v
主循环按优先级规则聚合：
  - preventContinuation 任一即生效
  - permissionBehavior 按 deny > ask > allow 取最高
  - blockingError 全部收集
  - additionalContexts 全部合并
  |
  v
最终 yield AggregatedHookResult
```

---

## 6. 关键风险点

1. **多个 hook 对 permission 的决策可能冲突**：一个 hook 返回 deny，另一个返回 allow，最终 deny 胜出。用户应确保多个 hook 的语义一致。

2. **不保证执行顺序**：执行速度和完成顺序均不可预测，对顺序有依赖的逻辑可能出错。

3. **`executeHooksOutsideREPL()` 使用 `Promise.all`**：虽然回调内部都有 try-catch 不会 reject，但如果未来引入未捕获异常，`Promise.all` 会级联失败。

4. **后台异步 hook（AsyncHookRegistry）**：通过全局 Map 管理，多个 hook 可能同时影响同一个会话状态。

---

## 7. 关键文件索引

| 文件 | 关键行 | 作用 |
|------|--------|------|
| `src/utils/hooks.ts` | 1952 | `executeHooks()` 核心执行入口 |
| `src/utils/hooks.ts` | 2143 | `matchingHooks.map()` 并发启动所有 hook |
| `src/utils/hooks.ts` | 2744 | `all(hookPromises)` 并发消费结果 |
| `src/utils/hooks.ts` | 2821 | `permissionBehavior` 优先级聚合 |
| `src/utils/hooks.ts` | 1603 | `getMatchingHooks()` 匹配和去重逻辑 |
| `src/utils/generators.ts` | 32 | `all()` 并发 generator 调度器 |
| `src/bootstrap/state.ts` | 1419 | `registerHookCallbacks()` 全局注册 |
| `src/utils/hooks/sessionHooks.ts` | 68 | `addSessionHook()` 会话级注册 |
| `src/utils/hooks/hooksSettings.ts` | 92 | `getAllHooks()` settings 来源收集 |
| `src/utils/hooks/AsyncHookRegistry.ts` | 30 | 后台异步 hook 管理 |
