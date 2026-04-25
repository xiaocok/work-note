---
name: hook-execution-analysis
description: 分析 Claude Code 中多个相同类型 hook 注册后的执行机制，包括注册、匹配、并发执行和结果聚合
type: reference
originSessionId: 1317ca55-5dd8-4997-917e-a189ee21f859
---
# Hook 的多注册执行机制分析

## 回答摘要

**多个相同类型的 hook 是并发执行的，按注册先后全部启动，然后通过 `Promise.race` 轮询逐个收集结果。它们会在特定场景下相互干扰（如 permission 决策冲突），但大部分场景下是隔离的。**

---

## 关键文件

| 文件 | 角色 |
|------|------|
| `utils/hooks.ts` | 核心 hook 执行逻辑 |
| `utils/generators.ts` | `all()` 函数实现并发 generator 调度 |
| `bootstrap/state.ts` | 全局 hook 注册中心 `registerHookCallbacks()` |
| `utils/hooks/sessionHooks.ts` | 会话级别 hook 管理 |
| `utils/hooks/hooksSettings.ts` | 从 settings 文件收集所有 hooks |
| `utils/hooks/AsyncHookRegistry.ts` | 异步后台 hook 进程管理 |
| `utils/hooks/hooksConfigManager.ts` | 按事件和 matcher 分组 |

## 注册流程

多个相同类型的 hook 来自**四个来源**，在 `getHooksConfig()` 中合并为一个数组：

1. **settings 文件** — 从 `userSettings`、`projectSettings`、`localSettings` 读取
2. **全局注册 hooks** — 存在 `STATE.registeredHooks` 中，通过 `registerHookCallbacks()` 追加
3. **会话 hooks** — 存在 `appState.sessionHooks` 中，agent/skill 级别
4. **函数 hooks** — 内存中的 TS 回调（如结构化输出强制）

去重策略：同源（同一 plugin/skill）内，相同的 `command`/`prompt`/`url` 会被去重。

## 执行模型：并发

### REPL 上下文（`executeHooks()`）

```typescript
// 第 2143 行：将所有匹配的 hook 转为 async generator 数组
const hookPromises = matchingHooks.map(async function* ({ hook, ... }, hookIndex) {
  // 每个 hook 各自创建一个独立的 AbortSignal（含独立超时）
  // 然后执行对应的 hook 类型
})

// 第 2744 行：通过 all() 函数并发消费这些 generator
for await (const result of all(hookPromises)) {
  // 收集、聚合结果
}
```

`all()` 函数（`generators.ts`）的实现：
- 所有 generator **同时启动**
- 使用 `Promise.race()` 轮询哪个先 yield 出值
- 默认**无并发上限**（`concurrencyCap = Infinity`）
- 先完成的 hook 先被消费，不保证顺序

### REPL 外上下文（`executeHooksOutsideREPL()`）

```typescript
const hookPromises = matchingHooks.map(async (hook, hookIndex) => { ... })
return await Promise.all(hookPromises)
```

同样是**并行执行**，使用 `Promise.all`。

## 独立超时控制

每个 hook 有独立的超时（第 2195-2200 行）：
- 每个 hook 可配置自己的 `timeout`
- 默认超时：`TOOL_HOOK_EXECUTION_TIMEOUT_MS`（15 秒）
- 通过 `createCombinedAbortSignal` 创建独立信号
- **一个 hook 超时不会影响其他 hooks**

## 相互干扰分析

### 不会干扰的方面
| 方面 | 原因 |
|------|------|
| **执行超时** | 每个 hook 独立 AbortSignal |
| **进程隔离** | command hooks 各自启动独立子进程 |
| **JSON 输出解析** | 每个 hook 的 stdout 独立解析 |
| **后台异步** | `AsyncHookRegistry` 按 `processId` 独立管理 |
| **错误** | 单个 hook 异常被 `all()` 的 `Promise.race` 隔离 |

### 可能干扰的方面
| 方面 | 行为 |
|------|------|
| **`permissionBehavior`** | `deny > ask > allow` 优先级规则，多个 hook 返回不同值会冲突 |
| **`preventContinuation`** | 任一 hook 设置即可阻止，其他 hook 的设置会被覆盖 |
| **`updatedInput`** | 仅最后一轮生效 |
| **`blockingError`** | 全部收集，多个 blocking errors 都会传递 |
| **`additionalContexts`** | 全部合并，不会丢失 |
| **`watchPaths`** | 全部合并 |
| **`initialUserMessage`** | 取最后一个设置的 |

### 关键结论
- **不保证执行完成顺序**，取决于 hook 自身的执行速度
- **不保证结果优先级**（permission 除外，它有硬编码优先级规则）
- **进程是隔离的**，但**逻辑结果可能冲突**
- 如果一个 hook 返回 `block`，另一个返回 `allow`，则 **deny 胜出**

## 错误隔离

`executeHooks()` 中使用 `all()` 函数：
- `all()` 内部用 `Promise.race`，单次迭代的拒绝不会传播
- 但注意：如果一个 hook 在其 generator 中抛出了未捕获的异常，`all()` 的迭代会失败

`executeHooksOutsideREPL()` 使用 `Promise.all`：
- 但 `matchingHooks.map` 的回调内部都有 `try-catch`，不会 reject
- 所以实际上不会级联失败
