

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

https://code.claude.com/docs/zh-CN/permissions.md

https://code.claude.com/docs/zh-CN/common-workflows.md



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

https://code.claude.com/docs/zh-CN/permissions

https://code.claude.com/docs/zh-CN/common-workflows



## Claude Code 设置



使用全局和项目级设置以及环境变量配置 Claude Code。

Claude Code 提供多种设置来配置其行为以满足您的需求。您可以在使用交互式 REPL 时运行 `/config` 命令来配置 Claude Code，这会打开一个选项卡式设置界面，您可以在其中查看状态信息并修改配置选项。

### [配置作用域](https://code.claude.com/docs/zh-CN/settings#配置作用域)

Claude Code 使用**作用域系统**来确定配置应用的位置以及与谁共享。了解作用域可以帮助您决定如何为个人使用、团队协作或企业部署配置 Claude Code。

**[可用作用域](https://code.claude.com/docs/zh-CN/settings#可用作用域)**

| 作用域      | 位置                                                         | 影响范围               | 与团队共享？     |
| :---------- | :----------------------------------------------------------- | :--------------------- | :--------------- |
| **Managed** | 服务器管理的设置、plist / 注册表或系统级 `managed-settings.json` | 机器上的所有用户       | 是（由 IT 部署） |
| **User**    | `~/.claude/` 目录                                            | 您，跨所有项目         | 否               |
| **Project** | 存储库中的 `.claude/`                                        | 此存储库上的所有协作者 | 是（提交到 git） |
| **Local**   | `.claude/*.local.*` 文件                                     | 您，仅在此存储库中     | 否（gitignored） |

**[何时使用每个作用域](https://code.claude.com/docs/zh-CN/settings#何时使用每个作用域)**

**Managed 作用域**用于：

- 必须在整个组织范围内强制执行的安全策略
- 无法被覆盖的合规要求
- 由 IT/DevOps 部署的标准化配置

**User 作用域**最适合：

- 您想在任何地方使用的个人偏好设置（主题、编辑器设置）
- 您在所有项目中使用的工具和 plugins
- API 密钥和身份验证（安全存储）

**Project 作用域**最适合：

- 团队共享的设置（权限、hooks、MCP servers）
- 整个团队应该拥有的 plugins
- 跨协作者标准化工具

**Local 作用域**最适合：

- 特定项目的个人覆盖
- 在与团队共享之前测试配置
- 对其他人不适用的特定于机器的设置

**[作用域如何相互作用](https://code.claude.com/docs/zh-CN/settings#作用域如何相互作用)**

当在多个作用域中配置相同的设置时，更具体的作用域优先：

1. **Managed**（最高）- 无法被任何内容覆盖
2. **命令行参数** - 临时会话覆盖
3. **Local** - 覆盖项目和用户设置
4. **Project** - 覆盖用户设置
5. **User**（最低）- 当没有其他内容指定设置时应用

例如，如果在用户设置中允许某个权限，但在项目设置中拒绝，则项目设置优先，权限被阻止。

**[哪些功能使用作用域](https://code.claude.com/docs/zh-CN/settings#哪些功能使用作用域)**

作用域适用于许多 Claude Code 功能：

| 功能            | 用户位置                  | 项目位置                           | 本地位置                      |
| :-------------- | :------------------------ | :--------------------------------- | :---------------------------- |
| **Settings**    | `~/.claude/settings.json` | `.claude/settings.json`            | `.claude/settings.local.json` |
| **Subagents**   | `~/.claude/agents/`       | `.claude/agents/`                  | —                             |
| **MCP servers** | `~/.claude.json`          | `.mcp.json`                        | `~/.claude.json`（每个项目）  |
| **Plugins**     | `~/.claude/settings.json` | `.claude/settings.json`            | `.claude/settings.local.json` |
| **CLAUDE.md**   | `~/.claude/CLAUDE.md`     | `CLAUDE.md` 或 `.claude/CLAUDE.md` | `CLAUDE.local.md`             |



### [可用设置](https://code.claude.com/docs/zh-CN/settings#可用设置)

`settings.json` 支持多个选项：

| 键                                | 描述                                                         | 示例                                                         |
| :-------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `apiKeyHelper`                    | 自定义脚本，在 `/bin/sh` 中执行，以生成身份验证值。此值将作为 `X-Api-Key` 和 `Authorization: Bearer` 标头发送用于模型请求 | `/bin/generate_temp_api_key.sh`                              |
| `cleanupPeriodDays`               | 非活跃超过此期间的会话在启动时被删除。设置为 `0` 立即删除所有会话。（默认：30 天） | `20`                                                         |
| `companyAnnouncements`            | 在启动时显示给用户的公告。如果提供多个公告，它们将随机循环显示。 | `["Welcome to Acme Corp! Review our code guidelines at docs.acme.com"]` |
| `env`                             | 将应用于每个会话的环境变量                                   | `{"FOO": "bar"}`                                             |
| `attribution`                     | 自定义 git 提交和拉取请求的归属。请参阅[归属设置](https://code.claude.com/docs/zh-CN/settings#attribution-settings) | `{"commit": "🤖 Generated with Claude Code", "pr": ""}`       |
| `includeCoAuthoredBy`             | **已弃用**：改用 `attribution`。是否在 git 提交和拉取请求中包含 `co-authored-by Claude` 署名（默认：`true`） | `false`                                                      |
| `permissions`                     | 有关权限结构，请参阅下表。                                   |                                                              |
| `hooks`                           | 配置自定义命令以在生命周期事件处运行。有关格式，请参阅 [hooks 文档](https://code.claude.com/docs/zh-CN/hooks) | 请参阅 [hooks](https://code.claude.com/docs/zh-CN/hooks)     |
| `disableAllHooks`                 | 禁用所有 [hooks](https://code.claude.com/docs/zh-CN/hooks) 和任何自定义[状态行](https://code.claude.com/docs/zh-CN/statusline) | `true`                                                       |
| `allowManagedHooksOnly`           | （Managed 设置仅） 防止加载用户、项目和 plugin hooks。仅允许 managed hooks 和 SDK hooks。请参阅[Hook 配置](https://code.claude.com/docs/zh-CN/settings#hook-configuration) | `true`                                                       |
| `allowManagedPermissionRulesOnly` | （Managed 设置仅） 防止用户和项目设置定义 `allow`、`ask` 或 `deny` 权限规则。仅应用 managed 设置中的规则。请参阅[Managed 专用设置](https://code.claude.com/docs/zh-CN/permissions#managed-only-settings) | `true`                                                       |
| `allowManagedMcpServersOnly`      | （Managed 设置仅） 仅尊重 managed 设置中的 `allowedMcpServers`。`deniedMcpServers` 仍从所有源合并。用户仍可以添加 MCP servers，但仅应用管理员定义的允许列表。请参阅[Managed MCP 配置](https://code.claude.com/docs/zh-CN/mcp#managed-mcp-configuration) | `true`                                                       |
| `model`                           | 覆盖 Claude Code 使用的默认模型                              | `"claude-sonnet-4-6"`                                        |
| `availableModels`                 | 限制用户可以通过 `/model`、`--model`、Config 工具或 `ANTHROPIC_MODEL` 选择的模型。不影响默认选项。请参阅[限制模型选择](https://code.claude.com/docs/zh-CN/model-config#restrict-model-selection) | `["sonnet", "haiku"]`                                        |
| `otelHeadersHelper`               | 生成动态 OpenTelemetry 标头的脚本。在启动时和定期运行（请参阅[动态标头](https://code.claude.com/docs/zh-CN/monitoring-usage#dynamic-headers)） | `/bin/generate_otel_headers.sh`                              |
| `statusLine`                      | 配置自定义状态行以显示上下文。请参阅[`statusLine` 文档](https://code.claude.com/docs/zh-CN/statusline) | `{"type": "command", "command": "~/.claude/statusline.sh"}`  |
| `fileSuggestion`                  | 为 `@` 文件自动完成配置自定义脚本。请参阅[文件建议设置](https://code.claude.com/docs/zh-CN/settings#file-suggestion-settings) | `{"type": "command", "command": "~/.claude/file-suggestion.sh"}` |
| `respectGitignore`                | 控制 `@` 文件选择器是否尊重 `.gitignore` 模式。当为 `true`（默认）时，匹配 `.gitignore` 模式的文件被排除在建议之外 | `false`                                                      |
| `outputStyle`                     | 配置输出样式以调整系统提示。请参阅[输出样式文档](https://code.claude.com/docs/zh-CN/output-styles) | `"Explanatory"`                                              |
| `forceLoginMethod`                | 使用 `claudeai` 限制登录到 Claude.ai 账户，`console` 限制登录到 Claude Console（API 使用计费）账户 | `claudeai`                                                   |
| `forceLoginOrgUUID`               | 指定组织的 UUID 以在登录期间自动选择它，绕过组织选择步骤。需要设置 `forceLoginMethod` | `"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"`                     |
| `enableAllProjectMcpServers`      | 自动批准项目 `.mcp.json` 文件中定义的所有 MCP servers        | `true`                                                       |
| `enabledMcpjsonServers`           | 要批准的 `.mcp.json` 文件中特定 MCP servers 的列表           | `["memory", "github"]`                                       |
| `disabledMcpjsonServers`          | 要拒绝的 `.mcp.json` 文件中特定 MCP servers 的列表           | `["filesystem"]`                                             |
| `allowedMcpServers`               | 在 managed-settings.json 中设置时，用户可以配置的 MCP servers 的允许列表。未定义 = 无限制，空数组 = 锁定。适用于所有作用域。拒绝列表优先。请参阅[Managed MCP 配置](https://code.claude.com/docs/zh-CN/mcp#managed-mcp-configuration) | `[{ "serverName": "github" }]`                               |
| `deniedMcpServers`                | 在 managed-settings.json 中设置时，明确阻止的 MCP servers 的拒绝列表。适用于所有作用域，包括 managed servers。拒绝列表优先于允许列表。请参阅[Managed MCP 配置](https://code.claude.com/docs/zh-CN/mcp#managed-mcp-configuration) | `[{ "serverName": "filesystem" }]`                           |
| `strictKnownMarketplaces`         | 在 managed-settings.json 中设置时，用户可以添加的 plugin marketplaces 的允许列表。未定义 = 无限制，空数组 = 锁定。仅适用于 marketplace 添加。请参阅[Managed marketplace 限制](https://code.claude.com/docs/zh-CN/plugin-marketplaces#managed-marketplace-restrictions) | `[{ "source": "github", "repo": "acme-corp/plugins" }]`      |
| `blockedMarketplaces`             | （Managed 设置仅） marketplace 源的阻止列表。在下载前检查被阻止的源，因此它们永远不会接触文件系统。请参阅[Managed marketplace 限制](https://code.claude.com/docs/zh-CN/plugin-marketplaces#managed-marketplace-restrictions) | `[{ "source": "github", "repo": "untrusted/plugins" }]`      |
| `awsAuthRefresh`                  | 修改 `.aws` 目录的自定义脚本（请参阅[高级凭证配置](https://code.claude.com/docs/zh-CN/amazon-bedrock#advanced-credential-configuration)） | `aws sso login --profile myprofile`                          |
| `awsCredentialExport`             | 输出包含 AWS 凭证的 JSON 的自定义脚本（请参阅[高级凭证配置](https://code.claude.com/docs/zh-CN/amazon-bedrock#advanced-credential-configuration)） | `/bin/generate_aws_grant.sh`                                 |
| `alwaysThinkingEnabled`           | 为所有会话默认启用[扩展思考](https://code.claude.com/docs/zh-CN/common-workflows#use-extended-thinking-thinking-mode)。通常通过 `/config` 命令而不是直接编辑来配置 | `true`                                                       |
| `plansDirectory`                  | 自定义计划文件的存储位置。路径相对于项目根目录。默认：`~/.claude/plans` | `"./plans"`                                                  |
| `showTurnDuration`                | 在响应后显示轮次持续时间消息（例如”Cooked for 1m 6s”）。设置为 `false` 以隐藏这些消息 | `true`                                                       |
| `spinnerVerbs`                    | 自定义在 spinner 和轮次持续时间消息中显示的操作动词。将 `mode` 设置为 `"replace"` 以仅使用您的动词，或 `"append"` 以将它们添加到默认值 | `{"mode": "append", "verbs": ["Pondering", "Crafting"]}`     |
| `language`                        | 配置 Claude 的首选响应语言（例如 `"japanese"`、`"spanish"`、`"french"`）。Claude 将默认以此语言响应 | `"japanese"`                                                 |
| `autoUpdatesChannel`              | 遵循更新的发布渠道。使用 `"stable"` 获取通常约一周前的版本并跳过有重大回归的版本，或使用 `"latest"`（默认）获取最新版本 | `"stable"`                                                   |
| `spinnerTipsEnabled`              | 在 Claude 工作时在 spinner 中显示提示。设置为 `false` 以禁用提示（默认：`true`） | `false`                                                      |
| `spinnerTipsOverride`             | 使用自定义字符串覆盖 spinner 提示。`tips`：提示字符串数组。`excludeDefault`：如果为 `true`，仅显示自定义提示；如果为 `false` 或不存在，自定义提示与内置提示合并 | `{ "excludeDefault": true, "tips": ["Use our internal tool X"] }` |
| `terminalProgressBarEnabled`      | 启用终端进度条，在 Windows Terminal 和 iTerm2 等支持的终端中显示进度（默认：`true`） | `false`                                                      |
| `prefersReducedMotion`            | 减少或禁用 UI 动画（spinners、shimmer、flash 效果）以实现可访问性 | `true`                                                       |
| `teammateMode`                    | [agent team](https://code.claude.com/docs/zh-CN/agent-teams) 队友的显示方式：`auto`（在 tmux 或 iTerm2 中选择分割窗格，否则为进程内）、`in-process` 或 `tmux`。请参阅[设置 agent teams](https://code.claude.com/docs/zh-CN/agent-teams#set-up-agent-teams) | `"in-process"`                                               |

#### [权限设置](https://code.claude.com/docs/zh-CN/settings#权限设置)

| 键                             | 描述                                                         | 示例                                                         |
| :----------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `allow`                        | 允许工具使用的权限规则数组。有关模式匹配详细信息，请参阅下面的[权限规则语法](https://code.claude.com/docs/zh-CN/settings#permission-rule-syntax) | `[ "Bash(git diff *)" ]`                                     |
| `ask`                          | 要求在工具使用时确认的权限规则数组。有关详细信息，请参阅下面的[权限规则语法](https://code.claude.com/docs/zh-CN/settings#permission-rule-syntax) | `[ "Bash(git push *)" ]`                                     |
| `deny`                         | 拒绝工具使用的权限规则数组。使用此选项从 Claude Code 访问中排除敏感文件。请参阅[权限规则语法](https://code.claude.com/docs/zh-CN/settings#permission-rule-syntax)和[Bash 权限限制](https://code.claude.com/docs/zh-CN/permissions#tool-specific-permission-rules) | `[ "WebFetch", "Bash(curl *)", "Read(./.env)", "Read(./secrets/**)" ]` |
| `additionalDirectories`        | Claude 有权访问的其他[工作目录](https://code.claude.com/docs/zh-CN/permissions#working-directories) | `[ "../docs/" ]`                                             |
| `defaultMode`                  | 打开 Claude Code 时的默认[权限模式](https://code.claude.com/docs/zh-CN/permissions#permission-modes) | `"acceptEdits"`                                              |
| `disableBypassPermissionsMode` | 设置为 `"disable"` 以防止激活 `bypassPermissions` 模式。这禁用 `--dangerously-skip-permissions` 命令行标志。请参阅[managed 设置](https://code.claude.com/docs/zh-CN/permissions#managed-only-settings) | `"disable"`                                                  |

#### [权限规则语法](https://code.claude.com/docs/zh-CN/settings#权限规则语法)

权限规则遵循格式 `Tool` 或 `Tool(specifier)`。规则按顺序评估：首先是拒绝规则，然后是询问，最后是允许。第一个匹配的规则获胜。快速示例：

| 规则                           | 效果                             |
| :----------------------------- | :------------------------------- |
| `Bash`                         | 匹配所有 Bash 命令               |
| `Bash(npm run *)`              | 匹配以 `npm run` 开头的命令      |
| `Read(./.env)`                 | 匹配读取 `.env` 文件             |
| `WebFetch(domain:example.com)` | 匹配对 example.com 的 fetch 请求 |

有关完整的规则语法参考，包括通配符行为、Read、Edit、WebFetch、MCP 和 Task 规则的工具特定模式，以及 Bash 模式的安全限制，请参阅[权限规则语法](https://code.claude.com/docs/zh-CN/permissions#permission-rule-syntax)。

### 

### [Claude 可用的工具](https://code.claude.com/docs/zh-CN/settings#claude-可用的工具)

Claude Code 可以访问一组强大的工具，帮助它理解和修改您的代码库：

| 工具                | 描述                                                         | 需要权限 |
| :------------------ | :----------------------------------------------------------- | :------- |
| **AskUserQuestion** | 提出多选问题以收集需求或澄清歧义                             | 否       |
| **Bash**            | 在您的环境中执行 shell 命令（请参阅下面的[Bash 工具行为](https://code.claude.com/docs/zh-CN/settings#bash-tool-behavior)） | 是       |
| **TaskOutput**      | 从后台任务（bash shell 或 subagent）检索输出                 | 否       |
| **Edit**            | 对特定文件进行有针对性的编辑                                 | 是       |
| **ExitPlanMode**    | 提示用户退出 Plan Mode 并开始编码                            | 是       |
| **Glob**            | 基于模式匹配查找文件                                         | 否       |
| **Grep**            | 在文件内容中搜索模式                                         | 否       |
| **KillShell**       | 按其 ID 杀死正在运行的后台 bash shell                        | 否       |
| **MCPSearch**       | 当启用[工具搜索](https://code.claude.com/docs/zh-CN/mcp#scale-with-mcp-tool-search)时搜索和加载 MCP 工具 | 否       |
| **NotebookEdit**    | 修改 Jupyter notebook 单元格                                 | 是       |
| **Read**            | 读取文件的内容                                               | 否       |
| **Skill**           | 在主对话中执行[skill](https://code.claude.com/docs/zh-CN/skills#control-who-invokes-a-skill) | 是       |
| **Task**            | 运行 sub-agent 以处理复杂的多步骤任务                        | 否       |
| **TaskCreate**      | 在任务列表中创建新任务                                       | 否       |
| **TaskGet**         | 检索特定任务的完整详细信息                                   | 否       |
| **TaskList**        | 列出所有任务及其当前状态                                     | 否       |
| **TaskUpdate**      | 更新任务状态、依赖项、详细信息或删除任务                     | 否       |
| **WebFetch**        | 从指定的 URL 获取内容                                        | 是       |
| **WebSearch**       | 执行带有域过滤的网络搜索                                     | 是       |
| **Write**           | 创建或覆盖文件                                               | 是       |
| **LSP**             | 通过语言服务器的代码智能。在文件编辑后自动报告类型错误和警告。还支持导航操作：跳转到定义、查找引用、获取类型信息、列出符号、查找实现、跟踪调用层次结构。需要[代码智能 plugin](https://code.claude.com/docs/zh-CN/discover-plugins#code-intelligence)及其语言服务器二进制文件 | 否       |

权限规则可以使用 `/allowed-tools` 或在[权限设置](https://code.claude.com/docs/zh-CN/settings#available-settings)中配置。另请参阅[工具特定权限规则](https://code.claude.com/docs/zh-CN/permissions#tool-specific-permission-rules)。

#### [Bash 工具行为](https://code.claude.com/docs/zh-CN/settings#bash-工具行为)

Bash 工具执行 shell 命令，具有以下持久性行为：

- **工作目录持久化**：当 Claude 更改工作目录（例如 `cd /path/to/dir`）时，后续 Bash 命令将在该目录中执行。您可以使用 `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR=1` 在每个命令后重置为项目目录。
- **环境变量不持久化**：在一个 Bash 命令中设置的环境变量（例如 `export MY_VAR=value`）**不**在后续 Bash 命令中可用。每个 Bash 命令在新的 shell 环境中运行。

要使环境变量在 Bash 命令中可用，您有**三个选项**：**选项 1：在启动 Claude Code 之前激活环境**（最简单的方法）在启动 Claude Code 之前在您的终端中激活您的虚拟环境：

```
conda activate myenv
# 或：source /path/to/venv/bin/activate
claude
```

这适用于 shell 环境，但在 Claude 的 Bash 命令中设置的环境变量不会在命令之间持久化。**选项 2：在启动 Claude Code 之前设置 CLAUDE_ENV_FILE**（持久环境设置）导出包含您的环境设置的 shell 脚本的路径：

```
export CLAUDE_ENV_FILE=/path/to/env-setup.sh
claude
```

其中 `/path/to/env-setup.sh` 包含：

```
conda activate myenv
# 或：source /path/to/venv/bin/activate
# 或：export MY_VAR=value
```

Claude Code 将在每个 Bash 命令之前获取此文件，使环境在所有命令中持久化。**选项 3：使用 SessionStart hook**（项目特定配置）在 `.claude/settings.json` 中配置：

```
{
  "hooks": {
    "SessionStart": [{
      "matcher": "startup",
      "hooks": [{
        "type": "command",
        "command": "echo 'conda activate myenv' >> \"$CLAUDE_ENV_FILE\""
      }]
    }]
  }
}
```

hook 写入 `$CLAUDE_ENV_FILE`，然后在每个 Bash 命令之前获取。这对于团队共享的项目配置很理想。有关选项 3 的更多详细信息，请参阅[SessionStart hooks](https://code.claude.com/docs/zh-CN/hooks#persist-environment-variables)。



## Plugin

### Skills

**任务内容**为 Claude 提供特定操作的分步说明，如部署、提交或代码生成。这些通常是你想使用 `/skill-name` 直接调用的操作，而不是让 Claude 决定何时运行它们。添加 `disable-model-invocation: true` 以防止 Claude 自动触发它。



#### [Frontmatter 参考](https://code.claude.com/docs/zh-CN/skills#frontmatter-参考)

除了 markdown 内容外，你可以使用 `SKILL.md` 文件顶部 `---` 标记之间的 YAML frontmatter 字段来配置 skill 行为：

```
---
name: my-skill
description: What this skill does
disable-model-invocation: true
allowed-tools: Read, Grep
---

Your skill instructions here...
```

所有字段都是可选的。建议使用 `description`，以便 Claude 知道何时使用该 skill。

| 字段                       | 必需 | 描述                                                         |
| :------------------------- | :--- | :----------------------------------------------------------- |
| `name`                     | 否   | Skill 的显示名称。如果省略，使用目录名称。仅小写字母、数字和连字符（最多 64 个字符）。 |
| `description`              | 推荐 | Skill 的功能以及何时使用它。Claude 使用这个来决定何时应用该 skill。如果省略，使用 markdown 内容的第一段。 |
| `argument-hint`            | 否   | 自动完成期间显示的提示，指示预期的参数。示例：`[issue-number]` 或 `[filename] [format]`。 |
| `disable-model-invocation` | 否   | 设置为 `true` 以防止 Claude 自动加载此 skill。用于你想使用 `/name` 手动触发的工作流。默认值：`false`。 |
| `user-invocable`           | 否   | 设置为 `false` 以从 `/` 菜单中隐藏。用于用户不应直接调用的背景知识。默认值：`true`。 |
| `allowed-tools`            | 否   | 当此 skill 处于活动状态时，Claude 可以使用而无需请求权限的工具。 |
| `model`                    | 否   | 当此 skill 处于活动状态时要使用的模型。                      |
| `context`                  | 否   | 设置为 `fork` 以在分叉的 subagent 上下文中运行。             |
| `agent`                    | 否   | 当设置 `context: fork` 时要使用的 subagent 类型。            |
| `hooks`                    | 否   | 限定于此 skill 生命周期的 hooks。有关配置格式，请参阅 [Skills 和 agents 中的 Hooks](https://code.claude.com/docs/zh-CN/hooks#hooks-in-skills-and-agents)。 |

##### [可用的字符串替换](https://code.claude.com/docs/zh-CN/skills#可用的字符串替换)

Skills 支持 skill 内容中动态值的字符串替换：

| 变量                   | 描述                                                         |
| :--------------------- | :----------------------------------------------------------- |
| `$ARGUMENTS`           | 调用 skill 时传递的所有参数。如果 `$ARGUMENTS` 不在内容中，参数作为 `ARGUMENTS: <value>` 追加。 |
| `$ARGUMENTS[N]`        | 按 0 基索引访问特定参数，如 `$ARGUMENTS[0]` 表示第一个参数。 |
| `$N`                   | `$ARGUMENTS[N]` 的简写，如 `$0` 表示第一个参数或 `$1` 表示第二个参数。 |
| `${CLAUDE_SESSION_ID}` | 当前会话 ID。用于日志记录、创建会话特定文件或将 skill 输出与会话关联。 |

**使用替换的示例：**

```markdown
---
name: session-logger
description: Log activity for this session
---

Log the following to logs/${CLAUDE_SESSION_ID}.log:

$ARGUMENTS
```

### 

从 `SKILL.md` 中引用支持文件，以便 Claude 知道每个文件包含什么以及何时加载它：

```markdown
## Additional resources

- For complete API details, see [reference.md](reference.md)
- For usage examples, see [examples.md](examples.md)
```





#### [限制工具访问](https://code.claude.com/docs/zh-CN/skills#限制工具访问)

使用 `allowed-tools` 字段来限制当 skill 处于活动状态时 Claude 可以使用哪些工具。此 skill 创建了一个只读模式，其中 Claude 可以浏览文件但不能修改它们：

```
---
name: safe-reader
description: Read files without making changes
allowed-tools: Read, Grep, Glob
---
```

#### [将参数传递给 skills](https://code.claude.com/docs/zh-CN/skills#将参数传递给-skills)

你和 Claude 都可以在调用 skill 时传递参数。参数可通过 `$ARGUMENTS` 占位符获得。此 skill 通过编号修复 GitHub 问题。`$ARGUMENTS` 占位符被替换为 skill 名称后面的任何内容：

```
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
---

Fix GitHub issue $ARGUMENTS following our coding standards.

1. Read the issue description
2. Understand the requirements
3. Implement the fix
4. Write tests
5. Create a commit
```

当你运行 `/fix-issue 123` 时，Claude 收到”Fix GitHub issue **123** following our coding standards…”如果你使用参数调用 skill 但 skill 不包含 `$ARGUMENTS`，Claude Code 会将 `ARGUMENTS: <your input>` 追加到 skill 内容的末尾，以便 Claude 仍然看到你输入的内容。要按位置访问单个参数，使用 `$ARGUMENTS[N]` 或较短的 `$N`：

```
---
name: migrate-component
description: Migrate a component from one framework to another
---

Migrate the $ARGUMENTS[0] component from $ARGUMENTS[1] to $ARGUMENTS[2].
Preserve all existing behavior and tests.
```

运行 `/migrate-component SearchBar React Vue` 会将 `$ARGUMENTS[0]` 替换为 `SearchBar`，`$ARGUMENTS[1]` 替换为 `React`，`$ARGUMENTS[2]` 替换为 `Vue`。使用 `$N` 简写的相同 skill：

```
---
name: migrate-component
description: Migrate a component from one framework to another
---

Migrate the $0 component from $1 to $2.
Preserve all existing behavior and tests.
```



#### [注入动态上下文](https://code.claude.com/docs/zh-CN/skills#注入动态上下文)

`!`command` “ 语法在 skill 内容发送给 Claude 之前运行 shell 命令。命令输出替换占位符，因此 Claude 接收实际数据，而不是命令本身。此 skill 通过使用 GitHub CLI 获取实时 PR 数据来总结拉取请求。` !`gh pr diff` “ 和其他命令首先运行，它们的输出被插入到提示中：

```
---
name: pr-summary
description: Summarize changes in a pull request
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## Pull request context
- PR diff: !`gh pr diff`
- PR comments: !`gh pr view --comments`
- Changed files: !`gh pr diff --name-only`

## Your task
Summarize this pull request...
```

当此 skill 运行时：

1. 每个 `!`command` “ 立即执行（在 Claude 看到任何东西之前）
2. 输出替换 skill 内容中的占位符
3. Claude 接收具有实际 PR 数据的完全渲染的提示

这是预处理，不是 Claude 执行的东西。Claude 只看到最终结果。



要在 skill 中启用[扩展思考](https://code.claude.com/docs/zh-CN/common-workflows#use-extended-thinking-thinking-mode)，在你的 skill 内容中的任何地方包含单词”ultrathink”。

### 

### Subagent

#### [使用专门的 subagents](https://code.claude.com/docs/zh-CN/common-workflows#使用专门的-subagents)

假设您想使用专门的 AI subagents 来更有效地处理特定任务。

1. 查看可用的 subagents

```
> /agents
```

这显示所有可用的 subagents 并让您创建新的。

2. 自动使用 subagents

Claude Code 自动将适当的任务委派给专门的 subagents：

```
> review my recent code changes for security issues
> run all tests and fix any failures
```

3. 明确请求特定的 subagents

```
> use the code-reviewer subagent to check the auth module
> have the debugger subagent investigate why users can't log in
```

4. 为您的工作流程创建自定义 subagents

```
> /agents
```

然后选择”Create New subagent”并按照提示定义：

- 描述 subagent 目的的唯一标识符（例如，`code-reviewer`、`api-designer`）。
- Claude 何时应使用此 agent
- 它可以访问哪些工具
- 描述 agent 角色和行为的系统提示



提示：

- 在 `.claude/agents/` 中创建项目特定的 subagents 以供团队共享
- 使用描述性的 `description` 字段来启用自动委派
- 限制工具访问权限为每个 subagent 实际需要的内容
- 查看[subagents 文档](https://code.claude.com/docs/zh-CN/sub-agents)了解详细示例



#### [使用 Plan Mode 进行安全的代码分析](https://code.claude.com/docs/zh-CN/common-workflows#使用-plan-mode-进行安全的代码分析)

Plan Mode 指示 Claude 通过使用只读操作分析代码库来创建计划，非常适合探索代码库、规划复杂更改或安全地审查代码。在 Plan Mode 中，Claude 使用 [`AskUserQuestion`](https://code.claude.com/docs/zh-CN/settings#tools-available-to-claude) 来收集需求并在提出计划之前澄清您的目标。

**[何时使用 Plan Mode](https://code.claude.com/docs/zh-CN/common-workflows#何时使用-plan-mode)**

- **多步骤实现**：当您的功能需要编辑许多文件时
- **代码探索**：当您想在更改任何内容之前彻底研究代码库时
- **交互式开发**：当您想与 Claude 迭代方向时

**[如何使用 Plan Mode](https://code.claude.com/docs/zh-CN/common-workflows#如何使用-plan-mode)**

**在会话期间打开 Plan Mode**您可以在会话期间使用 **Shift+Tab** 循环切换权限模式来切换到 Plan Mode。如果您处于 Normal Mode，**Shift+Tab** 首先切换到 Auto-Accept Mode，在终端底部显示 `⏵⏵ accept edits on`。随后的 **Shift+Tab** 将切换到 Plan Mode，显示 `⏸ plan mode on`。当[agent team](https://code.claude.com/docs/zh-CN/agent-teams)处于活动状态时，循环还包括 Delegate Mode。**在 Plan Mode 中启动新会话**要在 Plan Mode 中启动新会话，请使用 `--permission-mode plan` 标志：

```
claude --permission-mode plan
```

**在 Plan Mode 中运行”无头”查询**您也可以直接在 Plan Mode 中使用 `-p` 运行查询（即在[”无头模式”](https://code.claude.com/docs/zh-CN/headless)中）：

```
claude --permission-mode plan -p "Analyze the authentication system and suggest improvements"
```

**[示例：规划复杂的重构](https://code.claude.com/docs/zh-CN/common-workflows#示例：规划复杂的重构)**

```
claude --permission-mode plan
> I need to refactor our authentication system to use OAuth2. Create a detailed migration plan.
```

Claude 分析当前实现并创建全面的计划。通过后续问题进行细化：

```
> What about backward compatibility?
> How should we handle database migration?
```



按 `Ctrl+G` 在默认文本编辑器中打开计划，您可以在 Claude 继续之前直接编辑它。

**[将 Plan Mode 配置为默认值](https://code.claude.com/docs/zh-CN/common-workflows#将-plan-mode-配置为默认值)**

```
// .claude/settings.json
{
  "permissions": {
    "defaultMode": "plan"
  }
}
```

有关更多配置选项，请参阅[设置文档](https://code.claude.com/docs/zh-CN/settings#available-settings)。





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