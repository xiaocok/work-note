# Claude Code 技能系统消息机制分析

本文档分析了 Claude Code 中技能（Skill）系统如何与大模型交互，包括消息发送机制、触发条件和配置方式。

## 目录

1. [技能消息发送机制概述](#1-技能消息发送机制概述)
2. [skill_listing 消息](#2-skill_listing-消息)
3. [skill_discovery 消息](#3-skill_discovery-消息)
4. [消息类型与结构](#4-消息类型与结构)
5. [增量发送机制](#5-增量发送机制)
6. [功能开关配置](#6-功能开关配置)
7. [总结](#7-总结)

---

## 1. 技能消息发送机制概述

### 1.1 核心流程

当用户发送消息给大模型时，技能信息通过 **attachment** 机制附加到消息中，最终转换为 `<system_reminder>` 格式的用户消息发送给大模型。

### 1.2 消息发送流程

```
用户输入 → processUserInput → getAttachmentMessages → skill_listing/skill_discovery attachment
    ↓
normalizeAttachmentForAPI → createUserMessage + wrapMessagesInSystemReminder
    ↓
mergeUserMessagesAndToolResults → 合并到用户消息
    ↓
发送给大模型
```

### 1.3 关键代码位置

| 功能 | 文件位置 |
|------|---------|
| Attachment 定义 | `src/utils/attachments.ts#L532-L542` |
| 消息转换 | `src/utils/messages.ts#L3728-L3737` |
| 消息合并 | `src/utils/messages.ts#L2372-L2387` |
| API 格式化 | `src/utils/messages.ts#L1989-L2056` |

---

## 2. skill_listing 消息

### 2.1 定义

```typescript
// attachments.ts
{
  type: 'skill_listing',
  content: string,        // 格式化后的技能列表内容
  skillCount: number,     // 技能数量
  isInitial: boolean      // 是否为初始批次
}
```

### 2.2 触发条件

| 场景 | 触发时机 |
|------|---------|
| **首次对话** | 用户首次输入时发送所有可用技能 |
| **技能变更** | 检测到技能文件变化时发送新增技能 |
| **子代理启动** | 每个子代理独立发送技能列表 |

### 2.3 变更检测机制

通过 **chokidar 文件监视器** 实时监控技能目录变化：

```typescript
// skillChangeDetector.ts
watcher.on('add', handleChange)
watcher.on('change', handleChange)
watcher.on('unlink', handleChange)
```

变更后调用 `resetSentSkillNames()` 清空已发送记录，下次消息发送时会发送所有未在 `sentSkillNames` 中的技能。

### 2.4 发送给大模型的格式

```json
{
  "type": "user",
  "message": {
    "content": "<system_reminder>\nThe following skills are available for use with the Skill tool:\n\n- web-dev: Create production-grade web interfaces...\n- pdf: Work with PDF files...\n</system_reminder>"
  }
}
```

### 2.5 渐进式披露实现

```typescript
// attachments.ts
const sentSkillNames = new Map<string, Set<string>>()

// 查找未发送的技能
const newSkills = allCommands.filter(cmd => !sent.has(cmd.name))

// 标记为已发送
for (const cmd of newSkills) {
  sent.add(cmd.name)
}
```

---

## 3. skill_discovery 消息

### 3.1 定义

```typescript
// attachments.ts
{
  type: 'skill_discovery',
  skills: { name: string; description: string; shortId?: string }[],
  signal: DiscoverySignal,
  source: 'native' | 'aki' | 'both'
}
```

### 3.2 触发条件

`skill_discovery` 是**实验性功能**，需要 `EXPERIMENTAL_SKILL_SEARCH` feature flag 开启。

| 触发场景 | 位置 | 说明 |
|---------|------|------|
| **Turn 0** | `getAttachmentMessages` | 用户首次输入时，基于输入调用 AI 知识库发现相关技能 |
| **写入枢轴检测** | `query.ts` 循环中 | 检测到用户写入文件时触发 |

### 3.3 触发代码

```typescript
// attachments.ts
...(feature('EXPERIMENTAL_SKILL_SEARCH') &&
skillSearchModules &&
!options?.skipSkillDiscovery
  ? [
      maybe('skill_discovery', () =>
        skillSearchModules.prefetch.getTurnZeroSkillDiscovery(
          input,
          messages ?? [],
          context,
        ),
      ),
    ]
  : [])
```

### 3.4 发送给大模型的格式

```json
{
  "type": "user",
  "message": {
    "content": "<system_reminder>\nSkills relevant to your task:\n\n- skill1: description1\n- skill2: description2\n\nThese skills encode project-specific conventions. Invoke via Skill(\"<name>\") for complete instructions.\n</system_reminder>"
  }
}
```

### 3.5 与 skill_listing 的区别

| 特性 | skill_listing | skill_discovery |
|------|---------------|-----------------|
| **触发时机** | 首次对话 + 技能变更时 | Turn 0 + 写入枢轴检测 |
| **内容** | 所有可用技能 | 与当前任务相关的技能 |
| **来源** | 本地加载的技能 | AI 知识库推荐 |
| **范围** | 广泛 | 精准推荐 |

---

## 4. 消息类型与结构

### 4.1 发送给大模型的消息类型

`normalizeMessagesForAPI` 函数返回 `(UserMessage | AssistantMessage)[]` 数组。

### 4.2 UserMessage 内容类型

所有内容都放在 `message.content` 字段中：

| 内容类型 | 格式 | 说明 |
|---------|------|------|
| 用户原始输入 | `{ type: 'text', text: '...' }` | 用户的问题 |
| 用户输入 + 图片 | `[ { type: 'text' }, { type: 'image' } ]` | 包含图片的消息 |
| 工具调用结果 | `{ type: 'tool_result', tool_use_id: '...', content: '...' }` | 工具执行返回 |
| system_reminder | `{ type: 'text', text: '<system_reminder>...</system_reminder>' }` | 各种系统提醒 |

### 4.3 AssistantMessage 内容类型

| 内容类型 | 格式 |
|---------|------|
| 文本回复 | `{ type: 'text', text: '...' }` |
| 思考过程 | `{ type: 'thinking', thinking: '...' }` |
| 工具调用 | `{ type: 'tool_use', id: '...', name: '...', input: {...} }` |

### 4.4 所有 attachment 类型

| 附件类型 | system_reminder 内容 |
|---------|---------------------|
| **skill_listing** | 可用技能列表 |
| **skill_discovery** | 任务相关的技能推荐 |
| **invoked_skills** | 本会话已调用的技能内容 |
| **opened_file_in_ide** | 用户在 IDE 中打开的文件 |
| **selected_lines_in_ide** | 用户选中的代码行 |
| **edited_text_file** | 被修改的文件变更 |
| **directory** | 目录内容 |
| **file** | 文件内容 |
| **todo_reminder** | Todo 列表提醒 |
| **task_reminder** | 任务提醒 |
| **nested_memory** | 嵌套记忆内容 |
| **relevant_memories** | 相关记忆 |
| **plan_file_reference** | 计划文件引用 |
| **queued_command** | 排队的命令 |
| **team_context** | 团队协作上下文 |
| **teammate_mailbox** | 队友邮件 |

---

## 5. 增量发送机制

### 5.1 消息数组顺序

messages 数组中**序号靠前的是越早的消息**：

- 索引 0：最早的消息
- 索引 n-1：最近的消息

### 5.2 消息合并逻辑

当处理 attachment 消息时，会与相邻的用户消息合并：

```typescript
// messages.ts
const lastMessage = last(result)
if (lastMessage?.type === 'user') {
  result[result.length - 1] = attachmentMessage.reduce(
    (p, c) => mergeUserMessagesAndToolResults(p, c),
    lastMessage,
  )
  return
}
```

### 5.3 初始化 vs 增量发送

**两种情况都和用户消息一起发送，合并为同一个 user 消息**：

#### 初始化发送（isInitial=true）

```json
{
  "type": "user",
  "message": {
    "content": [
      { "type": "text", "text": "用户问题" },
      { 
        "type": "text", 
        "text": "<system_reminder>\nThe following skills are available...\n</system_reminder>"
      }
    ]
  }
}
```

#### 增量发送（isInitial=false）

结构相同，只是 content 中包含的是新增/变化后的技能列表。

### 5.4 增量标识

- `isInitial: true` - 初始/重置后的全量
- `isInitial: false` - 增量

**没有特殊的 system_reminder 标识符**，只是包含新增技能的文本。

---

## 6. 功能开关配置

### 6.1 feature() 函数

`feature()` 函数来自 `bun:bundle` 模块，检查 GrowthBook 功能开关：

```typescript
const skillSearchModules = feature('EXPERIMENTAL_SKILL_SEARCH')
  ? { ... }
  : null
```

### 6.2 配置方式

#### 环境变量方式（仅限 ant 内部用户）

```bash
export USER_TYPE=ant
export CLAUDE_INTERNAL_FC_OVERRIDES='{"EXPERIMENTAL_SKILL_SEARCH": true}'
```

#### 配置文件方式（仅限 ant 内部用户）

在 `~/.claude/settings.json` 中添加：

```json
{
  "growthBookOverrides": {
    "EXPERIMENTAL_SKILL_SEARCH": true
  }
}
```

### 6.3 优先级

| 优先级 | 来源 | 说明 |
|-------|------|------|
| 1 | 环境变量 `CLAUDE_INTERNAL_FC_OVERRIDES` | 最高优先级 |
| 2 | 配置文件 `growthBookOverrides` | ant 用户可通过 /config 设置 |
| 3 | 服务端 GrowthBook 值 | 默认值 |

### 6.4 重要限制

**这些覆盖方式都需要 `USER_TYPE === 'ant'`**，即只有 Anthropic 内部用户才能使用：

```typescript
if (process.env.USER_TYPE !== 'ant') return undefined
```

---

## 7. 总结

### 7.1 核心结论

1. **技能信息放在 user 消息的 `content` 字段中**，通过 `<system_reminder>` 机制发送
2. **skill_listing 和 skill_discovery 都与用户消息合并发送**，不是单独发送
3. **增量发送通过 `isInitial` 字段标识**，没有特殊的标识符
4. **消息数组按时间顺序排列**，索引越小消息越早
5. **EXPERIMENTAL_SKILL_SEARCH 功能由 GrowthBook 服务端控制**，普通用户无法直接启用

### 7.2 关键代码位置汇总

| 功能 | 文件位置 |
|------|---------|
| Attachment 定义 | `src/utils/attachments.ts#L532-L542` |
| skill_listing 生成 | `src/utils/attachments.ts#L2661-L2751` |
| skill_discovery 触发 | `src/utils/attachments.ts#L801-L813` |
| 消息转换 | `src/utils/messages.ts#L3728-L3737` |
| 消息合并 | `src/utils/messages.ts#L2372-L2387` |
| API 格式化 | `src/utils/messages.ts#L1989-L2056` |
| 技能变更检测 | `src/utils/skills/skillChangeDetector.ts#L237-L279` |
| GrowthBook 配置 | `src/services/analytics/growthbook.ts#L170-L220` |

### 7.3 流程图

```
┌─────────────────────────────────────────────────────────────────┐
│                        用户输入                                  │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    processUserInput                              │
│  - 创建用户消息                                                   │
│  - 获取 attachmentMessages (skill_listing, skill_discovery)     │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                 normalizeMessagesForAPI                          │
│  - 遍历消息数组                                                   │
│  - 调用 normalizeAttachmentForAPI 转换 attachment                │
│  - 合并相邻的用户消息                                              │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                   发送给大模型                                    │
│  - 历史消息 + 用户问题 + 技能列表                                  │
│  - 技能信息在 <system_reminder> 标签中                            │
└─────────────────────────────────────────────────────────────────┘
```
