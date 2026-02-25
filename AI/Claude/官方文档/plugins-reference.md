> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 插件参考

> Claude Code 插件系统的完整技术参考，包括架构、CLI 命令和组件规范。

<Tip>
  想要安装插件？请参阅[发现和安装插件](/zh-CN/discover-plugins)。如需创建插件，请参阅[插件](/zh-CN/plugins)。如需分发插件，请参阅[插件市场](/zh-CN/plugin-marketplaces)。
</Tip>

本参考提供了 Claude Code 插件系统的完整技术规范，包括组件架构、CLI 命令和开发工具。

**插件**是一个自包含的组件目录，用于扩展 Claude Code 的自定义功能。插件组件包括 skills、agents、hooks、MCP servers 和 LSP servers。

## 插件组件参考

### Skills

插件向 Claude Code 添加 skills，创建可由您或 Claude 调用的 `/name` 快捷方式。

**位置**：插件根目录中的 `skills/` 或 `commands/` 目录

**文件格式**：Skills 是包含 `SKILL.md` 的目录；commands 是简单的 markdown 文件

**Skill 结构**：

```
skills/
├── pdf-processor/
│   ├── SKILL.md
│   ├── reference.md (可选)
│   └── scripts/ (可选)
└── code-reviewer/
    └── SKILL.md
```

**集成行为**：

* 安装插件时会自动发现 skills 和 commands
* Claude 可以根据任务上下文自动调用它们
* Skills 可以在 SKILL.md 旁边包含支持文件

有关完整详情，请参阅 [Skills](/zh-CN/skills)。

### Agents

插件可以为特定任务提供专门的 subagents，Claude 可以在适当时自动调用。

**位置**：插件根目录中的 `agents/` 目录

**文件格式**：描述 agent 功能的 Markdown 文件

**Agent 结构**：

```markdown  theme={null}
---
name: agent-name
description: 该 agent 的专长以及 Claude 应何时调用它
---

详细的系统提示，描述 agent 的角色、专业知识和行为。
```

**集成点**：

* Agents 出现在 `/agents` 界面中
* Claude 可以根据任务上下文自动调用 agents
* Agents 可以由用户手动调用
* 插件 agents 与内置 Claude agents 一起工作

有关完整详情，请参阅 [Subagents](/zh-CN/sub-agents)。

### Hooks

插件可以提供事件处理程序，自动响应 Claude Code 事件。

**位置**：插件根目录中的 `hooks/hooks.json`，或在 plugin.json 中内联

**格式**：具有事件匹配器和操作的 JSON 配置

**Hook 配置**：

```json  theme={null}
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/format-code.sh"
          }
        ]
      }
    ]
  }
}
```

**可用事件**：

* `PreToolUse`：Claude 使用任何工具之前
* `PostToolUse`：Claude 成功使用任何工具之后
* `PostToolUseFailure`：Claude 工具执行失败之后
* `PermissionRequest`：显示权限对话框时
* `UserPromptSubmit`：用户提交提示时
* `Notification`：Claude Code 发送通知时
* `Stop`：Claude 尝试停止时
* `SubagentStart`：subagent 启动时
* `SubagentStop`：subagent 尝试停止时
* `SessionStart`：会话开始时
* `SessionEnd`：会话结束时
* `TeammateIdle`：agent 团队队友即将空闲时
* `TaskCompleted`：任务被标记为已完成时
* `PreCompact`：对话历史被压缩之前

**Hook 类型**：

* `command`：执行 shell 命令或脚本
* `prompt`：使用 LLM 评估提示（使用 `$ARGUMENTS` 占位符表示上下文）
* `agent`：运行具有工具的 agentic 验证器以完成复杂验证任务

### MCP servers

插件可以捆绑 Model Context Protocol (MCP) servers 以将 Claude Code 与外部工具和服务连接。

**位置**：插件根目录中的 `.mcp.json`，或在 plugin.json 中内联

**格式**：标准 MCP 服务器配置

**MCP 服务器配置**：

```json  theme={null}
{
  "mcpServers": {
    "plugin-database": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
      "env": {
        "DB_PATH": "${CLAUDE_PLUGIN_ROOT}/data"
      }
    },
    "plugin-api-client": {
      "command": "npx",
      "args": ["@company/mcp-server", "--plugin-mode"],
      "cwd": "${CLAUDE_PLUGIN_ROOT}"
    }
  }
}
```

**集成行为**：

* 启用插件时，插件 MCP servers 会自动启动
* Servers 在 Claude 的工具包中显示为标准 MCP 工具
* 服务器功能与 Claude 的现有工具无缝集成
* 插件 servers 可以独立于用户 MCP servers 进行配置

### LSP servers

<Tip>
  想要使用 LSP 插件？从官方市场安装它们：在 `/plugin` Discover 选项卡中搜索"lsp"。本部分记录如何为官方市场未涵盖的语言创建 LSP 插件。
</Tip>

插件可以提供 [Language Server Protocol](https://microsoft.github.io/language-server-protocol/) (LSP) servers，在处理代码库时为 Claude 提供实时代码智能。

LSP 集成提供：

* **即时诊断**：Claude 在每次编辑后立即看到错误和警告
* **代码导航**：转到定义、查找引用和悬停信息
* **语言感知**：代码符号的类型信息和文档

**位置**：插件根目录中的 `.lsp.json`，或在 `plugin.json` 中内联

**格式**：将语言服务器名称映射到其配置的 JSON 配置

**`.lsp.json` 文件格式**：

```json  theme={null}
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

**在 `plugin.json` 中内联**：

```json  theme={null}
{
  "name": "my-plugin",
  "lspServers": {
    "go": {
      "command": "gopls",
      "args": ["serve"],
      "extensionToLanguage": {
        ".go": "go"
      }
    }
  }
}
```

**必需字段：**

| 字段                    | 描述                         |
| :-------------------- | :------------------------- |
| `command`             | 要执行的 LSP 二进制文件（必须在 PATH 中） |
| `extensionToLanguage` | 将文件扩展名映射到语言标识符             |

**可选字段：**

| 字段                      | 描述                                          |
| :---------------------- | :------------------------------------------ |
| `args`                  | LSP 服务器的命令行参数                               |
| `transport`             | 通信传输：`stdio`（默认）或 `socket`                  |
| `env`                   | 启动服务器时要设置的环境变量                              |
| `initializationOptions` | 在初始化期间传递给服务器的选项                             |
| `settings`              | 通过 `workspace/didChangeConfiguration` 传递的设置 |
| `workspaceFolder`       | 服务器的工作区文件夹路径                                |
| `startupTimeout`        | 等待服务器启动的最长时间（毫秒）                            |
| `shutdownTimeout`       | 等待正常关闭的最长时间（毫秒）                             |
| `restartOnCrash`        | 服务器崩溃时是否自动重启                                |
| `maxRestarts`           | 放弃前的最大重启尝试次数                                |

<Warning>
  **您必须单独安装语言服务器二进制文件。** LSP 插件配置 Claude Code 如何连接到语言服务器，但它们不包括服务器本身。如果在 `/plugin` Errors 选项卡中看到 `Executable not found in $PATH`，请为您的语言安装所需的二进制文件。
</Warning>

**可用的 LSP 插件：**

| 插件               | 语言服务器                      | 安装命令                                                                            |
| :--------------- | :------------------------- | :------------------------------------------------------------------------------ |
| `pyright-lsp`    | Pyright (Python)           | `pip install pyright` 或 `npm install -g pyright`                                |
| `typescript-lsp` | TypeScript Language Server | `npm install -g typescript-language-server typescript`                          |
| `rust-lsp`       | rust-analyzer              | [查看 rust-analyzer 安装](https://rust-analyzer.github.io/manual.html#installation) |

首先安装语言服务器，然后从市场安装插件。

***

## 插件安装范围

安装插件时，您选择一个**范围**来确定插件的可用位置以及谁可以使用它：

| 范围        | 设置文件                                   | 用例                 |
| :-------- | :------------------------------------- | :----------------- |
| `user`    | `~/.claude/settings.json`              | 在所有项目中可用的个人插件（默认）  |
| `project` | `.claude/settings.json`                | 通过版本控制共享的团队插件      |
| `local`   | `.claude/settings.local.json`          | 项目特定的插件，gitignored |
| `managed` | [托管设置](/zh-CN/settings#settings-files) | 托管插件（只读，仅更新）       |

插件使用与其他 Claude Code 配置相同的范围系统。有关安装说明和范围标志，请参阅[安装插件](/zh-CN/discover-plugins#install-plugins)。有关范围的完整说明，请参阅[配置范围](/zh-CN/settings#configuration-scopes)。

***

## 插件清单架构

`.claude-plugin/plugin.json` 文件定义了您的插件的元数据和配置。本部分记录所有支持的字段和选项。

清单是可选的。如果省略，Claude Code 会自动发现[默认位置](#file-locations-reference)中的组件，并从目录名称派生插件名称。当您需要提供元数据或自定义组件路径时，使用清单。

### 完整架构

```json  theme={null}
{
  "name": "plugin-name",
  "version": "1.2.0",
  "description": "简要插件描述",
  "author": {
    "name": "作者名称",
    "email": "author@example.com",
    "url": "https://github.com/author"
  },
  "homepage": "https://docs.example.com/plugin",
  "repository": "https://github.com/author/plugin",
  "license": "MIT",
  "keywords": ["keyword1", "keyword2"],
  "commands": ["./custom/commands/special.md"],
  "agents": "./custom/agents/",
  "skills": "./custom/skills/",
  "hooks": "./config/hooks.json",
  "mcpServers": "./mcp-config.json",
  "outputStyles": "./styles/",
  "lspServers": "./.lsp.json"
}
```

### 必需字段

如果包含清单，`name` 是唯一必需的字段。

| 字段     | 类型     | 描述                    | 示例                   |
| :----- | :----- | :-------------------- | :------------------- |
| `name` | string | 唯一标识符（kebab-case，无空格） | `"deployment-tools"` |

此名称用于命名空间组件。例如，在 UI 中，名为 `plugin-dev` 的插件的 agent `agent-creator` 将显示为 `plugin-dev:agent-creator`。

### 元数据字段

| 字段            | 类型     | 描述                                             | 示例                                                 |
| :------------ | :----- | :--------------------------------------------- | :------------------------------------------------- |
| `version`     | string | 语义版本。如果也在市场条目中设置，`plugin.json` 优先。您只需在一个地方设置它。 | `"2.1.0"`                                          |
| `description` | string | 插件目的的简要说明                                      | `"部署自动化工具"`                                        |
| `author`      | object | 作者信息                                           | `{"name": "Dev Team", "email": "dev@company.com"}` |
| `homepage`    | string | 文档 URL                                         | `"https://docs.example.com"`                       |
| `repository`  | string | 源代码 URL                                        | `"https://github.com/user/plugin"`                 |
| `license`     | string | 许可证标识符                                         | `"MIT"`、`"Apache-2.0"`                             |
| `keywords`    | array  | 发现标签                                           | `["deployment", "ci-cd"]`                          |

### 组件路径字段

| 字段             | 类型                    | 描述                                                                                                     | 示例                                    |
| :------------- | :-------------------- | :----------------------------------------------------------------------------------------------------- | :------------------------------------ |
| `commands`     | string\|array         | 其他命令文件/目录                                                                                              | `"./custom/cmd.md"` 或 `["./cmd1.md"]` |
| `agents`       | string\|array         | 其他 agent 文件                                                                                            | `"./custom/agents/reviewer.md"`       |
| `skills`       | string\|array         | 其他 skill 目录                                                                                            | `"./custom/skills/"`                  |
| `hooks`        | string\|array\|object | Hook 配置路径或内联配置                                                                                         | `"./my-extra-hooks.json"`             |
| `mcpServers`   | string\|array\|object | MCP 配置路径或内联配置                                                                                          | `"./my-extra-mcp-config.json"`        |
| `outputStyles` | string\|array         | 其他输出样式文件/目录                                                                                            | `"./styles/"`                         |
| `lspServers`   | string\|array\|object | [Language Server Protocol](https://microsoft.github.io/language-server-protocol/) 配置用于代码智能（转到定义、查找引用等） | `"./.lsp.json"`                       |

### 路径行为规则

**重要**：自定义路径补充默认目录 - 它们不替换默认目录。

* 如果 `commands/` 存在，它会与自定义命令路径一起加载
* 所有路径必须相对于插件根目录并以 `./` 开头
* 来自自定义路径的命令使用相同的命名和命名空间规则
* 可以将多个路径指定为数组以获得灵活性

**路径示例**：

```json  theme={null}
{
  "commands": [
    "./specialized/deploy.md",
    "./utilities/batch-process.md"
  ],
  "agents": [
    "./custom-agents/reviewer.md",
    "./custom-agents/tester.md"
  ]
}
```

### 环境变量

**`${CLAUDE_PLUGIN_ROOT}`**：包含插件目录的绝对路径。在 hooks、MCP servers 和脚本中使用此变量，以确保无论安装位置如何都能正确路径。

```json  theme={null}
{
  "hooks": {
    "PostToolUse": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/process.sh"
          }
        ]
      }
    ]
  }
}
```

***

## 插件缓存和文件解析

插件通过以下两种方式之一指定：

* 通过 `claude --plugin-dir`，在会话期间。
* 通过市场，为未来的会话安装。

出于安全和验证目的，Claude Code 将\_市场\_插件复制到用户的本地**插件缓存**（`~/.claude/plugins/cache`），而不是就地使用它们。在开发引用外部文件的插件时，理解此行为很重要。

### 路径遍历限制

已安装的插件无法引用其目录外的文件。遍历插件根目录外的路径（例如 `../shared-utils`）在安装后将不起作用，因为这些外部文件不会被复制到缓存中。

### 使用外部依赖

如果您的插件需要访问其目录外的文件，您可以在插件目录中创建指向外部文件的符号链接。在复制过程中会遵守符号链接：

```bash  theme={null}
# 在您的插件目录内
ln -s /path/to/shared-utils ./shared-utils
```

符号链接的内容将被复制到插件缓存中。这在保持缓存系统安全优势的同时提供了灵活性。

***

## 插件目录结构

### 标准插件布局

完整的插件遵循此结构：

```
enterprise-plugin/
├── .claude-plugin/           # 元数据目录（可选）
│   └── plugin.json             # 插件清单
├── commands/                 # 默认命令位置
│   ├── status.md
│   └── logs.md
├── agents/                   # 默认 agent 位置
│   ├── security-reviewer.md
│   ├── performance-tester.md
│   └── compliance-checker.md
├── skills/                   # Agent Skills
│   ├── code-reviewer/
│   │   └── SKILL.md
│   └── pdf-processor/
│       ├── SKILL.md
│       └── scripts/
├── hooks/                    # Hook 配置
│   ├── hooks.json           # 主 hook 配置
│   └── security-hooks.json  # 其他 hooks
├── settings.json            # 插件的默认设置
├── .mcp.json                # MCP 服务器定义
├── .lsp.json                # LSP 服务器配置
├── scripts/                 # Hook 和实用脚本
│   ├── security-scan.sh
│   ├── format-code.py
│   └── deploy.js
├── LICENSE                  # 许可证文件
└── CHANGELOG.md             # 版本历史
```

<Warning>
  `.claude-plugin/` 目录包含 `plugin.json` 文件。所有其他目录（commands/、agents/、skills/、hooks/）必须在插件根目录，而不是在 `.claude-plugin/` 内。
</Warning>

### 文件位置参考

| 组件              | 默认位置                         | 目的                                                 |
| :-------------- | :--------------------------- | :------------------------------------------------- |
| **清单**          | `.claude-plugin/plugin.json` | 插件元数据和配置（可选）                                       |
| **命令**          | `commands/`                  | Skill Markdown 文件（旧版；新 skills 使用 `skills/`）        |
| **Agents**      | `agents/`                    | Subagent Markdown 文件                               |
| **Skills**      | `skills/`                    | 具有 `<name>/SKILL.md` 结构的 Skills                    |
| **Hooks**       | `hooks/hooks.json`           | Hook 配置                                            |
| **MCP servers** | `.mcp.json`                  | MCP 服务器定义                                          |
| **LSP servers** | `.lsp.json`                  | 语言服务器配置                                            |
| **设置**          | `settings.json`              | 启用插件时应用的默认配置。目前仅支持 [`agent`](/zh-CN/sub-agents) 设置 |

***

## CLI 命令参考

Claude Code 提供 CLI 命令用于非交互式插件管理，对脚本和自动化很有用。

### plugin install

从可用市场安装插件。

```bash  theme={null}
claude plugin install <plugin> [options]
```

**参数：**

* `<plugin>`：插件名称或 `plugin-name@marketplace-name` 用于特定市场

**选项：**

| 选项                    | 描述                              | 默认值    |
| :-------------------- | :------------------------------ | :----- |
| `-s, --scope <scope>` | 安装范围：`user`、`project` 或 `local` | `user` |
| `-h, --help`          | 显示命令帮助                          |        |

范围确定将已安装的插件添加到哪个设置文件。例如，--scope project 写入 `.claude/settings.json` 中的 `enabledPlugins`，使插件对克隆项目存储库的每个人都可用。

**示例：**

```bash  theme={null}
# 安装到用户范围（默认）
claude plugin install formatter@my-marketplace

# 安装到项目范围（与团队共享）
claude plugin install formatter@my-marketplace --scope project

# 安装到本地范围（gitignored）
claude plugin install formatter@my-marketplace --scope local
```

### plugin uninstall

删除已安装的插件。

```bash  theme={null}
claude plugin uninstall <plugin> [options]
```

**参数：**

* `<plugin>`：插件名称或 `plugin-name@marketplace-name`

**选项：**

| 选项                    | 描述                               | 默认值    |
| :-------------------- | :------------------------------- | :----- |
| `-s, --scope <scope>` | 从范围卸载：`user`、`project` 或 `local` | `user` |
| `-h, --help`          | 显示命令帮助                           |        |

**别名：** `remove`、`rm`

### plugin enable

启用已禁用的插件。

```bash  theme={null}
claude plugin enable <plugin> [options]
```

**参数：**

* `<plugin>`：插件名称或 `plugin-name@marketplace-name`

**选项：**

| 选项                    | 描述                              | 默认值    |
| :-------------------- | :------------------------------ | :----- |
| `-s, --scope <scope>` | 启用范围：`user`、`project` 或 `local` | `user` |
| `-h, --help`          | 显示命令帮助                          |        |

### plugin disable

禁用插件而不卸载它。

```bash  theme={null}
claude plugin disable <plugin> [options]
```

**参数：**

* `<plugin>`：插件名称或 `plugin-name@marketplace-name`

**选项：**

| 选项                    | 描述                              | 默认值    |
| :-------------------- | :------------------------------ | :----- |
| `-s, --scope <scope>` | 禁用范围：`user`、`project` 或 `local` | `user` |
| `-h, --help`          | 显示命令帮助                          |        |

### plugin update

将插件更新到最新版本。

```bash  theme={null}
claude plugin update <plugin> [options]
```

**参数：**

* `<plugin>`：插件名称或 `plugin-name@marketplace-name`

**选项：**

| 选项                    | 描述                                        | 默认值    |
| :-------------------- | :---------------------------------------- | :----- |
| `-s, --scope <scope>` | 更新范围：`user`、`project`、`local` 或 `managed` | `user` |
| `-h, --help`          | 显示命令帮助                                    |        |

***

## 调试和开发工具

### 调试命令

使用 `claude --debug`（或 TUI 中的 `/debug`）查看插件加载详情：

这显示：

* 正在加载哪些插件
* 插件清单中的任何错误
* 命令、agent 和 hook 注册
* MCP 服务器初始化

### 常见问题

| 问题                                  | 原因                         | 解决方案                                                               |
| :---------------------------------- | :------------------------- | :----------------------------------------------------------------- |
| 插件未加载                               | 无效的 `plugin.json`          | 使用 `claude plugin validate` 或 `/plugin validate` 验证 JSON 语法        |
| 命令未出现                               | 目录结构错误                     | 确保 `commands/` 在根目录，而不是在 `.claude-plugin/` 中                       |
| Hooks 未触发                           | 脚本不可执行                     | 运行 `chmod +x script.sh`                                            |
| MCP 服务器失败                           | 缺少 `${CLAUDE_PLUGIN_ROOT}` | 对所有插件路径使用变量                                                        |
| 路径错误                                | 使用了绝对路径                    | 所有路径必须是相对的并以 `./` 开头                                               |
| LSP `Executable not found in $PATH` | 未安装语言服务器                   | 安装二进制文件（例如 `npm install -g typescript-language-server typescript`） |

### 示例错误消息

**清单验证错误**：

* `Invalid JSON syntax: Unexpected token } in JSON at position 142`：检查缺少的逗号、多余的逗号或未引用的字符串
* `Plugin has an invalid manifest file at .claude-plugin/plugin.json. Validation errors: name: Required`：缺少必需字段
* `Plugin has a corrupt manifest file at .claude-plugin/plugin.json. JSON parse error: ...`：JSON 语法错误

**插件加载错误**：

* `Warning: No commands found in plugin my-plugin custom directory: ./cmds. Expected .md files or SKILL.md in subdirectories.`：命令路径存在但不包含有效的命令文件
* `Plugin directory not found at path: ./plugins/my-plugin. Check that the marketplace entry has the correct path.`：marketplace.json 中的 `source` 路径指向不存在的目录
* `Plugin my-plugin has conflicting manifests: both plugin.json and marketplace entry specify components.`：删除重复的组件定义或在 marketplace 条目中删除 `strict: false`

### Hook 故障排除

**Hook 脚本未执行**：

1. 检查脚本是否可执行：`chmod +x ./scripts/your-script.sh`
2. 验证 shebang 行：第一行应该是 `#!/bin/bash` 或 `#!/usr/bin/env bash`
3. 检查路径是否使用 `${CLAUDE_PLUGIN_ROOT}`：`"command": "${CLAUDE_PLUGIN_ROOT}/scripts/your-script.sh"`
4. 手动测试脚本：`./scripts/your-script.sh`

**Hook 未在预期事件上触发**：

1. 验证事件名称是否正确（区分大小写）：`PostToolUse`，而不是 `postToolUse`
2. 检查匹配器模式是否与您的工具匹配：`"matcher": "Write|Edit"` 用于文件操作
3. 确认 hook 类型有效：`command`、`prompt` 或 `agent`

### MCP 服务器故障排除

**服务器未启动**：

1. 检查命令是否存在且可执行
2. 验证所有路径是否使用 `${CLAUDE_PLUGIN_ROOT}` 变量
3. 检查 MCP 服务器日志：`claude --debug` 显示初始化错误
4. 在 Claude Code 外手动测试服务器

**服务器工具未出现**：

1. 确保服务器在 `.mcp.json` 或 `plugin.json` 中正确配置
2. 验证服务器是否正确实现 MCP 协议
3. 检查调试输出中的连接超时

### 目录结构错误

**症状**：插件加载但组件（命令、agents、hooks）缺失。

**正确结构**：组件必须在插件根目录，而不是在 `.claude-plugin/` 内。只有 `plugin.json` 属于 `.claude-plugin/`。

```
my-plugin/
├── .claude-plugin/
│   └── plugin.json      ← 仅清单在此
├── commands/            ← 在根级别
├── agents/              ← 在根级别
└── hooks/               ← 在根级别
```

如果您的组件在 `.claude-plugin/` 内，请将它们移到插件根目录。

**调试清单**：

1. 运行 `claude --debug` 并查找"loading plugin"消息
2. 检查每个组件目录是否在调试输出中列出
3. 验证文件权限允许读取插件文件

***

## 分发和版本管理参考

### 版本管理

遵循语义版本控制进行插件发布：

```json  theme={null}
{
  "name": "my-plugin",
  "version": "2.1.0"
}
```

**版本格式**：`MAJOR.MINOR.PATCH`

* **MAJOR**：破坏性更改（不兼容的 API 更改）
* **MINOR**：新功能（向后兼容的添加）
* **PATCH**：错误修复（向后兼容的修复）

**最佳实践**：

* 从 `1.0.0` 开始进行第一个稳定版本
* 在分发更改之前更新 `plugin.json` 中的版本
* 在 `CHANGELOG.md` 文件中记录更改
* 使用预发布版本，如 `2.0.0-beta.1` 进行测试

<Warning>
  Claude Code 使用版本来确定是否更新您的插件。如果您更改了插件的代码但没有在 `plugin.json` 中提升版本，您的插件的现有用户由于缓存不会看到您的更改。

  如果您的插件在[市场](/zh-CN/plugin-marketplaces)目录中，您可以通过 `marketplace.json` 管理版本，而不是从 `plugin.json` 中省略 `version` 字段。
</Warning>

***

## 另请参阅

* [插件](/zh-CN/plugins) - 教程和实际用法
* [插件市场](/zh-CN/plugin-marketplaces) - 创建和管理市场
* [Skills](/zh-CN/skills) - Skill 开发详情
* [Subagents](/zh-CN/sub-agents) - Agent 配置和功能
* [Hooks](/zh-CN/hooks) - 事件处理和自动化
* [MCP](/zh-CN/mcp) - 外部工具集成
* [设置](/zh-CN/settings) - 插件的配置选项
