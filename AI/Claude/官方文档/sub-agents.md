> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 创建自定义 subagents

> 在 Claude Code 中创建和使用专门的 AI subagents，用于特定任务的工作流和改进的上下文管理。

Subagents 是处理特定类型任务的专门 AI 助手。每个 subagent 在自己的 context window 中运行，具有自定义系统提示、特定的工具访问权限和独立的权限。当 Claude 遇到与 subagent 描述相匹配的任务时，它会委托给该 subagent，该 subagent 独立工作并返回结果。

<Note>
  如果您需要多个代理并行工作并相互通信，请参阅 [agent teams](/zh-CN/agent-teams)。Subagents 在单个会话中工作；agent teams 跨多个独立会话进行协调。
</Note>

Subagents 帮助您：

* **保留上下文**，通过将探索和实现保持在主对话之外
* **强制执行约束**，通过限制 subagent 可以使用的工具
* **跨项目重用配置**，使用用户级 subagents
* **专门化行为**，为特定领域使用专注的系统提示
* **控制成本**，通过将任务路由到更快、更便宜的模型（如 Haiku）

Claude 使用每个 subagent 的描述来决定何时委托任务。创建 subagent 时，请编写清晰的描述，以便 Claude 知道何时使用它。

Claude Code 包括几个内置 subagents，如 **Explore**、**Plan** 和 **general-purpose**。您也可以创建自定义 subagents 来处理特定任务。本页涵盖 [内置 subagents](#built-in-subagents)、[如何创建您自己的](#quickstart-create-your-first-subagent)、[完整配置选项](#configure-subagents)、[使用 subagents 的模式](#work-with-subagents) 和 [示例 subagents](#example-subagents)。

## 内置 subagents

Claude Code 包括内置 subagents，Claude 在适当时自动使用。每个都继承父对话的权限，并具有额外的工具限制。

<Tabs>
  <Tab title="Explore">
    一个快速的、只读的代理，针对搜索和分析代码库进行了优化。

    * **模型**：Haiku（快速、低延迟）
    * **工具**：只读工具（拒绝访问 Write 和 Edit 工具）
    * **目的**：文件发现、代码搜索、代码库探索

    当 Claude 需要搜索或理解代码库而不进行更改时，它会委托给 Explore。这使探索结果保持在主对话上下文之外。

    调用 Explore 时，Claude 指定一个彻底程度级别：**quick** 用于有针对性的查找，**medium** 用于平衡的探索，或 **very thorough** 用于全面分析。
  </Tab>

  <Tab title="Plan">
    在 [plan mode](/zh-CN/common-workflows#use-plan-mode-for-safe-code-analysis) 期间用于在呈现计划之前收集上下文的研究代理。

    * **模型**：从主对话继承
    * **工具**：只读工具（拒绝访问 Write 和 Edit 工具）
    * **目的**：用于规划的代码库研究

    当您处于 plan mode 且 Claude 需要理解您的代码库时，它会将研究委托给 Plan subagent。这防止了无限嵌套（subagents 无法生成其他 subagents），同时仍然收集必要的上下文。
  </Tab>

  <Tab title="General-purpose">
    一个能够处理需要探索和操作的复杂多步骤任务的代理。

    * **模型**：从主对话继承
    * **工具**：所有工具
    * **目的**：复杂研究、多步骤操作、代码修改

    当任务需要探索和修改、复杂推理来解释结果或多个依赖步骤时，Claude 会委托给 general-purpose。
  </Tab>

  <Tab title="Other">
    Claude Code 包括用于特定任务的其他辅助代理。这些通常是自动调用的，因此您不需要直接使用它们。

    | 代理                | 模型     | Claude 何时使用它                 |
    | :---------------- | :----- | :--------------------------- |
    | Bash              | 继承     | 在单独的上下文中运行终端命令               |
    | statusline-setup  | Sonnet | 当您运行 `/statusline` 来配置您的状态行时 |
    | Claude Code Guide | Haiku  | 当您提出关于 Claude Code 功能的问题时    |
  </Tab>
</Tabs>

除了这些内置 subagents，您可以创建自己的，具有自定义提示、工具限制、权限模式、hooks 和 skills。以下部分展示了如何开始和自定义 subagents。

## 快速入门：创建您的第一个 subagent

Subagents 在带有 YAML frontmatter 的 Markdown 文件中定义。您可以 [手动创建它们](#write-subagent-files) 或使用 `/agents` 命令。

本演练指导您通过 `/agent` 命令创建用户级 subagent。该 subagent 审查代码并为代码库建议改进。

<Steps>
  <Step title="打开 subagents 界面">
    在 Claude Code 中，运行：

    ```
    /agents
    ```
  </Step>

  <Step title="创建新的用户级代理">
    选择 **Create new agent**，然后选择 **User-level**。这会将 subagent 保存到 `~/.claude/agents/`，以便在所有项目中可用。
  </Step>

  <Step title="使用 Claude 生成">
    选择 **Generate with Claude**。出现提示时，描述 subagent：

    ```
    A code improvement agent that scans files and suggests improvements
    for readability, performance, and best practices. It should explain
    each issue, show the current code, and provide an improved version.
    ```

    Claude 生成系统提示和配置。按 `e` 在编辑器中打开它，如果您想自定义它。
  </Step>

  <Step title="选择工具">
    对于只读审查者，取消选择除 **Read-only tools** 之外的所有内容。如果您保持所有工具被选中，subagent 会继承主对话可用的所有工具。
  </Step>

  <Step title="选择模型">
    选择 subagent 使用的模型。对于此示例代理，选择 **Sonnet**，它在分析代码模式时平衡了能力和速度。
  </Step>

  <Step title="选择颜色">
    为 subagent 选择背景颜色。这有助于您在 UI 中识别哪个 subagent 正在运行。
  </Step>

  <Step title="保存并尝试">
    保存 subagent。它立即可用（无需重启）。尝试它：

    ```
    Use the code-improver agent to suggest improvements in this project
    ```

    Claude 委托给您的新 subagent，它扫描代码库并返回改进建议。
  </Step>
</Steps>

现在您有了一个 subagent，可以在机器上的任何项目中使用它来分析代码库并建议改进。

您也可以手动创建 subagents 作为 Markdown 文件、通过 CLI 标志定义它们，或通过插件分发它们。以下部分涵盖所有配置选项。

## 配置 subagents

### 使用 /agents 命令

`/agents` 命令提供了一个交互式界面来管理 subagents。运行 `/agents` 来：

* 查看所有可用的 subagents（内置、用户、项目和插件）
* 使用引导式设置或 Claude 生成创建新 subagents
* 编辑现有 subagent 配置和工具访问
* 删除自定义 subagents
* 查看当存在重复项时哪些 subagents 是活跃的

这是创建和管理 subagents 的推荐方式。对于手动创建或自动化，您也可以直接添加 subagent 文件。

要从命令行列出所有配置的 subagents 而不启动交互式会话，请运行 `claude agents`。这显示按来源分组的代理，并指示哪些被更高优先级的定义覆盖。

### 选择 subagent 范围

Subagents 是带有 YAML frontmatter 的 Markdown 文件。根据范围将它们存储在不同的位置。当多个 subagents 共享相同的名称时，更高优先级的位置获胜。

| 位置                  | 范围      | 优先级   | 如何创建                           |
| :------------------ | :------ | :---- | :----------------------------- |
| `--agents` CLI 标志   | 当前会话    | 1（最高） | 启动 Claude Code 时传递 JSON        |
| `.claude/agents/`   | 当前项目    | 2     | 交互式或手动                         |
| `~/.claude/agents/` | 所有项目    | 3     | 交互式或手动                         |
| 插件的 `agents/` 目录    | 启用插件的位置 | 4（最低） | 随 [plugins](/zh-CN/plugins) 安装 |

**项目 subagents**（`.claude/agents/`）非常适合特定于代码库的 subagents。将它们检入版本控制，以便您的团队可以协作使用和改进它们。

**用户 subagents**（`~/.claude/agents/`）是在所有项目中可用的个人 subagents。

**CLI 定义的 subagents** 在启动 Claude Code 时作为 JSON 传递。它们仅存在于该会话中，不会保存到磁盘，使其对快速测试或自动化脚本很有用：

```bash  theme={null}
claude --agents '{
  "code-reviewer": {
    "description": "Expert code reviewer. Use proactively after code changes.",
    "prompt": "You are a senior code reviewer. Focus on code quality, security, and best practices.",
    "tools": ["Read", "Grep", "Glob", "Bash"],
    "model": "sonnet"
  }
}'
```

`--agents` 标志接受与基于文件的 subagents 相同的 [frontmatter](#supported-frontmatter-fields) 字段的 JSON：`description`、`prompt`、`tools`、`disallowedTools`、`model`、`permissionMode`、`mcpServers`、`hooks`、`maxTurns`、`skills` 和 `memory`。对系统提示使用 `prompt`，等同于基于文件的 subagents 中的 markdown 正文。有关完整的 JSON 格式，请参阅 [CLI 参考](/zh-CN/cli-reference#agents-flag-format)。

**插件 subagents** 来自您已安装的 [plugins](/zh-CN/plugins)。它们与您的自定义 subagents 一起出现在 `/agents` 中。有关创建插件 subagents 的详细信息，请参阅 [插件组件参考](/zh-CN/plugins-reference#agents)。

### 编写 subagent 文件

Subagent 文件使用 YAML frontmatter 进行配置，然后是 Markdown 中的系统提示：

<Note>
  Subagents 在会话启动时加载。如果您通过手动添加文件来创建 subagent，请重启会话或使用 `/agents` 立即加载它。
</Note>

```markdown  theme={null}
---
name: code-reviewer
description: Reviews code for quality and best practices
tools: Read, Glob, Grep
model: sonnet
---

You are a code reviewer. When invoked, analyze the code and provide
specific, actionable feedback on quality, security, and best practices.
```

Frontmatter 定义了 subagent 的元数据和配置。正文成为指导 subagent 行为的系统提示。Subagents 仅接收此系统提示（加上基本环境详细信息，如工作目录），而不是完整的 Claude Code 系统提示。

#### 支持的 frontmatter 字段

以下字段可以在 YAML frontmatter 中使用。只有 `name` 和 `description` 是必需的。

| 字段                | 必需 | 描述                                                                                                                                                                            |
| :---------------- | :- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`            | 是  | 使用小写字母和连字符的唯一标识符                                                                                                                                                              |
| `description`     | 是  | Claude 何时应委托给此 subagent                                                                                                                                                       |
| `tools`           | 否  | subagent 可以使用的 [工具](#available-tools)。如果省略，继承所有工具                                                                                                                             |
| `disallowedTools` | 否  | 要拒绝的工具，从继承或指定的列表中删除                                                                                                                                                           |
| `model`           | 否  | 要使用的 [模型](#choose-a-model)：`sonnet`、`opus`、`haiku` 或 `inherit`。默认为 `inherit`                                                                                                  |
| `permissionMode`  | 否  | [权限模式](#permission-modes)：`default`、`acceptEdits`、`dontAsk`、`bypassPermissions` 或 `plan`                                                                                      |
| `maxTurns`        | 否  | subagent 停止前的最大代理轮数                                                                                                                                                           |
| `skills`          | 否  | 在启动时加载到 subagent 上下文中的 [Skills](/zh-CN/skills)。注入完整的技能内容，而不仅仅是可用于调用。Subagents 不从父对话继承技能                                                                                       |
| `mcpServers`      | 否  | 此 subagent 可用的 [MCP servers](/zh-CN/mcp)。每个条目要么是引用已配置服务器的服务器名称（例如 `"slack"`），要么是内联定义，其中服务器名称为键，完整的 [MCP 服务器配置](/zh-CN/mcp#configure-mcp-servers) 为值                           |
| `hooks`           | 否  | 限定于此 subagent 的 [生命周期 hooks](#define-hooks-for-subagents)                                                                                                                     |
| `memory`          | 否  | [持久内存范围](#enable-persistent-memory)：`user`、`project` 或 `local`。启用跨会话学习                                                                                                        |
| `background`      | 否  | 设置为 `true` 以始终将此 subagent 作为 [后台任务](#run-subagents-in-foreground-or-background) 运行。默认值：`false`                                                                                |
| `isolation`       | 否  | 设置为 `worktree` 以在临时 [git worktree](/zh-CN/common-workflows#run-parallel-claude-code-sessions-with-git-worktrees) 中运行 subagent，为其提供存储库的隔离副本。如果 subagent 不进行任何更改，worktree 会自动清理 |

### 选择模型

`model` 字段控制 subagent 使用的 [AI 模型](/zh-CN/model-config)：

* **模型别名**：使用可用的别名之一：`sonnet`、`opus` 或 `haiku`
* **inherit**：使用与主对话相同的模型
* **省略**：如果未指定，默认为 `inherit`（使用与主对话相同的模型）

### 控制 subagent 能力

您可以通过工具访问、权限模式和条件规则来控制 subagents 可以做什么。

#### 可用工具

Subagents 可以使用 Claude Code 的任何 [内部工具](/zh-CN/settings#tools-available-to-claude)。默认情况下，subagents 从主对话继承所有工具，包括 MCP 工具。

要限制工具，请使用 `tools` 字段（允许列表）或 `disallowedTools` 字段（拒绝列表）：

```yaml  theme={null}
---
name: safe-researcher
description: Research agent with restricted capabilities
tools: Read, Grep, Glob, Bash
disallowedTools: Write, Edit
---
```

#### 限制可以生成的 subagents

当代理作为主线程运行 `claude --agent` 时，它可以使用 Task 工具生成 subagents。要限制可以生成的 subagent 类型，请在 `tools` 字段中使用 `Task(agent_type)` 语法：

```yaml  theme={null}
---
name: coordinator
description: Coordinates work across specialized agents
tools: Task(worker, researcher), Read, Bash
---
```

这是一个允许列表：只有 `worker` 和 `researcher` subagents 可以生成。如果代理尝试生成任何其他类型，请求失败，代理在其提示中仅看到允许的类型。要在允许所有其他类型的同时阻止特定代理，请改用 [`permissions.deny`](#disable-specific-subagents)。

要允许生成任何 subagent 而不受限制，请使用不带括号的 `Task`：

```yaml  theme={null}
tools: Task, Read, Bash
```

如果 `Task` 完全从 `tools` 列表中省略，代理无法生成任何 subagents。此限制仅适用于作为主线程运行 `claude --agent` 的代理。Subagents 无法生成其他 subagents，因此 `Task(agent_type)` 在 subagent 定义中无效。

#### 权限模式

`permissionMode` 字段控制 subagent 如何处理权限提示。Subagents 从主对话继承权限上下文，但可以覆盖模式。

| 模式                  | 行为                    |
| :------------------ | :-------------------- |
| `default`           | 标准权限检查与提示             |
| `acceptEdits`       | 自动接受文件编辑              |
| `dontAsk`           | 自动拒绝权限提示（显式允许的工具仍然有效） |
| `bypassPermissions` | 跳过所有权限检查              |
| `plan`              | Plan mode（只读探索）       |

<Warning>
  谨慎使用 `bypassPermissions`。它跳过所有权限检查，允许 subagent 在没有批准的情况下执行任何操作。
</Warning>

如果父级使用 `bypassPermissions`，这优先并且无法被覆盖。

#### 将技能预加载到 subagents

使用 `skills` 字段在启动时将技能内容注入到 subagent 的上下文中。这为 subagent 提供领域知识，而无需在执行期间发现和加载技能。

```yaml  theme={null}
---
name: api-developer
description: Implement API endpoints following team conventions
skills:
  - api-conventions
  - error-handling-patterns
---

Implement API endpoints. Follow the conventions and patterns from the preloaded skills.
```

每个技能的完整内容被注入到 subagent 的上下文中，而不仅仅是可用于调用。Subagents 不从父对话继承技能；您必须明确列出它们。

<Note>
  这与 [在 subagent 中运行技能](/zh-CN/skills#run-skills-in-a-subagent) 相反。在 subagent 中使用 `skills` 时，subagent 控制系统提示并加载技能内容。在技能中使用 `context: fork` 时，技能内容被注入到您指定的代理中。两者都使用相同的底层系统。
</Note>

#### 启用持久内存

`memory` 字段为 subagent 提供了一个在对话中存活的持久目录。Subagent 使用此目录随时间积累知识，例如代码库模式、调试见解和架构决策。

```yaml  theme={null}
---
name: code-reviewer
description: Reviews code for quality and best practices
memory: user
---

You are a code reviewer. As you review code, update your agent memory with
patterns, conventions, and recurring issues you discover.
```

根据内存应该应用的广泛程度选择范围：

| 范围        | 位置                                            | 何时使用                          |
| :-------- | :-------------------------------------------- | :---------------------------- |
| `user`    | `~/.claude/agent-memory/<name-of-agent>/`     | subagent 应该记住跨所有项目的学习         |
| `project` | `.claude/agent-memory/<name-of-agent>/`       | subagent 的知识是特定于项目的并可通过版本控制共享 |
| `local`   | `.claude/agent-memory-local/<name-of-agent>/` | subagent 的知识是特定于项目的但不应检入版本控制  |

启用内存时：

* Subagent 的系统提示包括读取和写入内存目录的说明。
* Subagent 的系统提示还包括内存目录中 `MEMORY.md` 的前 200 行，以及如果超过 200 行则策划 `MEMORY.md` 的说明。
* Read、Write 和 Edit 工具会自动启用，以便 subagent 可以管理其内存文件。

##### 持久内存提示

* `user` 是推荐的默认范围。当 subagent 的知识仅与特定代码库相关时，使用 `project` 或 `local`。
* 要求 subagent 在开始工作前查阅其内存："Review this PR, and check your memory for patterns you've seen before."
* 要求 subagent 在完成任务后更新其内存："Now that you're done, save what you learned to your memory." 随着时间的推移，这会构建一个知识库，使 subagent 更有效。
* 直接在 subagent 的 markdown 文件中包含内存说明，以便它主动维护自己的知识库：

  ```markdown  theme={null}
  Update your agent memory as you discover codepaths, patterns, library
  locations, and key architectural decisions. This builds up institutional
  knowledge across conversations. Write concise notes about what you found
  and where.
  ```

#### 使用 hooks 的条件规则

为了更动态地控制工具使用，请使用 `PreToolUse` hooks 在执行前验证操作。当您需要允许工具的某些操作同时阻止其他操作时，这很有用。

此示例创建一个仅允许只读数据库查询的 subagent。`PreToolUse` hook 在每个 Bash 命令执行前运行 `command` 中指定的脚本：

```yaml  theme={null}
---
name: db-reader
description: Execute read-only database queries
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---
```

Claude Code [通过 stdin 将 hook 输入作为 JSON 传递](/zh-CN/hooks#pretooluse-input) 给 hook 命令。验证脚本读取此 JSON，提取 Bash 命令，并 [以代码 2 退出](/zh-CN/hooks#exit-code-2-behavior-per-event) 以阻止写入操作：

```bash  theme={null}
#!/bin/bash
# ./scripts/validate-readonly-query.sh

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# Block SQL write operations (case-insensitive)
if echo "$COMMAND" | grep -iE '\b(INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|TRUNCATE)\b' > /dev/null; then
  echo "Blocked: Only SELECT queries are allowed" >&2
  exit 2
fi

exit 0
```

有关完整的输入架构，请参阅 [Hook 输入](/zh-CN/hooks#pretooluse-input)，有关退出代码如何影响行为，请参阅 [退出代码](/zh-CN/hooks#exit-code-output)。

#### 禁用特定 subagents

您可以通过将 subagents 添加到 [settings](/zh-CN/settings#permission-settings) 中的 `deny` 数组来防止 Claude 使用特定 subagents。使用格式 `Task(subagent-name)`，其中 `subagent-name` 与 subagent 的 name 字段匹配。

```json  theme={null}
{
  "permissions": {
    "deny": ["Task(Explore)", "Task(my-custom-agent)"]
  }
}
```

这适用于内置和自定义 subagents。您也可以使用 `--disallowedTools` CLI 标志：

```bash  theme={null}
claude --disallowedTools "Task(Explore)"
```

有关权限规则的更多详细信息，请参阅 [权限文档](/zh-CN/permissions#tool-specific-permission-rules)。

### 为 subagents 定义 hooks

Subagents 可以定义在 subagent 生命周期中运行的 [hooks](/zh-CN/hooks)。有两种方式来配置 hooks：

1. **在 subagent 的 frontmatter 中**：定义仅在该 subagent 活跃时运行的 hooks
2. **在 `settings.json` 中**：定义在 subagents 启动或停止时在主会话中运行的 hooks

#### Subagent frontmatter 中的 Hooks

直接在 subagent 的 markdown 文件中定义 hooks。这些 hooks 仅在该特定 subagent 活跃时运行，并在其完成时清理。

支持所有 [hook 事件](/zh-CN/hooks#hook-events)。subagents 最常见的事件是：

| 事件            | 匹配器输入 | 何时触发                                   |
| :------------ | :---- | :------------------------------------- |
| `PreToolUse`  | 工具名称  | 在 subagent 使用工具之前                      |
| `PostToolUse` | 工具名称  | 在 subagent 使用工具之后                      |
| `Stop`        | （无）   | 当 subagent 完成时（在运行时转换为 `SubagentStop`） |

此示例使用 `PreToolUse` hook 验证 Bash 命令，并使用 `PostToolUse` 在文件编辑后运行 linter：

```yaml  theme={null}
---
name: code-reviewer
description: Review code changes with automatic linting
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-command.sh $TOOL_INPUT"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/run-linter.sh"
---
```

Frontmatter 中的 `Stop` hooks 会自动转换为 `SubagentStop` 事件。

#### 用于 subagent 事件的项目级 hooks

在 `settings.json` 中配置 hooks，以响应主会话中的 subagent 生命周期事件。

| 事件              | 匹配器输入  | 何时触发             |
| :-------------- | :----- | :--------------- |
| `SubagentStart` | 代理类型名称 | 当 subagent 开始执行时 |
| `SubagentStop`  | 代理类型名称 | 当 subagent 完成时   |

两个事件都支持匹配器以按名称针对特定代理类型。此示例仅在 `db-agent` subagent 启动时运行设置脚本，并在任何 subagent 停止时运行清理脚本：

```json  theme={null}
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "db-agent",
        "hooks": [
          { "type": "command", "command": "./scripts/setup-db-connection.sh" }
        ]
      }
    ],
    "SubagentStop": [
      {
        "hooks": [
          { "type": "command", "command": "./scripts/cleanup-db-connection.sh" }
        ]
      }
    ]
  }
}
```

有关完整的 hook 配置格式，请参阅 [Hooks](/zh-CN/hooks)。

## 使用 subagents

### 理解自动委托

Claude 根据您请求中的任务描述、subagent 配置中的 `description` 字段和当前上下文自动委托任务。为了鼓励主动委托，在 subagent 的 description 字段中包含"use proactively"之类的短语。

您也可以明确请求特定的 subagent：

```
Use the test-runner subagent to fix failing tests
Have the code-reviewer subagent look at my recent changes
```

### 在前台或后台运行 subagents

Subagents 可以在前台（阻塞）或后台（并发）运行：

* **前台 subagents** 阻塞主对话直到完成。权限提示和澄清问题（如 [`AskUserQuestion`](/zh-CN/settings#tools-available-to-claude)）会传递给您。
* **后台 subagents** 在您继续工作时并发运行。启动前，Claude Code 会提示输入 subagent 需要的任何工具权限，确保它具有必要的批准。运行后，subagent 继承这些权限并自动拒绝任何未预先批准的内容。如果后台 subagent 需要提出澄清问题，该工具调用失败，但 subagent 继续。

如果后台 subagent 因缺少权限而失败，您可以 [恢复它](#resume-subagents) 在前台以使用交互式提示重试。

Claude 根据任务决定是否在前台或后台运行 subagents。您也可以：

* 要求 Claude "run this in the background"
* 按 **Ctrl+B** 将运行中的任务放入后台

要禁用所有后台任务功能，请将 `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS` 环境变量设置为 `1`。请参阅 [环境变量](/zh-CN/settings#environment-variables)。

### 常见模式

#### 隔离高容量操作

subagents 最有效的用途之一是隔离产生大量输出的操作。运行测试、获取文档或处理日志文件可能会消耗大量上下文。通过将这些委托给 subagent，详细输出保留在 subagent 的上下文中，而只有相关摘要返回到主对话。

```
Use a subagent to run the test suite and report only the failing tests with their error messages
```

#### 并行运行研究

对于独立调查，生成多个 subagents 以同时工作：

```
Research the authentication, database, and API modules in parallel using separate subagents
```

每个 subagent 独立探索其区域，然后 Claude 综合这些发现。当研究路径不相互依赖时，这效果最好。

<Warning>
  当 subagents 完成时，其结果返回到主对话。运行许多 subagents，每个都返回详细结果，可能会消耗大量上下文。
</Warning>

对于需要持续并行性或超过上下文窗口的任务，[agent teams](/zh-CN/agent-teams) 为每个工作者提供自己的独立上下文。

#### 链接 subagents

对于多步骤工作流，要求 Claude 按顺序使用 subagents。每个 subagent 完成其任务并将结果返回给 Claude，然后将相关上下文传递给下一个 subagent。

```
Use the code-reviewer subagent to find performance issues, then use the optimizer subagent to fix them
```

### 在 subagents 和主对话之间选择

在以下情况下使用 **主对话**：

* 任务需要频繁的来回或迭代细化
* 多个阶段共享重要上下文（规划 → 实现 → 测试）
* 您正在进行快速、有针对性的更改
* 延迟很重要。Subagents 从头开始，可能需要时间来收集上下文

在以下情况下使用 **subagents**：

* 任务产生您不需要在主上下文中的详细输出
* 您想强制执行特定的工具限制或权限
* 工作是自包含的，可以返回摘要

当您想要在主对话上下文中运行的可重用提示或工作流而不是隔离的 subagent 上下文时，请考虑改用 [Skills](/zh-CN/skills)。

<Note>
  Subagents 无法生成其他 subagents。如果您的工作流需要嵌套委托，请使用 [Skills](/zh-CN/skills) 或从主对话 [链接 subagents](#chain-subagents)。
</Note>

### 管理 subagent 上下文

#### 恢复 subagents

每个 subagent 调用都会创建一个具有新鲜上下文的新实例。要继续现有 subagent 的工作而不是重新开始，请要求 Claude 恢复它。

恢复的 subagents 保留其完整的对话历史，包括所有先前的工具调用、结果和推理。Subagent 从停止的地方继续，而不是从头开始。

当 subagent 完成时，Claude 接收其代理 ID。要恢复 subagent，请要求 Claude 继续之前的工作：

```
Use the code-reviewer subagent to review the authentication module
[Agent completes]

Continue that code review and now analyze the authorization logic
[Claude resumes the subagent with full context from previous conversation]
```

您也可以要求 Claude 提供代理 ID（如果您想明确引用它），或在 `~/.claude/projects/{project}/{sessionId}/subagents/` 的转录文件中找到 ID。每个转录存储为 `agent-{agentId}.jsonl`。

Subagent 转录独立于主对话持久化：

* **主对话压缩**：当主对话压缩时，subagent 转录不受影响。它们存储在单独的文件中。
* **会话持久化**：Subagent 转录在其会话中持久化。您可以通过恢复相同会话在重启 Claude Code 后 [恢复 subagent](#resume-subagents)。
* **自动清理**：转录根据 `cleanupPeriodDays` 设置进行清理（默认：30 天）。

#### 自动压缩

Subagents 支持使用与主对话相同的逻辑进行自动压缩。默认情况下，自动压缩在大约 95% 容量时触发。要更早触发压缩，请将 `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE` 设置为较低的百分比（例如 `50`）。有关详细信息，请参阅 [环境变量](/zh-CN/settings#environment-variables)。

压缩事件记录在 subagent 转录文件中：

```json  theme={null}
{
  "type": "system",
  "subtype": "compact_boundary",
  "compactMetadata": {
    "trigger": "auto",
    "preTokens": 167189
  }
}
```

`preTokens` 值显示压缩发生前使用了多少令牌。

## 示例 subagents

这些示例演示了构建 subagents 的有效模式。将它们用作起点，或使用 Claude 生成自定义版本。

<Tip>
  **最佳实践：**

  * **设计专注的 subagents：** 每个 subagent 应该在一个特定任务中表现出色
  * **编写详细的描述：** Claude 使用描述来决定何时委托
  * **限制工具访问：** 仅授予安全和专注所需的权限
  * **检入版本控制：** 与您的团队共享项目 subagents
</Tip>

### 代码审查者

一个只读 subagent，审查代码而不修改它。此示例展示了如何设计具有有限工具访问权限（无 Edit 或 Write）和详细提示的专注 subagent，该提示明确指定要查找的内容以及如何格式化输出。

```markdown  theme={null}
---
name: code-reviewer
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: inherit
---

You are a senior code reviewer ensuring high standards of code quality and security.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Begin review immediately

Review checklist:
- Code is clear and readable
- Functions and variables are well-named
- No duplicated code
- Proper error handling
- No exposed secrets or API keys
- Input validation implemented
- Good test coverage
- Performance considerations addressed

Provide feedback organized by priority:
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (consider improving)

Include specific examples of how to fix issues.
```

### 调试器

一个既可以分析又可以修复问题的 subagent。与代码审查者不同，这个包括 Edit，因为修复错误需要修改代码。提示提供了从诊断到验证的清晰工作流。

```markdown  theme={null}
---
name: debugger
description: Debugging specialist for errors, test failures, and unexpected behavior. Use proactively when encountering any issues.
tools: Read, Edit, Bash, Grep, Glob
---

You are an expert debugger specializing in root cause analysis.

When invoked:
1. Capture error message and stack trace
2. Identify reproduction steps
3. Isolate the failure location
4. Implement minimal fix
5. Verify solution works

Debugging process:
- Analyze error messages and logs
- Check recent code changes
- Form and test hypotheses
- Add strategic debug logging
- Inspect variable states

For each issue, provide:
- Root cause explanation
- Evidence supporting the diagnosis
- Specific code fix
- Testing approach
- Prevention recommendations

Focus on fixing the underlying issue, not the symptoms.
```

### 数据科学家

用于数据分析工作的特定领域 subagent。此示例展示了如何为典型编码任务之外的专门工作流创建 subagents。它明确设置 `model: sonnet` 以获得更强大的分析能力。

```markdown  theme={null}
---
name: data-scientist
description: Data analysis expert for SQL queries, BigQuery operations, and data insights. Use proactively for data analysis tasks and queries.
tools: Bash, Read, Write
model: sonnet
---

You are a data scientist specializing in SQL and BigQuery analysis.

When invoked:
1. Understand the data analysis requirement
2. Write efficient SQL queries
3. Use BigQuery command line tools (bq) when appropriate
4. Analyze and summarize results
5. Present findings clearly

Key practices:
- Write optimized SQL queries with proper filters
- Use appropriate aggregations and joins
- Include comments explaining complex logic
- Format results for readability
- Provide data-driven recommendations

For each analysis:
- Explain the query approach
- Document any assumptions
- Highlight key findings
- Suggest next steps based on data

Always ensure queries are efficient and cost-effective.
```

### 数据库查询验证器

一个允许 Bash 访问但验证命令以仅允许只读 SQL 查询的 subagent。此示例展示了如何在需要比 `tools` 字段提供的更精细控制时使用 `PreToolUse` hooks。

```markdown  theme={null}
---
name: db-reader
description: Execute read-only database queries. Use when analyzing data or generating reports.
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---

You are a database analyst with read-only access. Execute SELECT queries to answer questions about the data.

When asked to analyze data:
1. Identify which tables contain the relevant data
2. Write efficient SELECT queries with appropriate filters
3. Present results clearly with context

You cannot modify data. If asked to INSERT, UPDATE, DELETE, or modify schema, explain that you only have read access.
```

Claude Code [通过 stdin 将 hook 输入作为 JSON 传递](/zh-CN/hooks#pretooluse-input) 给 hook 命令。验证脚本读取此 JSON，提取正在执行的命令，并根据 SQL 写入操作列表检查它。如果检测到写入操作，脚本 [以代码 2 退出](/zh-CN/hooks#exit-code-2-behavior-per-event) 以阻止执行并通过 stderr 向 Claude 返回错误消息。

在项目中的任何位置创建验证脚本。路径必须与 hook 配置中的 `command` 字段匹配：

```bash  theme={null}
#!/bin/bash
# Blocks SQL write operations, allows SELECT queries

# Read JSON input from stdin
INPUT=$(cat)

# Extract the command field from tool_input using jq
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

if [ -z "$COMMAND" ]; then
  exit 0
fi

# Block write operations (case-insensitive)
if echo "$COMMAND" | grep -iE '\b(INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|TRUNCATE|REPLACE|MERGE)\b' > /dev/null; then
  echo "Blocked: Write operations not allowed. Use SELECT queries only." >&2
  exit 2
fi

exit 0
```

使脚本可执行：

```bash  theme={null}
chmod +x ./scripts/validate-readonly-query.sh
```

Hook 通过 stdin 接收 JSON，Bash 命令在 `tool_input.command` 中。退出代码 2 阻止操作并将错误消息反馈给 Claude。有关退出代码和 [Hook 输入](/zh-CN/hooks#pretooluse-input) 的完整输入架构的详细信息，请参阅 [Hooks](/zh-CN/hooks#exit-code-output)。

## 后续步骤

现在您了解了 subagents，请探索这些相关功能：

* [使用插件分发 subagents](/zh-CN/plugins) 以跨团队或项目共享 subagents
* [以编程方式运行 Claude Code](/zh-CN/headless) 使用 Agent SDK 进行 CI/CD 和自动化
* [使用 MCP servers](/zh-CN/mcp) 为 subagents 提供对外部工具和数据的访问
