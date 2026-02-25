> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 创建插件

> 创建自定义插件以使用 skills、agents、hooks 和 MCP servers 扩展 Claude Code。

Plugins 让你能够使用自定义功能扩展 Claude Code，这些功能可以在项目和团队中共享。本指南涵盖如何使用 skills、agents、hooks 和 MCP servers 创建自己的插件。

想要安装现有插件？请参阅[发现和安装插件](/zh-CN/discover-plugins)。有关完整的技术规范，请参阅[插件参考](/zh-CN/plugins-reference)。

## 何时使用插件与独立配置

Claude Code 支持两种方式来添加自定义 skills、agents 和 hooks：

| 方法                                          | Skill 名称             | 最适合                       |
| :------------------------------------------ | :------------------- | :------------------------ |
| **独立**（`.claude/` 目录）                       | `/hello`             | 个人工作流、项目特定的自定义、快速实验       |
| **插件**（包含 `.claude-plugin/plugin.json` 的目录） | `/plugin-name:hello` | 与团队成员共享、分发到社区、版本化发布、跨项目重用 |

**在以下情况下使用独立配置**：

* 你正在为单个项目自定义 Claude Code
* 配置是个人的，不需要共享
* 你在打包 skills 或 hooks 之前进行实验
* 你想要简短的 skill 名称，如 `/hello` 或 `/review`

**在以下情况下使用插件**：

* 你想与团队或社区共享功能
* 你需要在多个项目中使用相同的 skills/agents
* 你想要版本控制和轻松更新扩展
* 你通过市场进行分发
* 你可以接受命名空间化的 skills，如 `/my-plugin:hello`（命名空间可防止插件之间的冲突）

<Tip>
  从 `.claude/` 中的独立配置开始进行快速迭代，然后在准备好共享时[转换为插件](#convert-existing-configurations-to-plugins)。
</Tip>

## 快速开始

本快速开始将引导你创建一个具有自定义 skill 的插件。你将创建一个清单（定义插件的配置文件）、添加一个 skill，并使用 `--plugin-dir` 标志在本地测试它。

### 前置条件

* Claude Code [已安装并已认证](/zh-CN/quickstart#step-1-install-claude-code)
* Claude Code 版本 1.0.33 或更高版本（运行 `claude --version` 检查）

<Note>
  如果你没有看到 `/plugin` 命令，请将 Claude Code 更新到最新版本。有关升级说明，请参阅[故障排除](/zh-CN/troubleshooting)。
</Note>

### 创建你的第一个插件

<Steps>
  <Step title="创建插件目录">
    每个插件都位于其自己的目录中，该目录包含清单和你的 skills、agents 或 hooks。现在创建一个：

    ```bash  theme={null}
    mkdir my-first-plugin
    ```
  </Step>

  <Step title="创建插件清单">
    位于 `.claude-plugin/plugin.json` 的清单文件定义了你的插件的身份：其名称、描述和版本。Claude Code 使用此元数据在插件管理器中显示你的插件。

    在你的插件文件夹内创建 `.claude-plugin` 目录：

    ```bash  theme={null}
    mkdir my-first-plugin/.claude-plugin
    ```

    然后使用以下内容创建 `my-first-plugin/.claude-plugin/plugin.json`：

    ```json my-first-plugin/.claude-plugin/plugin.json theme={null}
    {
    "name": "my-first-plugin",
    "description": "A greeting plugin to learn the basics",
    "version": "1.0.0",
    "author": {
    "name": "Your Name"
    }
    }
    ```

    | 字段            | 目的                                                           |
    | :------------ | :----------------------------------------------------------- |
    | `name`        | 唯一标识符和 skill 命名空间。Skills 以此为前缀（例如 `/my-first-plugin:hello`）。 |
    | `description` | 在浏览或安装插件时在插件管理器中显示。                                          |
    | `version`     | 使用[语义版本控制](/zh-CN/plugins-reference#version-management)跟踪发布。 |
    | `author`      | 可选。有助于归属。                                                    |

    有关 `homepage`、`repository` 和 `license` 等其他字段，请参阅[完整清单架构](/zh-CN/plugins-reference#plugin-manifest-schema)。
  </Step>

  <Step title="添加 skill">
    Skills 位于 `skills/` 目录中。每个 skill 是一个包含 `SKILL.md` 文件的文件夹。文件夹名称成为 skill 名称，以插件的命名空间为前缀（在名为 `my-first-plugin` 的插件中的 `hello/` 创建 `/my-first-plugin:hello`）。

    在你的插件文件夹中创建一个 skill 目录：

    ```bash  theme={null}
    mkdir -p my-first-plugin/skills/hello
    ```

    然后使用以下内容创建 `my-first-plugin/skills/hello/SKILL.md`：

    ```markdown my-first-plugin/skills/hello/SKILL.md theme={null}
    ---
    description: Greet the user with a friendly message
    disable-model-invocation: true
    ---

    Greet the user warmly and ask how you can help them today.
    ```
  </Step>

  <Step title="测试你的插件">
    使用 `--plugin-dir` 标志运行 Claude Code 以加载你的插件：

    ```bash  theme={null}
    claude --plugin-dir ./my-first-plugin
    ```

    Claude Code 启动后，尝试你的新命令：

    ```shell  theme={null}
    /my-first-plugin:hello
    ```

    你将看到 Claude 用问候语回应。运行 `/help` 以查看你的命令在插件命名空间下列出。

    <Note>
      **为什么要命名空间？** 插件 skills 总是命名空间化的（如 `/greet:hello`），以防止多个插件具有相同名称的 skills 时发生冲突。

      要更改命名空间前缀，请更新 `plugin.json` 中的 `name` 字段。
    </Note>
  </Step>

  <Step title="添加 skill 参数">
    通过接受用户输入使你的 skill 动态化。`$ARGUMENTS` 占位符捕获用户在 skill 名称后提供的任何文本。

    更新你的 `hello.md` 文件：

    ```markdown my-first-plugin/commands/hello.md theme={null}
    ---
    description: Greet the user with a personalized message
    ---

    # Hello Command

    Greet the user named "$ARGUMENTS" warmly and ask how you can help them today. Make the greeting personal and encouraging.
    ```

    重启 Claude Code 以获取更改，然后尝试使用你的名字运行命令：

    ```shell  theme={null}
    /my-first-plugin:hello Alex
    ```

    Claude 将按名字问候你。有关向 skills 传递参数的更多信息，请参阅[Skills](/zh-CN/skills#pass-arguments-to-skills)。
  </Step>
</Steps>

你已成功创建并测试了一个具有以下关键组件的插件：

* **插件清单**（`.claude-plugin/plugin.json`）：描述你的插件的元数据
* **命令目录**（`commands/`）：包含你的自定义 skills
* **Skill 参数**（`$ARGUMENTS`）：捕获用户输入以实现动态行为

<Tip>
  `--plugin-dir` 标志对开发和测试很有用。当你准备好与他人共享你的插件时，请参阅[创建和分发插件市场](/zh-CN/plugin-marketplaces)。
</Tip>

## 插件结构概览

你已创建了一个具有 skill 的插件，但插件可以包含更多内容：自定义 agents、hooks、MCP servers 和 LSP servers。

<Warning>
  **常见错误**：不要将 `commands/`、`agents/`、`skills/` 或 `hooks/` 放在 `.claude-plugin/` 目录内。只有 `plugin.json` 应该在 `.claude-plugin/` 内。所有其他目录必须在插件根级别。
</Warning>

| 目录                | 位置    | 目的                                  |
| :---------------- | :---- | :---------------------------------- |
| `.claude-plugin/` | 插件根目录 | 包含 `plugin.json` 清单（如果组件使用默认位置，则可选） |
| `commands/`       | 插件根目录 | 作为 Markdown 文件的 Skills              |
| `agents/`         | 插件根目录 | 自定义 agent 定义                        |
| `skills/`         | 插件根目录 | 具有 `SKILL.md` 文件的 Agent Skills      |
| `hooks/`          | 插件根目录 | `hooks.json` 中的事件处理程序               |
| `.mcp.json`       | 插件根目录 | MCP server 配置                       |
| `.lsp.json`       | 插件根目录 | 用于代码智能的 LSP server 配置               |

<Note>
  **后续步骤**：准备好添加更多功能了吗？跳转到[开发更复杂的插件](#develop-more-complex-plugins)以添加 agents、hooks、MCP servers 和 LSP servers。有关所有插件组件的完整技术规范，请参阅[插件参考](/zh-CN/plugins-reference)。
</Note>

## 开发更复杂的插件

一旦你对基本插件感到满意，你可以创建更复杂的扩展。

### 向你的插件添加 Skills

插件可以包含[Agent Skills](/zh-CN/skills)以扩展 Claude 的功能。Skills 是模型调用的：Claude 根据任务上下文自动使用它们。

在你的插件根目录添加一个 `skills/` 目录，其中包含包含 `SKILL.md` 文件的 Skill 文件夹：

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── code-review/
        └── SKILL.md
```

每个 `SKILL.md` 需要包含 `name` 和 `description` 字段的 frontmatter，后跟说明：

```yaml  theme={null}
---
name: code-review
description: Reviews code for best practices and potential issues. Use when reviewing code, checking PRs, or analyzing code quality.
---

When reviewing code, check for:
1. Code organization and structure
2. Error handling
3. Security concerns
4. Test coverage
```

安装插件后，重启 Claude Code 以加载 Skills。有关完整的 Skill 编写指南，包括渐进式披露和工具限制，请参阅[Agent Skills](/zh-CN/skills)。

### 向你的插件添加 LSP servers

<Tip>
  对于 TypeScript、Python 和 Rust 等常见语言，请从官方市场安装预构建的 LSP 插件。仅当你需要支持尚未覆盖的语言时，才创建自定义 LSP 插件。
</Tip>

LSP（Language Server Protocol）插件为 Claude 提供实时代码智能。如果你需要支持没有官方 LSP 插件的语言，你可以通过向你的插件添加 `.lsp.json` 文件来创建自己的插件：

```json .lsp.json theme={null}
{
  "go": {
    "command": "gopls",
    "args": ["serve"],
    "extensionToLanguage": {
      ".go": "go"
    }
  }
}
```

安装你的插件的用户必须在其机器上安装语言服务器二进制文件。

有关完整的 LSP 配置选项，请参阅[LSP servers](/zh-CN/plugins-reference#lsp-servers)。

### 组织复杂的插件

对于具有许多组件的插件，按功能组织你的目录结构。有关完整的目录布局和组织模式，请参阅[插件目录结构](/zh-CN/plugins-reference#plugin-directory-structure)。

### 在本地测试你的插件

使用 `--plugin-dir` 标志在开发期间测试插件。这会直接加载你的插件，无需安装。

```bash  theme={null}
claude --plugin-dir ./my-plugin
```

当你对插件进行更改时，重启 Claude Code 以获取更新。测试你的插件组件：

* 使用 `/command-name` 尝试你的命令
* 检查 agents 是否出现在 `/agents` 中
* 验证 hooks 是否按预期工作

<Tip>
  你可以通过多次指定标志来一次加载多个插件：

  ```bash  theme={null}
  claude --plugin-dir ./plugin-one --plugin-dir ./plugin-two
  ```
</Tip>

### 调试插件问题

如果你的插件不按预期工作：

1. **检查结构**：确保你的目录在插件根目录，而不是在 `.claude-plugin/` 内
2. **单独测试组件**：分别检查每个命令、agent 和 hook
3. **使用验证和调试工具**：有关 CLI 命令和故障排除技术，请参阅[调试和开发工具](/zh-CN/plugins-reference#debugging-and-development-tools)

### 共享你的插件

当你的插件准备好共享时：

1. **添加文档**：包含一个 `README.md`，其中包含安装和使用说明
2. **对你的插件进行版本控制**：在你的 `plugin.json` 中使用[语义版本控制](/zh-CN/plugins-reference#version-management)
3. **创建或使用市场**：通过[插件市场](/zh-CN/plugin-marketplaces)进行分发以供安装
4. **与他人测试**：在更广泛的分发之前让团队成员测试插件

一旦你的插件在市场中，其他人可以使用[发现和安装插件](/zh-CN/discover-plugins)中的说明安装它。

<Note>
  有关完整的技术规范、调试技术和分发策略，请参阅[插件参考](/zh-CN/plugins-reference)。
</Note>

## 将现有配置转换为插件

如果你已经在 `.claude/` 目录中有 skills 或 hooks，你可以将它们转换为插件以便于共享和分发。

### 迁移步骤

<Steps>
  <Step title="创建插件结构">
    创建一个新的插件目录：

    ```bash  theme={null}
    mkdir -p my-plugin/.claude-plugin
    ```

    在 `my-plugin/.claude-plugin/plugin.json` 处创建清单文件：

    ```json my-plugin/.claude-plugin/plugin.json theme={null}
    {
      "name": "my-plugin",
      "description": "Migrated from standalone configuration",
      "version": "1.0.0"
    }
    ```
  </Step>

  <Step title="复制你现有的文件">
    将你现有的配置复制到插件目录：

    ```bash  theme={null}
    # Copy commands
    cp -r .claude/commands my-plugin/

    # Copy agents (if any)
    cp -r .claude/agents my-plugin/

    # Copy skills (if any)
    cp -r .claude/skills my-plugin/
    ```
  </Step>

  <Step title="迁移 hooks">
    如果你在设置中有 hooks，请创建一个 hooks 目录：

    ```bash  theme={null}
    mkdir my-plugin/hooks
    ```

    使用你的 hooks 配置创建 `my-plugin/hooks/hooks.json`。从你的 `.claude/settings.json` 或 `settings.local.json` 复制 `hooks` 对象，因为格式相同。命令在 stdin 上接收 hook 输入作为 JSON，所以使用 `jq` 提取文件路径：

    ```json my-plugin/hooks/hooks.json theme={null}
    {
      "hooks": {
        "PostToolUse": [
          {
            "matcher": "Write|Edit",
            "hooks": [{ "type": "command", "command": "jq -r '.tool_input.file_path' | xargs npm run lint:fix" }]
          }
        ]
      }
    }
    ```
  </Step>

  <Step title="测试你迁移的插件">
    加载你的插件以验证一切正常：

    ```bash  theme={null}
    claude --plugin-dir ./my-plugin
    ```

    测试每个组件：运行你的命令，检查 agents 是否出现在 `/agents` 中，并验证 hooks 是否正确触发。
  </Step>
</Steps>

### 迁移时的变化

| 独立（`.claude/`）           | 插件                           |
| :----------------------- | :--------------------------- |
| 仅在一个项目中可用                | 可以通过市场共享                     |
| `.claude/commands/` 中的文件 | `plugin-name/commands/` 中的文件 |
| `settings.json` 中的 Hooks | `hooks/hooks.json` 中的 Hooks  |
| 必须手动复制以共享                | 使用 `/plugin install` 安装      |

<Note>
  迁移后，你可以从 `.claude/` 中删除原始文件以避免重复。加载时插件版本将优先。
</Note>

## 后续步骤

现在你了解了 Claude Code 的插件系统，以下是针对不同目标的建议路径：

### 对于插件用户

* [发现和安装插件](/zh-CN/discover-plugins)：浏览市场并安装插件
* [配置团队市场](/zh-CN/discover-plugins#configure-team-marketplaces)：为你的团队设置存储库级别的插件

### 对于插件开发者

* [创建和分发市场](/zh-CN/plugin-marketplaces)：打包和共享你的插件
* [插件参考](/zh-CN/plugins-reference)：完整的技术规范
* 深入了解特定的插件组件：
  * [Skills](/zh-CN/skills)：skill 开发详情
  * [Subagents](/zh-CN/sub-agents)：agent 配置和功能
  * [Hooks](/zh-CN/hooks)：事件处理和自动化
  * [MCP](/zh-CN/mcp)：外部工具集成
