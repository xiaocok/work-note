

# 官方文档

https://code.claude.com/docs/llms.txt



## Claude Code 文档导航图

https://code.claude.com/docs/zh-CN/claude_code_docs_map.md





官方的原始markdown

https://code.claude.com/docs/zh-CN/hooks-guide.md

https://code.claude.com/docs/zh-CN/sub-agents.md

https://code.claude.com/docs/zh-CN/plugins.md

https://code.claude.com/docs/zh-CN/plugins-reference.md

https://code.claude.com/docs/zh-CN/discover-plugins.md

https://code.claude.com/docs/zh-CN/skills.md

https://code.claude.com/docs/zh-CN/mcp.md

https://code.claude.com/docs/zh-CN/cli-reference.md

https://code.claude.com/docs/zh-CN/interactive-mode.md



官方地址

https://code.claude.com/docs/zh-CN/hooks-guide

https://code.claude.com/docs/zh-CN/sub-agents

https://code.claude.com/docs/zh-CN/plugins

https://code.claude.com/docs/zh-CN/plugins-reference

https://code.claude.com/docs/zh-CN/discover-plugins

https://code.claude.com/docs/zh-CN/skills

https://code.claude.com/docs/zh-CN/mcp

https://code.claude.com/docs/zh-CN/cli-reference

https://code.claude.com/docs/zh-CN/interactive-mode



## Plugin

### [Hooks](https://code.claude.com/docs/zh-CN/plugins-reference#hooks)

插件可以提供事件处理程序，自动响应 Claude Code 事件。**位置**：插件根目录中的 `hooks/hooks.json`，或在 plugin.json 中内联**格式**：具有事件匹配器和操作的 JSON 配置**Hook 配置**：

```
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

- `PreToolUse`：Claude 使用任何工具之前
- `PostToolUse`：Claude 成功使用任何工具之后
- `PostToolUseFailure`：Claude 工具执行失败之后
- `PermissionRequest`：显示权限对话框时
- `UserPromptSubmit`：用户提交提示时
- `Notification`：Claude Code 发送通知时
- `Stop`：Claude 尝试停止时
- `SubagentStart`：subagent 启动时
- `SubagentStop`：subagent 尝试停止时
- `SessionStart`：会话开始时
- `SessionEnd`：会话结束时
- `TeammateIdle`：agent 团队队友即将空闲时
- `TaskCompleted`：任务被标记为已完成时
- `PreCompact`：对话历史被压缩之前

**Hook 类型**：

- `command`：执行 shell 命令或脚本
- `prompt`：使用 LLM 评估提示（使用 `$ARGUMENTS` 占位符表示上下文）
- `agent`：运行具有工具的 agentic 验证器以完成复杂验证任务



### [MCP servers](https://code.claude.com/docs/zh-CN/plugins-reference#mcp-servers)

插件可以捆绑 Model Context Protocol (MCP) servers 以将 Claude Code 与外部工具和服务连接。**位置**：插件根目录中的 `.mcp.json`，或在 plugin.json 中内联**格式**：标准 MCP 服务器配置**MCP 服务器配置**：

```
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

- 启用插件时，插件 MCP servers 会自动启动
- Servers 在 Claude 的工具包中显示为标准 MCP 工具
- 服务器功能与 Claude 的现有工具无缝集成
- 插件 servers 可以独立于用户 MCP servers 进行配置



#### [`.mcp.json` 中的环境变量扩展](https://code.claude.com/docs/zh-CN/mcp#mcp-json-中的环境变量扩展)

Claude Code 支持 `.mcp.json` 文件中的环境变量扩展，允许团队共享配置，同时为特定于机器的路径和 API 密钥等敏感值保持灵活性。**支持的语法：**

- `${VAR}` - 扩展为环境变量 `VAR` 的值
- `${VAR:-default}` - 如果设置了 `VAR`，则扩展为 `VAR`，否则使用 `default`

**扩展位置：** 环境变量可以在以下位置扩展：

- `command` - 服务器可执行文件路径
- `args` - 命令行参数
- `env` - 传递给服务器的环境变量
- `url` - 对于 HTTP 服务器类型
- `headers` - 对于 HTTP 服务器身份验证

**带有变量扩展的示例：**

```
{
  "mcpServers": {
    "api-server": {
      "type": "http",
      "url": "${API_BASE_URL:-https://api.example.com}/mcp",
      "headers": {
        "Authorization": "Bearer ${API_KEY}"
      }
    }
  }
}
```

如果未设置必需的环境变量且没有默认值，Claude Code 将无法解析配置。



### [插件安装范围](https://code.claude.com/docs/zh-CN/plugins-reference#插件安装范围)

安装插件时，您选择一个**范围**来确定插件的可用位置以及谁可以使用它：

| 范围      | 设置文件                                                     | 用例                               |
| :-------- | :----------------------------------------------------------- | :--------------------------------- |
| `user`    | `~/.claude/settings.json`                                    | 在所有项目中可用的个人插件（默认） |
| `project` | `.claude/settings.json`                                      | 通过版本控制共享的团队插件         |
| `local`   | `.claude/settings.local.json`                                | 项目特定的插件，gitignored         |
| `managed` | [托管设置](https://code.claude.com/docs/zh-CN/settings#settings-files) | 托管插件（只读，仅更新）           |

插件使用与其他 Claude Code 配置相同的范围系统。有关安装说明和范围标志，请参阅[安装插件](https://code.claude.com/docs/zh-CN/discover-plugins#install-plugins)。有关范围的完整说明，请参阅[配置范围](https://code.claude.com/docs/zh-CN/settings#configuration-scopes)。

------

### [插件清单架构](https://code.claude.com/docs/zh-CN/plugins-reference#插件清单架构)

`.claude-plugin/plugin.json` 文件定义了您的插件的元数据和配置。本部分记录所有支持的字段和选项。清单是可选的。如果省略，Claude Code 会自动发现[默认位置](https://code.claude.com/docs/zh-CN/plugins-reference#file-locations-reference)中的组件，并从目录名称派生插件名称。当您需要提供元数据或自定义组件路径时，使用清单。

##### [完整架构](https://code.claude.com/docs/zh-CN/plugins-reference#完整架构)

```
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

##### [必需字段](https://code.claude.com/docs/zh-CN/plugins-reference#必需字段)

如果包含清单，`name` 是唯一必需的字段。

| 字段   | 类型   | 描述                             | 示例                 |
| :----- | :----- | :------------------------------- | :------------------- |
| `name` | string | 唯一标识符（kebab-case，无空格） | `"deployment-tools"` |

此名称用于命名空间组件。例如，在 UI 中，名为 `plugin-dev` 的插件的 agent `agent-creator` 将显示为 `plugin-dev:agent-creator`。

##### [元数据字段](https://code.claude.com/docs/zh-CN/plugins-reference#元数据字段)

| 字段          | 类型   | 描述                                                         | 示例                                               |
| :------------ | :----- | :----------------------------------------------------------- | :------------------------------------------------- |
| `version`     | string | 语义版本。如果也在市场条目中设置，`plugin.json` 优先。您只需在一个地方设置它。 | `"2.1.0"`                                          |
| `description` | string | 插件目的的简要说明                                           | `"部署自动化工具"`                                 |
| `author`      | object | 作者信息                                                     | `{"name": "Dev Team", "email": "dev@company.com"}` |
| `homepage`    | string | 文档 URL                                                     | `"https://docs.example.com"`                       |
| `repository`  | string | 源代码 URL                                                   | `"https://github.com/user/plugin"`                 |
| `license`     | string | 许可证标识符                                                 | `"MIT"`、`"Apache-2.0"`                            |
| `keywords`    | array  | 发现标签                                                     | `["deployment", "ci-cd"]`                          |

##### [组件路径字段](https://code.claude.com/docs/zh-CN/plugins-reference#组件路径字段)

| 字段           | 类型                  | 描述                                                         | 示例                                   |
| :------------- | :-------------------- | :----------------------------------------------------------- | :------------------------------------- |
| `commands`     | string\|array         | 其他命令文件/目录                                            | `"./custom/cmd.md"` 或 `["./cmd1.md"]` |
| `agents`       | string\|array         | 其他 agent 文件                                              | `"./custom/agents/reviewer.md"`        |
| `skills`       | string\|array         | 其他 skill 目录                                              | `"./custom/skills/"`                   |
| `hooks`        | string\|array\|object | Hook 配置路径或内联配置                                      | `"./my-extra-hooks.json"`              |
| `mcpServers`   | string\|array\|object | MCP 配置路径或内联配置                                       | `"./my-extra-mcp-config.json"`         |
| `outputStyles` | string\|array         | 其他输出样式文件/目录                                        | `"./styles/"`                          |
| `lspServers`   | string\|array\|object | [Language Server Protocol](https://microsoft.github.io/language-server-protocol/) 配置用于代码智能（转到定义、查找引用等） | `"./.lsp.json"`                        |

##### [路径行为规则](https://code.claude.com/docs/zh-CN/plugins-reference#路径行为规则)

**重要**：自定义路径补充默认目录 - 它们不替换默认目录。

- 如果 `commands/` 存在，它会与自定义命令路径一起加载
- 所有路径必须相对于插件根目录并以 `./` 开头
- 来自自定义路径的命令使用相同的命名和命名空间规则
- 可以将多个路径指定为数组以获得灵活性

**路径示例**：

```
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

##### [环境变量](https://code.claude.com/docs/zh-CN/plugins-reference#环境变量)

**`${CLAUDE_PLUGIN_ROOT}`**：包含插件目录的绝对路径。在 hooks、MCP servers 和脚本中使用此变量，以确保无论安装位置如何都能正确路径。

```
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



###  插件目录结构

#### [标准插件布局](https://code.claude.com/docs/zh-CN/plugins-reference#标准插件布局)

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



`.claude-plugin/` 目录包含 `plugin.json` 文件。所有其他目录（commands/、agents/、skills/、hooks/）必须在插件根目录，而不是在 `.claude-plugin/` 内。

#### [文件位置参考](https://code.claude.com/docs/zh-CN/plugins-reference#文件位置参考)

| 组件            | 默认位置                     | 目的                                                         |
| :-------------- | :--------------------------- | :----------------------------------------------------------- |
| **清单**        | `.claude-plugin/plugin.json` | 插件元数据和配置（可选）                                     |
| **命令**        | `commands/`                  | Skill Markdown 文件（旧版；新 skills 使用 `skills/`）        |
| **Agents**      | `agents/`                    | Subagent Markdown 文件                                       |
| **Skills**      | `skills/`                    | 具有 `<name>/SKILL.md` 结构的 Skills                         |
| **Hooks**       | `hooks/hooks.json`           | Hook 配置                                                    |
| **MCP servers** | `.mcp.json`                  | MCP 服务器定义                                               |
| **LSP servers** | `.lsp.json`                  | 语言服务器配置                                               |
| **设置**        | `settings.json`              | 启用插件时应用的默认配置。目前仅支持 [`agent`](https://code.claude.com/docs/zh-CN/sub-agents) 设置 |