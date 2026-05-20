# SDK 中的 Agent Skills 使用指南

## 概述

Agent Skills 通过专业能力扩展 Claude，Claude 会在相关时自动调用这些能力。Skills 被打包为 `SKILL.md` 文件，包含说明、描述和可选的支持资源。

## Skills 如何与 SDK 配合使用

1. **定义为文件系统工件**：在特定目录（`.claude/skills/`）中创建为 `SKILL.md` 文件
2. **从文件系统加载**：Skills 从由 `settingSources`（TypeScript）或 `setting_sources`（Python）管理的文件系统位置加载
3. **自动发现**：加载文件系统设置后，在启动时从用户和项目目录发现 Skill 元数据；触发时加载完整内容
4. **由模型调用**：Claude 根据上下文自动选择何时使用它们
5. **通过 `skills` 选项过滤**：发现的 Skills 默认启用。传递 Skill 名称列表、`"all"` 或 `[]` 来控制会话中可用的 Skills

**与子代理的区别**：子代理可以通过编程方式定义，Skills 必须创建为文件系统工件，SDK 不提供用于注册 Skills 的编程 API。

## Skills 配置选项（TypeScript）

### 启用所有 Skills

```typescript
import { query } from "@anthropic-ai/claude-agent-sdk";

for await (const message of query({
  prompt: "Help me process this PDF document",
  options: {
    cwd: "/path/to/project",
    settingSources: ["user", "project"],
    skills: "all",
    allowedTools: ["Read", "Write", "Bash"],
  }
})) {
  console.log(message);
}
```

### 只启用特定 Skills

```typescript
const options = {
  settingSources: ["user", "project"],
  skills: ["pdf", "docx"],
};
```

### 禁用所有 Skills

```typescript
const options = {
  skills: [],
};
```

## Skills 配置选项（Python）

### 启用所有 Skills

```python
from claude_agent_sdk import query, ClaudeAgentOptions

options = ClaudeAgentOptions(
    cwd="/path/to/project",
    setting_sources=["user", "project"],
    skills="all",
    allowed_tools=["Read", "Write", "Bash"],
)

async for message in query(
    prompt="Help me process this PDF document",
    options=options
):
    print(message)
```

### 只启用特定 Skills

```python
options = ClaudeAgentOptions(
    setting_sources=["user", "project"],
    skills=["pdf", "docx"],
)
```

### 禁用所有 Skills

```python
options = ClaudeAgentOptions(
    skills=[],
)
```

## Skills 的加载位置

| 来源 | 路径 | 说明 |
|------|------|------|
| **项目 Skills** | `.claude/skills/` | 通过 git 共享给团队 |
| **用户 Skills** | `~/.claude/skills/` | 个人 Skills |
| **插件 Skills** | 与插件捆绑 | - |

## 默认行为

**不设置 `skills` 字段时，默认启用所有发现的 Skills**，与 Claude Code CLI 行为匹配。

## 重要限制

### 1. 不支持会话中途修改 Skills

`s` 是 `query()` 调用的配置参数，**不支持在会话中途动态增加或减少 Skills**。

| 操作 | 支持情况 |
|------|---------|
| 会话中途增加 Skills | ❌ 不支持 |
| 会话中途减少 Skills | ❌ 不支持 |
| Skills 变化自动通知模型 | ❌ 不支持 |
| 通过新 query() 改变 Skills | ✅ 支持 |

### 2. SKILL.md 中的 allowed-tools 在 SDK 中不适用

`SKILL.md` 中的 `allowed-tools` frontmatter 字段仅在直接使用 Claude Code CLI 时受支持。**通过 SDK 使用 Skills 时，通过查询配置中的主 `allowedTools` 选项控制工具访问**。

```typescript
// SDK 中使用 allowedTools 控制工具权限
const options = {
  settingSources: ["user", "project"],
  skills: "all",
  allowedTools: ["Read", "Grep", "Glob"],
};
```

### 3. skills 选项是过滤器，不是沙箱

未列出的 Skills 对模型隐藏，并被 Skill 工具拒绝，但它们的文件仍在磁盘上，可通过 Read 和 Bash 访问。

## 替代方案

### 方案 1：新会话 + 不同 Skills 配置

```typescript
// 会话1：使用特定 Skills
for await (const msg of query({
  prompt: "任务A",
  options: { skills: ["pdf", "docx"] }
})) {}

// 后续任务：使用不同的 Skills
for await (const msg of query({
  prompt: "任务B",
  options: { skills: ["image-processing"] }
})) {}
```

### 方案 2：通过 Hook 注入提示

```typescript
const options = {
  skills: "all",
  hooks: {
    UserPromptSubmit: [{
      hooks: [{
        type: "prompt",
        prompt: "当前只使用以下 Skills: pdf, docx"
      }]
    }]
  }
};
```

### 方案 3：使用子代理处理不同任务

```typescript
const options = {
  agents: {
    "pdf-processor": {
      prompt: "专门处理 PDF",
      tools: ["Read", "Bash"],
    },
    "image-processor": {
      prompt: "专门处理图片",
      tools: ["Read", "Bash"],
    }
  }
};
```

## 创建 Skills

Skills 定义为包含带有 YAML frontmatter 和 Markdown 内容的 `SKILL.md` 文件的目录。`description` 字段确定 Claude 何时调用您的 Skill。

### 目录结构

```
.claude/skills/processing-pdfs/
└── SKILL.md
```

### SKILL.md 示例

```markdown
---
name: pdf-processor
description: 当用户需要处理 PDF 文件时使用此技能，包括提取文本、合并、拆分等操作
allowed-tools:
  - Read
  - Bash
---

# PDF 处理技能

此技能帮助用户处理 PDF 文件...

## 常用操作

- 提取文本内容
- 合并多个 PDF
- 拆分 PDF
- 转换为其他格式
```

## 工具限制

SKILL.md 中的 `allowed-tools` frontmatter 字段在 SDK 中**不生效**，工具权限统一通过主 `allowedTools` 选项控制。

## 故障排除

### 找不到 Skills

1. **检查 settingSources 配置**：确保包含 `"user"` 和/或 `"project"`

```typescript
// ✅ Skills 会加载
const options = {
  settingSources: ["user", "project"],
  skills: "all",
};

// ❌ Skills 不会加载
const options = {
  settingSources: [],
  skills: "all",
};
```

2. **检查工作目录**：SDK 相对于 `cwd` 选项加载 Skills

```typescript
const options = {
  cwd: "/path/to/project", // 必须包含 .claude/skills/
  settingSources: ["user", "project"],
  skills: "all",
};
```

3. **验证文件系统位置**

```bash
# 检查项目 Skills
ls .claude/skills/*/SKILL.md

# 检查个人 Skills
ls ~/.claude/skills/*/SKILL.md
```

### Skill 未被使用

1. **检查 `skills` 选项**：确认 Skill 名称已包含
2. **检查描述**：确保 `description` 具体且包含相关关键字

## 相关文档

- [Claude Code 中的 Agent Skills](https://code.claude.com/docs/zh-CN/skills)
- [Agent Skills 概述](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [Agent Skills 最佳实践](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [SDK 中的子代理](https://code.claude.com/docs/zh-CN/agent-sdk/subagents)
- [SDK 中的 Slash Commands](https://code.claude.com/docs/zh-CN/agent-sdk/slash-commands)
