> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# Claude Code 设置

> 使用全局和项目级设置以及环境变量配置 Claude Code。

Claude Code 提供多种设置来配置其行为以满足您的需求。您可以在使用交互式 REPL 时运行 `/config` 命令来配置 Claude Code，这会打开一个选项卡式设置界面，您可以在其中查看状态信息并修改配置选项。

## 配置作用域

Claude Code 使用**作用域系统**来确定配置应用的位置以及与谁共享。了解作用域可以帮助您决定如何为个人使用、团队协作或企业部署配置 Claude Code。

### 可用作用域

| 作用域         | 位置                                               | 影响范围        | 与团队共享？        |
| :---------- | :----------------------------------------------- | :---------- | :------------ |
| **Managed** | 服务器管理的设置、plist / 注册表或系统级 `managed-settings.json` | 机器上的所有用户    | 是（由 IT 部署）    |
| **User**    | `~/.claude/` 目录                                  | 您，跨所有项目     | 否             |
| **Project** | 存储库中的 `.claude/`                                 | 此存储库上的所有协作者 | 是（提交到 git）    |
| **Local**   | `.claude/*.local.*` 文件                           | 您，仅在此存储库中   | 否（gitignored） |

### 何时使用每个作用域

**Managed 作用域**用于：

* 必须在整个组织范围内强制执行的安全策略
* 无法被覆盖的合规要求
* 由 IT/DevOps 部署的标准化配置

**User 作用域**最适合：

* 您想在任何地方使用的个人偏好设置（主题、编辑器设置）
* 您在所有项目中使用的工具和 plugins
* API 密钥和身份验证（安全存储）

**Project 作用域**最适合：

* 团队共享的设置（权限、hooks、MCP servers）
* 整个团队应该拥有的 plugins
* 跨协作者标准化工具

**Local 作用域**最适合：

* 特定项目的个人覆盖
* 在与团队共享之前测试配置
* 对其他人不适用的特定于机器的设置

### 作用域如何相互作用

当在多个作用域中配置相同的设置时，更具体的作用域优先：

1. **Managed**（最高）- 无法被任何内容覆盖
2. **命令行参数** - 临时会话覆盖
3. **Local** - 覆盖项目和用户设置
4. **Project** - 覆盖用户设置
5. **User**（最低）- 当没有其他内容指定设置时应用

例如，如果在用户设置中允许某个权限，但在项目设置中拒绝，则项目设置优先，权限被阻止。

### 哪些功能使用作用域

作用域适用于许多 Claude Code 功能：

| 功能              | 用户位置                      | 项目位置                              | 本地位置                          |
| :-------------- | :------------------------ | :-------------------------------- | :---------------------------- |
| **Settings**    | `~/.claude/settings.json` | `.claude/settings.json`           | `.claude/settings.local.json` |
| **Subagents**   | `~/.claude/agents/`       | `.claude/agents/`                 | —                             |
| **MCP servers** | `~/.claude.json`          | `.mcp.json`                       | `~/.claude.json`（每个项目）        |
| **Plugins**     | `~/.claude/settings.json` | `.claude/settings.json`           | `.claude/settings.local.json` |
| **CLAUDE.md**   | `~/.claude/CLAUDE.md`     | `CLAUDE.md` 或 `.claude/CLAUDE.md` | `CLAUDE.local.md`             |

***

## 设置文件

`settings.json` 文件是我们用于通过分层设置配置 Claude Code 的官方机制：

* **用户设置**在 `~/.claude/settings.json` 中定义，适用于所有项目。
* **项目设置**保存在您的项目目录中：
  * `.claude/settings.json` 用于检入源代码管理并与您的团队共享的设置
  * `.claude/settings.local.json` 用于未检入的设置，适用于个人偏好和实验。Claude Code 将在创建 `.claude/settings.local.json` 时配置 git 以忽略它。
* **Managed 设置**：对于需要集中控制的组织，Claude Code 支持多种 managed 设置的交付机制。所有机制都使用相同的 JSON 格式，无法被用户或项目设置覆盖：

  * **服务器管理的设置**：通过 Claude.ai 管理员控制台从 Anthropic 的服务器交付。请参阅[服务器管理的设置](/zh-CN/server-managed-settings)。
  * **MDM/OS 级别策略**：通过 macOS 和 Windows 上的本机设备管理交付：
    * macOS：`com.anthropic.claudecode` 管理首选项域（通过 Jamf、Kandji 或其他 MDM 工具中的配置文件部署）
    * Windows：`HKLM\SOFTWARE\Policies\ClaudeCode` 注册表项，带有包含 JSON 的 `Settings` 值（REG\_SZ 或 REG\_EXPAND\_SZ）（通过组策略或 Intune 部署）
    * Windows（用户级别）：`HKCU\SOFTWARE\Policies\ClaudeCode`（最低策略优先级，仅在不存在管理员级别源时使用）
  * **基于文件**：`managed-settings.json` 和 `managed-mcp.json` 部署到系统目录：
    * macOS：`/Library/Application Support/ClaudeCode/`
    * Linux 和 WSL：`/etc/claude-code/`
    * Windows：`C:\Program Files\ClaudeCode\`

  有关详细信息，请参阅[managed 设置](/zh-CN/permissions#managed-only-settings)和[Managed MCP 配置](/zh-CN/mcp#managed-mcp-configuration)。

  <Note>
    Managed 部署也可以使用 `strictKnownMarketplaces` 限制**plugin marketplace 添加**。有关更多信息，请参阅[Managed marketplace 限制](/zh-CN/plugin-marketplaces#managed-marketplace-restrictions)。
  </Note>
* **其他配置**存储在 `~/.claude.json` 中。此文件包含您的偏好设置（主题、通知设置、编辑器模式）、OAuth 会话、[MCP server](/zh-CN/mcp) 配置（用于用户和本地作用域）、每个项目的状态（允许的工具、信任设置）和各种缓存。项目作用域的 MCP servers 单独存储在 `.mcp.json` 中。

<Note>
  Claude Code 自动创建配置文件的时间戳备份，并保留最近的五个备份以防止数据丢失。
</Note>

```JSON Example settings.json theme={null}
{
  "$schema": "https://json.schemastore.org/claude-code-settings.json",
  "permissions": {
    "allow": [
      "Bash(npm run lint)",
      "Bash(npm run test *)",
      "Read(~/.zshrc)"
    ],
    "deny": [
      "Bash(curl *)",
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)"
    ]
  },
  "env": {
    "CLAUDE_CODE_ENABLE_TELEMETRY": "1",
    "OTEL_METRICS_EXPORTER": "otlp"
  },
  "companyAnnouncements": [
    "Welcome to Acme Corp! Review our code guidelines at docs.acme.com",
    "Reminder: Code reviews required for all PRs",
    "New security policy in effect"
  ]
}
```

上面示例中的 `$schema` 行指向 Claude Code 设置的[官方 JSON schema](https://json.schemastore.org/claude-code-settings.json)。将其添加到您的 `settings.json` 可在 VS Code、Cursor 和任何其他支持 JSON schema 验证的编辑器中启用自动完成和内联验证。

### 可用设置

`settings.json` 支持多个选项：

| 键                                 | 描述                                                                                                                                                                                             | 示例                                                                      |
| :-------------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :---------------------------------------------------------------------- |
| `apiKeyHelper`                    | 自定义脚本，在 `/bin/sh` 中执行，以生成身份验证值。此值将作为 `X-Api-Key` 和 `Authorization: Bearer` 标头发送用于模型请求                                                                                                          | `/bin/generate_temp_api_key.sh`                                         |
| `cleanupPeriodDays`               | 非活跃超过此期间的会话在启动时被删除。设置为 `0` 立即删除所有会话。（默认：30 天）                                                                                                                                                  | `20`                                                                    |
| `companyAnnouncements`            | 在启动时显示给用户的公告。如果提供多个公告，它们将随机循环显示。                                                                                                                                                               | `["Welcome to Acme Corp! Review our code guidelines at docs.acme.com"]` |
| `env`                             | 将应用于每个会话的环境变量                                                                                                                                                                                  | `{"FOO": "bar"}`                                                        |
| `attribution`                     | 自定义 git 提交和拉取请求的归属。请参阅[归属设置](#attribution-settings)                                                                                                                                            | `{"commit": "🤖 Generated with Claude Code", "pr": ""}`                 |
| `includeCoAuthoredBy`             | **已弃用**：改用 `attribution`。是否在 git 提交和拉取请求中包含 `co-authored-by Claude` 署名（默认：`true`）                                                                                                              | `false`                                                                 |
| `permissions`                     | 有关权限结构，请参阅下表。                                                                                                                                                                                  |                                                                         |
| `hooks`                           | 配置自定义命令以在生命周期事件处运行。有关格式，请参阅 [hooks 文档](/zh-CN/hooks)                                                                                                                                           | 请参阅 [hooks](/zh-CN/hooks)                                               |
| `disableAllHooks`                 | 禁用所有 [hooks](/zh-CN/hooks) 和任何自定义[状态行](/zh-CN/statusline)                                                                                                                                      | `true`                                                                  |
| `allowManagedHooksOnly`           | （Managed 设置仅） 防止加载用户、项目和 plugin hooks。仅允许 managed hooks 和 SDK hooks。请参阅[Hook 配置](#hook-configuration)                                                                                          | `true`                                                                  |
| `allowManagedPermissionRulesOnly` | （Managed 设置仅） 防止用户和项目设置定义 `allow`、`ask` 或 `deny` 权限规则。仅应用 managed 设置中的规则。请参阅[Managed 专用设置](/zh-CN/permissions#managed-only-settings)                                                           | `true`                                                                  |
| `allowManagedMcpServersOnly`      | （Managed 设置仅） 仅尊重 managed 设置中的 `allowedMcpServers`。`deniedMcpServers` 仍从所有源合并。用户仍可以添加 MCP servers，但仅应用管理员定义的允许列表。请参阅[Managed MCP 配置](/zh-CN/mcp#managed-mcp-configuration)                     | `true`                                                                  |
| `model`                           | 覆盖 Claude Code 使用的默认模型                                                                                                                                                                         | `"claude-sonnet-4-6"`                                                   |
| `availableModels`                 | 限制用户可以通过 `/model`、`--model`、Config 工具或 `ANTHROPIC_MODEL` 选择的模型。不影响默认选项。请参阅[限制模型选择](/zh-CN/model-config#restrict-model-selection)                                                               | `["sonnet", "haiku"]`                                                   |
| `otelHeadersHelper`               | 生成动态 OpenTelemetry 标头的脚本。在启动时和定期运行（请参阅[动态标头](/zh-CN/monitoring-usage#dynamic-headers)）                                                                                                         | `/bin/generate_otel_headers.sh`                                         |
| `statusLine`                      | 配置自定义状态行以显示上下文。请参阅[`statusLine` 文档](/zh-CN/statusline)                                                                                                                                         | `{"type": "command", "command": "~/.claude/statusline.sh"}`             |
| `fileSuggestion`                  | 为 `@` 文件自动完成配置自定义脚本。请参阅[文件建议设置](#file-suggestion-settings)                                                                                                                                     | `{"type": "command", "command": "~/.claude/file-suggestion.sh"}`        |
| `respectGitignore`                | 控制 `@` 文件选择器是否尊重 `.gitignore` 模式。当为 `true`（默认）时，匹配 `.gitignore` 模式的文件被排除在建议之外                                                                                                                  | `false`                                                                 |
| `outputStyle`                     | 配置输出样式以调整系统提示。请参阅[输出样式文档](/zh-CN/output-styles)                                                                                                                                                | `"Explanatory"`                                                         |
| `forceLoginMethod`                | 使用 `claudeai` 限制登录到 Claude.ai 账户，`console` 限制登录到 Claude Console（API 使用计费）账户                                                                                                                    | `claudeai`                                                              |
| `forceLoginOrgUUID`               | 指定组织的 UUID 以在登录期间自动选择它，绕过组织选择步骤。需要设置 `forceLoginMethod`                                                                                                                                        | `"xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"`                                |
| `enableAllProjectMcpServers`      | 自动批准项目 `.mcp.json` 文件中定义的所有 MCP servers                                                                                                                                                        | `true`                                                                  |
| `enabledMcpjsonServers`           | 要批准的 `.mcp.json` 文件中特定 MCP servers 的列表                                                                                                                                                         | `["memory", "github"]`                                                  |
| `disabledMcpjsonServers`          | 要拒绝的 `.mcp.json` 文件中特定 MCP servers 的列表                                                                                                                                                         | `["filesystem"]`                                                        |
| `allowedMcpServers`               | 在 managed-settings.json 中设置时，用户可以配置的 MCP servers 的允许列表。未定义 = 无限制，空数组 = 锁定。适用于所有作用域。拒绝列表优先。请参阅[Managed MCP 配置](/zh-CN/mcp#managed-mcp-configuration)                                            | `[{ "serverName": "github" }]`                                          |
| `deniedMcpServers`                | 在 managed-settings.json 中设置时，明确阻止的 MCP servers 的拒绝列表。适用于所有作用域，包括 managed servers。拒绝列表优先于允许列表。请参阅[Managed MCP 配置](/zh-CN/mcp#managed-mcp-configuration)                                         | `[{ "serverName": "filesystem" }]`                                      |
| `strictKnownMarketplaces`         | 在 managed-settings.json 中设置时，用户可以添加的 plugin marketplaces 的允许列表。未定义 = 无限制，空数组 = 锁定。仅适用于 marketplace 添加。请参阅[Managed marketplace 限制](/zh-CN/plugin-marketplaces#managed-marketplace-restrictions) | `[{ "source": "github", "repo": "acme-corp/plugins" }]`                 |
| `blockedMarketplaces`             | （Managed 设置仅） marketplace 源的阻止列表。在下载前检查被阻止的源，因此它们永远不会接触文件系统。请参阅[Managed marketplace 限制](/zh-CN/plugin-marketplaces#managed-marketplace-restrictions)                                           | `[{ "source": "github", "repo": "untrusted/plugins" }]`                 |
| `awsAuthRefresh`                  | 修改 `.aws` 目录的自定义脚本（请参阅[高级凭证配置](/zh-CN/amazon-bedrock#advanced-credential-configuration)）                                                                                                       | `aws sso login --profile myprofile`                                     |
| `awsCredentialExport`             | 输出包含 AWS 凭证的 JSON 的自定义脚本（请参阅[高级凭证配置](/zh-CN/amazon-bedrock#advanced-credential-configuration)）                                                                                                 | `/bin/generate_aws_grant.sh`                                            |
| `alwaysThinkingEnabled`           | 为所有会话默认启用[扩展思考](/zh-CN/common-workflows#use-extended-thinking-thinking-mode)。通常通过 `/config` 命令而不是直接编辑来配置                                                                                       | `true`                                                                  |
| `plansDirectory`                  | 自定义计划文件的存储位置。路径相对于项目根目录。默认：`~/.claude/plans`                                                                                                                                                   | `"./plans"`                                                             |
| `showTurnDuration`                | 在响应后显示轮次持续时间消息（例如"Cooked for 1m 6s"）。设置为 `false` 以隐藏这些消息                                                                                                                                       | `true`                                                                  |
| `spinnerVerbs`                    | 自定义在 spinner 和轮次持续时间消息中显示的操作动词。将 `mode` 设置为 `"replace"` 以仅使用您的动词，或 `"append"` 以将它们添加到默认值                                                                                                       | `{"mode": "append", "verbs": ["Pondering", "Crafting"]}`                |
| `language`                        | 配置 Claude 的首选响应语言（例如 `"japanese"`、`"spanish"`、`"french"`）。Claude 将默认以此语言响应                                                                                                                     | `"japanese"`                                                            |
| `autoUpdatesChannel`              | 遵循更新的发布渠道。使用 `"stable"` 获取通常约一周前的版本并跳过有重大回归的版本，或使用 `"latest"`（默认）获取最新版本                                                                                                                        | `"stable"`                                                              |
| `spinnerTipsEnabled`              | 在 Claude 工作时在 spinner 中显示提示。设置为 `false` 以禁用提示（默认：`true`）                                                                                                                                       | `false`                                                                 |
| `spinnerTipsOverride`             | 使用自定义字符串覆盖 spinner 提示。`tips`：提示字符串数组。`excludeDefault`：如果为 `true`，仅显示自定义提示；如果为 `false` 或不存在，自定义提示与内置提示合并                                                                                        | `{ "excludeDefault": true, "tips": ["Use our internal tool X"] }`       |
| `terminalProgressBarEnabled`      | 启用终端进度条，在 Windows Terminal 和 iTerm2 等支持的终端中显示进度（默认：`true`）                                                                                                                                     | `false`                                                                 |
| `prefersReducedMotion`            | 减少或禁用 UI 动画（spinners、shimmer、flash 效果）以实现可访问性                                                                                                                                                  | `true`                                                                  |
| `teammateMode`                    | [agent team](/zh-CN/agent-teams) 队友的显示方式：`auto`（在 tmux 或 iTerm2 中选择分割窗格，否则为进程内）、`in-process` 或 `tmux`。请参阅[设置 agent teams](/zh-CN/agent-teams#set-up-agent-teams)                               | `"in-process"`                                                          |

### 权限设置

| 键                              | 描述                                                                                                                                                | 示例                                                                     |
| :----------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------ | :--------------------------------------------------------------------- |
| `allow`                        | 允许工具使用的权限规则数组。有关模式匹配详细信息，请参阅下面的[权限规则语法](#permission-rule-syntax)                                                                                  | `[ "Bash(git diff *)" ]`                                               |
| `ask`                          | 要求在工具使用时确认的权限规则数组。有关详细信息，请参阅下面的[权限规则语法](#permission-rule-syntax)                                                                                  | `[ "Bash(git push *)" ]`                                               |
| `deny`                         | 拒绝工具使用的权限规则数组。使用此选项从 Claude Code 访问中排除敏感文件。请参阅[权限规则语法](#permission-rule-syntax)和[Bash 权限限制](/zh-CN/permissions#tool-specific-permission-rules)    | `[ "WebFetch", "Bash(curl *)", "Read(./.env)", "Read(./secrets/**)" ]` |
| `additionalDirectories`        | Claude 有权访问的其他[工作目录](/zh-CN/permissions#working-directories)                                                                                      | `[ "../docs/" ]`                                                       |
| `defaultMode`                  | 打开 Claude Code 时的默认[权限模式](/zh-CN/permissions#permission-modes)                                                                                    | `"acceptEdits"`                                                        |
| `disableBypassPermissionsMode` | 设置为 `"disable"` 以防止激活 `bypassPermissions` 模式。这禁用 `--dangerously-skip-permissions` 命令行标志。请参阅[managed 设置](/zh-CN/permissions#managed-only-settings) | `"disable"`                                                            |

### 权限规则语法

权限规则遵循格式 `Tool` 或 `Tool(specifier)`。规则按顺序评估：首先是拒绝规则，然后是询问，最后是允许。第一个匹配的规则获胜。

快速示例：

| 规则                             | 效果                         |
| :----------------------------- | :------------------------- |
| `Bash`                         | 匹配所有 Bash 命令               |
| `Bash(npm run *)`              | 匹配以 `npm run` 开头的命令        |
| `Read(./.env)`                 | 匹配读取 `.env` 文件             |
| `WebFetch(domain:example.com)` | 匹配对 example.com 的 fetch 请求 |

有关完整的规则语法参考，包括通配符行为、Read、Edit、WebFetch、MCP 和 Task 规则的工具特定模式，以及 Bash 模式的安全限制，请参阅[权限规则语法](/zh-CN/permissions#permission-rule-syntax)。

### Sandbox 设置

配置高级 sandboxing 行为。Sandboxing 将 bash 命令与您的文件系统和网络隔离。有关详细信息，请参阅[Sandboxing](/zh-CN/sandboxing)。

**文件系统和网络限制**通过 Read、Edit 和 WebFetch 权限规则配置，而不是通过这些 sandbox 设置。

| 键                                 | 描述                                                                                                                                                                                  | 示例                              |
| :-------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :------------------------------ |
| `enabled`                         | 启用 bash sandboxing（macOS、Linux 和 WSL2）。默认：false                                                                                                                                     | `true`                          |
| `autoAllowBashIfSandboxed`        | 当 sandboxed 时自动批准 bash 命令。默认：true                                                                                                                                                   | `true`                          |
| `excludedCommands`                | 应在 sandbox 外运行的命令                                                                                                                                                                   | `["git", "docker"]`             |
| `allowUnsandboxedCommands`        | 允许命令通过 `dangerouslyDisableSandbox` 参数在 sandbox 外运行。当设置为 `false` 时，`dangerouslyDisableSandbox` 逃生舱口完全禁用，所有命令必须 sandboxed（或在 `excludedCommands` 中）。对于需要严格 sandboxing 的企业策略很有用。默认：true | `false`                         |
| `network.allowUnixSockets`        | sandbox 中可访问的 Unix socket 路径（用于 SSH 代理等）                                                                                                                                            | `["~/.ssh/agent-socket"]`       |
| `network.allowAllUnixSockets`     | 允许 sandbox 中的所有 Unix socket 连接。默认：false                                                                                                                                             | `true`                          |
| `network.allowLocalBinding`       | 允许绑定到 localhost 端口（仅 macOS）。默认：false                                                                                                                                                | `true`                          |
| `network.allowedDomains`          | 允许出站网络流量的域数组。支持通配符（例如 `*.example.com`）。                                                                                                                                             | `["github.com", "*.npmjs.org"]` |
| `network.allowManagedDomainsOnly` | （Managed 设置仅） 仅尊重 managed 设置中的 `allowedDomains` 和 `WebFetch(domain:...)` 允许规则。来自用户、项目和本地设置的域被忽略。拒绝的域仍从所有源受尊重。默认：false                                                               | `true`                          |
| `network.httpProxyPort`           | 如果您想自带代理，则使用的 HTTP 代理端口。如果未指定，Claude 将运行自己的代理。                                                                                                                                      | `8080`                          |
| `network.socksProxyPort`          | 如果您想自带代理，则使用的 SOCKS5 代理端口。如果未指定，Claude 将运行自己的代理。                                                                                                                                    | `8081`                          |
| `enableWeakerNestedSandbox`       | 为无特权 Docker 环境启用较弱的 sandbox（仅 Linux 和 WSL2）。**降低安全性。** 默认：false                                                                                                                     | `true`                          |

**配置示例：**

```json  theme={null}
{
  "sandbox": {
    "enabled": true,
    "autoAllowBashIfSandboxed": true,
    "excludedCommands": ["docker"],
    "network": {
      "allowedDomains": ["github.com", "*.npmjs.org", "registry.yarnpkg.com"],
      "allowUnixSockets": [
        "/var/run/docker.sock"
      ],
      "allowLocalBinding": true
    }
  },
  "permissions": {
    "deny": [
      "Read(.envrc)",
      "Read(~/.aws/**)"
    ]
  }
}
```

**文件系统和网络限制**使用标准权限规则：

* 使用 `Read` 拒绝规则阻止 Claude 读取特定文件或目录
* 使用 `Edit` 允许规则让 Claude 写入当前工作目录之外的目录
* 使用 `Edit` 拒绝规则阻止写入特定路径
* 使用 `WebFetch` 允许/拒绝规则控制 Claude 可以访问哪些网络域

### 归属设置

Claude Code 为 git 提交和拉取请求添加归属。这些分别配置：

* 提交默认使用 [git trailers](https://git-scm.com/docs/git-interpret-trailers)（如 `Co-Authored-By`），可以自定义或禁用
* 拉取请求描述是纯文本

| 键        | 描述                                 |
| :------- | :--------------------------------- |
| `commit` | git 提交的归属，包括任何 trailers。空字符串隐藏提交归属 |
| `pr`     | 拉取请求描述的归属。空字符串隐藏拉取请求归属             |

**默认提交归属：**

```
🤖 Generated with [Claude Code](https://claude.com/claude-code)

   Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
```

**默认拉取请求归属：**

```
🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

**示例：**

```json  theme={null}
{
  "attribution": {
    "commit": "Generated with AI\n\nCo-Authored-By: AI <ai@example.com>",
    "pr": ""
  }
}
```

<Note>
  `attribution` 设置优先于已弃用的 `includeCoAuthoredBy` 设置。要隐藏所有归属，请将 `commit` 和 `pr` 设置为空字符串。
</Note>

### 文件建议设置

为 `@` 文件路径自动完成配置自定义命令。内置文件建议使用快速文件系统遍历，但大型 monorepos 可能受益于项目特定的索引，例如预构建的文件索引或自定义工具。

```json  theme={null}
{
  "fileSuggestion": {
    "type": "command",
    "command": "~/.claude/file-suggestion.sh"
  }
}
```

该命令使用与 [hooks](/zh-CN/hooks) 相同的环境变量运行，包括 `CLAUDE_PROJECT_DIR`。它通过 stdin 接收包含 `query` 字段的 JSON：

```json  theme={null}
{"query": "src/comp"}
```

将换行符分隔的文件路径输出到 stdout（当前限制为 15）：

```
src/components/Button.tsx
src/components/Modal.tsx
src/components/Form.tsx
```

**示例：**

```bash  theme={null}
#!/bin/bash
query=$(cat | jq -r '.query')
your-repo-file-index --query "$query" | head -20
```

### Hook 配置

**Managed 设置仅**：控制允许运行哪些 hooks。此设置只能在[managed 设置](#settings-files)中配置，为管理员提供对 hook 执行的严格控制。

**当 `allowManagedHooksOnly` 为 `true` 时的行为：**

* 加载 Managed hooks 和 SDK hooks
* 阻止用户 hooks、项目 hooks 和 plugin hooks

**配置：**

```json  theme={null}
{
  "allowManagedHooksOnly": true
}
```

### 设置优先级

设置按优先级顺序应用。从最高到最低：

1. **Managed 设置**（[服务器管理](/zh-CN/server-managed-settings)、[MDM/OS 级别策略](#configuration-scopes)或[managed 设置](/zh-CN/settings#settings-files)）
   * 由 IT 通过服务器交付、MDM 配置文件、注册表策略或 managed 设置文件部署的策略
   * 无法被用户或项目设置覆盖
   * 在 managed 层内，优先级为：server-managed > MDM/OS 级别策略 > `managed-settings.json` > HKCU 注册表（仅 Windows）。仅使用一个 managed 源；源不合并。

2. **命令行参数**
   * 特定会话的临时覆盖

3. **本地项目设置**（`.claude/settings.local.json`）
   * 个人项目特定设置

4. **共享项目设置**（`.claude/settings.json`）
   * 源代码管理中的团队共享项目设置

5. **用户设置**（`~/.claude/settings.json`）
   * 个人全局设置

此层次结构确保组织策略始终被强制执行，同时仍允许团队和个人自定义其体验。

例如，如果您的用户设置允许 `Bash(npm run *)`，但项目的共享设置拒绝它，则项目设置优先，命令被阻止。

### 验证活跃设置

在 Claude Code 中运行 `/status` 以查看哪些设置源处于活跃状态以及它们来自何处。输出显示每个配置层（managed、user、project）及其来源，例如 `Enterprise managed settings (remote)`、`Enterprise managed settings (plist)`、`Enterprise managed settings (HKLM)` 或 `Enterprise managed settings (file)`。如果设置文件包含错误，`/status` 会报告问题，以便您可以修复它。

### 关于配置系统的关键要点

* **内存文件（`CLAUDE.md`）**：包含 Claude 在启动时加载的说明和上下文
* **设置文件（JSON）**：配置权限、环境变量和工具行为
* **Skills**：可以使用 `/skill-name` 调用或由 Claude 自动加载的自定义提示
* **MCP servers**：使用其他工具和集成扩展 Claude Code
* **优先级**：更高级别的配置（Managed）覆盖更低级别的配置（User/Project）
* **继承**：设置被合并，更具体的设置添加到或覆盖更广泛的设置

### 系统提示

Claude Code 的内部系统提示未发布。要添加自定义说明，请使用 `CLAUDE.md` 文件或 `--append-system-prompt` 标志。

### 排除敏感文件

要防止 Claude Code 访问包含敏感信息（如 API 密钥、secrets 和环境文件）的文件，请在您的 `.claude/settings.json` 文件中使用 `permissions.deny` 设置：

```json  theme={null}
{
  "permissions": {
    "deny": [
      "Read(./.env)",
      "Read(./.env.*)",
      "Read(./secrets/**)",
      "Read(./config/credentials.json)",
      "Read(./build)"
    ]
  }
}
```

这替代了已弃用的 `ignorePatterns` 配置。匹配这些模式的文件被排除在文件发现和搜索结果之外，对这些文件的读取操作被拒绝。

## Subagent 配置

Claude Code 支持可在用户和项目级别配置的自定义 AI subagents。这些 subagents 存储为带有 YAML frontmatter 的 Markdown 文件：

* **用户 subagents**：`~/.claude/agents/` - 在所有项目中可用
* **项目 subagents**：`.claude/agents/` - 特定于您的项目，可与您的团队共享

Subagent 文件定义具有自定义提示和工具权限的专门 AI 助手。在 [subagents 文档](/zh-CN/sub-agents)中了解有关创建和使用 subagents 的更多信息。

## Plugin 配置

Claude Code 支持一个 plugin 系统，让您使用 skills、agents、hooks 和 MCP servers 扩展功能。Plugins 通过 marketplaces 分发，可在用户和存储库级别配置。

### Plugin 设置

`settings.json` 中与 plugin 相关的设置：

```json  theme={null}
{
  "enabledPlugins": {
    "formatter@acme-tools": true,
    "deployer@acme-tools": true,
    "analyzer@security-plugins": false
  },
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": "github",
      "repo": "acme-corp/claude-plugins"
    }
  }
}
```

#### `enabledPlugins`

控制启用哪些 plugins。格式：`"plugin-name@marketplace-name": true/false`

**作用域**：

* **用户设置**（`~/.claude/settings.json`）：个人 plugin 偏好
* **项目设置**（`.claude/settings.json`）：与团队共享的项目特定 plugins
* **本地设置**（`.claude/settings.local.json`）：每台机器的覆盖（未提交）

**示例**：

```json  theme={null}
{
  "enabledPlugins": {
    "code-formatter@team-tools": true,
    "deployment-tools@team-tools": true,
    "experimental-features@personal": false
  }
}
```

#### `extraKnownMarketplaces`

定义应为存储库提供的其他 marketplaces。通常在存储库级别设置中使用，以确保团队成员有权访问所需的 plugin 源。

**当存储库包含 `extraKnownMarketplaces` 时**：

1. 当团队成员信任该文件夹时，会提示他们安装 marketplace
2. 然后提示团队成员从该 marketplace 安装 plugins
3. 用户可以跳过不需要的 marketplaces 或 plugins（存储在用户设置中）
4. 安装尊重信任边界并需要明确同意

**示例**：

```json  theme={null}
{
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": {
        "source": "github",
        "repo": "acme-corp/claude-plugins"
      }
    },
    "security-plugins": {
      "source": {
        "source": "git",
        "url": "https://git.example.com/security/plugins.git"
      }
    }
  }
}
```

**Marketplace 源类型**：

* `github`：GitHub 存储库（使用 `repo`）
* `git`：任何 git URL（使用 `url`）
* `directory`：本地文件系统路径（使用 `path`，仅用于开发）
* `hostPattern`：regex 模式以匹配 marketplace 主机（使用 `hostPattern`）

#### `strictKnownMarketplaces`

**Managed 设置仅**：控制用户允许添加哪些 plugin marketplaces。此设置只能在[managed 设置](/zh-CN/settings#settings-files)中配置，为管理员提供对 marketplace 源的严格控制。

**Managed 设置文件位置**：

* **macOS**：`/Library/Application Support/ClaudeCode/managed-settings.json`
* **Linux 和 WSL**：`/etc/claude-code/managed-settings.json`
* **Windows**：`C:\Program Files\ClaudeCode\managed-settings.json`

**关键特征**：

* 仅在 managed 设置（`managed-settings.json`）中可用
* 无法被用户或项目设置覆盖（最高优先级）
* 在网络/文件系统操作之前强制执行（被阻止的源永远不会执行）
* 对源规范使用精确匹配（包括 git 源的 `ref`、`path`），除了 `hostPattern`，它使用 regex 匹配

**允许列表行为**：

* `undefined`（默认）：无限制 - 用户可以添加任何 marketplace
* 空数组 `[]`：完全锁定 - 用户无法添加任何新 marketplaces
* 源列表：用户只能添加完全匹配的 marketplaces

**所有支持的源类型**：

允许列表支持七种 marketplace 源类型。大多数源使用精确匹配，而 `hostPattern` 使用 regex 匹配 marketplace 主机。

1. **GitHub 存储库**：

```json  theme={null}
{ "source": "github", "repo": "acme-corp/approved-plugins" }
{ "source": "github", "repo": "acme-corp/security-tools", "ref": "v2.0" }
{ "source": "github", "repo": "acme-corp/plugins", "ref": "main", "path": "marketplace" }
```

字段：`repo`（必需）、`ref`（可选：分支/标签/SHA）、`path`（可选：子目录）

2. **Git 存储库**：

```json  theme={null}
{ "source": "git", "url": "https://gitlab.example.com/tools/plugins.git" }
{ "source": "git", "url": "https://bitbucket.org/acme-corp/plugins.git", "ref": "production" }
{ "source": "git", "url": "ssh://git@git.example.com/plugins.git", "ref": "v3.1", "path": "approved" }
```

字段：`url`（必需）、`ref`（可选：分支/标签/SHA）、`path`（可选：子目录）

3. **基于 URL 的 marketplaces**：

```json  theme={null}
{ "source": "url", "url": "https://plugins.example.com/marketplace.json" }
{ "source": "url", "url": "https://cdn.example.com/marketplace.json", "headers": { "Authorization": "Bearer ${TOKEN}" } }
```

字段：`url`（必需）、`headers`（可选：用于身份验证访问的 HTTP 标头）

<Note>
  基于 URL 的 marketplaces 仅下载 `marketplace.json` 文件。它们不从服务器下载 plugin 文件。基于 URL 的 marketplaces 中的 Plugins 必须使用外部源（GitHub、npm 或 git URL）而不是相对路径。对于具有相对路径的 plugins，改用基于 Git 的 marketplace。有关详细信息，请参阅[故障排除](/zh-CN/plugin-marketplaces#plugins-with-relative-paths-fail-in-url-based-marketplaces)。
</Note>

4. **NPM 包**：

```json  theme={null}
{ "source": "npm", "package": "@acme-corp/claude-plugins" }
{ "source": "npm", "package": "@acme-corp/approved-marketplace" }
```

字段：`package`（必需，支持作用域包）

5. **文件路径**：

```json  theme={null}
{ "source": "file", "path": "/usr/local/share/claude/acme-marketplace.json" }
{ "source": "file", "path": "/opt/acme-corp/plugins/marketplace.json" }
```

字段：`path`（必需：marketplace.json 文件的绝对路径）

6. **目录路径**：

```json  theme={null}
{ "source": "directory", "path": "/usr/local/share/claude/acme-plugins" }
{ "source": "directory", "path": "/opt/acme-corp/approved-marketplaces" }
```

字段：`path`（必需：包含 `.claude-plugin/marketplace.json` 的目录的绝对路径）

7. **主机模式匹配**：

```json  theme={null}
{ "source": "hostPattern", "hostPattern": "^github\\.example\\.com$" }
{ "source": "hostPattern", "hostPattern": "^gitlab\\.internal\\.example\\.com$" }
```

字段：`hostPattern`（必需：regex 模式以匹配 marketplace 主机）

当您想允许来自特定主机的所有 marketplaces 而不枚举每个存储库时，使用主机模式匹配。这对于具有内部 GitHub Enterprise 或 GitLab 服务器的组织很有用，开发人员在其中创建自己的 marketplaces。

按源类型的主机提取：

* `github`：始终匹配 `github.com`
* `git`：从 URL 提取主机名（支持 HTTPS 和 SSH 格式）
* `url`：从 URL 提取主机名
* `npm`、`file`、`directory`：不支持主机模式匹配

**配置示例**：

示例：仅允许特定 marketplaces：

```json  theme={null}
{
  "strictKnownMarketplaces": [
    {
      "source": "github",
      "repo": "acme-corp/approved-plugins"
    },
    {
      "source": "github",
      "repo": "acme-corp/security-tools",
      "ref": "v2.0"
    },
    {
      "source": "url",
      "url": "https://plugins.example.com/marketplace.json"
    },
    {
      "source": "npm",
      "package": "@acme-corp/compliance-plugins"
    }
  ]
}
```

示例 - 禁用所有 marketplace 添加：

```json  theme={null}
{
  "strictKnownMarketplaces": []
}
```

示例：允许来自内部 git 服务器的所有 marketplaces：

```json  theme={null}
{
  "strictKnownMarketplaces": [
    {
      "source": "hostPattern",
      "hostPattern": "^github\\.example\\.com$"
    }
  ]
}
```

**精确匹配要求**：

Marketplace 源必须**完全匹配**才能允许用户的添加。对于基于 git 的源（`github` 和 `git`），这包括所有可选字段：

* `repo` 或 `url` 必须完全匹配
* `ref` 字段必须完全匹配（或两者都未定义）
* `path` 字段必须完全匹配（或两者都未定义）

不匹配的源示例：

```json  theme={null}
// 这些是不同的源：
{ "source": "github", "repo": "acme-corp/plugins" }
{ "source": "github", "repo": "acme-corp/plugins", "ref": "main" }

// 这些也是不同的：
{ "source": "github", "repo": "acme-corp/plugins", "path": "marketplace" }
{ "source": "github", "repo": "acme-corp/plugins" }
```

**与 `extraKnownMarketplaces` 的比较**：

| 方面         | `strictKnownMarketplaces` | `extraKnownMarketplaces` |
| ---------- | ------------------------- | ------------------------ |
| **目的**     | 组织策略强制执行                  | 团队便利                     |
| **设置文件**   | 仅 `managed-settings.json` | 任何设置文件                   |
| **行为**     | 阻止非允许列表添加                 | 自动安装缺失的 marketplaces     |
| **何时强制执行** | 在网络/文件系统操作之前              | 在用户信任提示之后                |
| **可以被覆盖**  | 否（最高优先级）                  | 是（由更高优先级设置）              |
| **源格式**    | 直接源对象                     | 具有嵌套源的命名 marketplace     |
| **用例**     | 合规、安全限制                   | 入职、标准化                   |

**格式差异**：

`strictKnownMarketplaces` 使用直接源对象：

```json  theme={null}
{
  "strictKnownMarketplaces": [
    { "source": "github", "repo": "acme-corp/plugins" }
  ]
}
```

`extraKnownMarketplaces` 需要命名 marketplaces：

```json  theme={null}
{
  "extraKnownMarketplaces": {
    "acme-tools": {
      "source": { "source": "github", "repo": "acme-corp/plugins" }
    }
  }
}
```

**重要说明**：

* 限制在任何网络请求或文件系统操作之前检查
* 被阻止时，用户会看到清晰的错误消息，指示源被 managed 策略阻止
* 限制仅适用于添加新 marketplaces；以前安装的 marketplaces 保持可访问
* Managed 设置具有最高优先级，无法被覆盖

有关面向用户的文档，请参阅[Managed marketplace 限制](/zh-CN/plugin-marketplaces#managed-marketplace-restrictions)。

### 管理 plugins

使用 `/plugin` 命令以交互方式管理 plugins：

* 浏览来自 marketplaces 的可用 plugins
* 安装/卸载 plugins
* 启用/禁用 plugins
* 查看 plugin 详细信息（提供的命令、agents、hooks）
* 添加/删除 marketplaces

在 [plugins 文档](/zh-CN/plugins)中了解有关 plugin 系统的更多信息。

## 环境变量

Claude Code 支持以下环境变量来控制其行为：

<Note>
  所有环境变量也可以在 [`settings.json`](#available-settings) 中配置。这是为每个会话自动设置环境变量或为整个团队或组织推出一组环境变量的有用方式。
</Note>

| 变量                                             | 目的                                                                                                                                                                                                                                                                                  |     |
| :--------------------------------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
| `ANTHROPIC_API_KEY`                            | 作为 `X-Api-Key` 标头发送的 API 密钥，通常用于 Claude SDK（对于交互式使用，运行 `/login`）                                                                                                                                                                                                                    |     |
| `ANTHROPIC_AUTH_TOKEN`                         | `Authorization` 标头的自定义值（您在此处设置的值将以 `Bearer ` 为前缀）                                                                                                                                                                                                                                   |     |
| `ANTHROPIC_CUSTOM_HEADERS`                     | 要添加到请求的自定义标头（`Name: Value` 格式，多个标头用换行符分隔）                                                                                                                                                                                                                                           |     |
| `ANTHROPIC_DEFAULT_HAIKU_MODEL`                | 请参阅[模型配置](/zh-CN/model-config#environment-variables)                                                                                                                                                                                                                                |     |
| `ANTHROPIC_DEFAULT_OPUS_MODEL`                 | 请参阅[模型配置](/zh-CN/model-config#environment-variables)                                                                                                                                                                                                                                |     |
| `ANTHROPIC_DEFAULT_SONNET_MODEL`               | 请参阅[模型配置](/zh-CN/model-config#environment-variables)                                                                                                                                                                                                                                |     |
| `ANTHROPIC_FOUNDRY_API_KEY`                    | Microsoft Foundry 身份验证的 API 密钥（请参阅[Microsoft Foundry](/zh-CN/microsoft-foundry)）                                                                                                                                                                                                    |     |
| `ANTHROPIC_FOUNDRY_BASE_URL`                   | Foundry 资源的完整基础 URL（例如 `https://my-resource.services.ai.azure.com/anthropic`）。`ANTHROPIC_FOUNDRY_RESOURCE` 的替代方案（请参阅[Microsoft Foundry](/zh-CN/microsoft-foundry)）                                                                                                                  |     |
| `ANTHROPIC_FOUNDRY_RESOURCE`                   | Foundry 资源名称（例如 `my-resource`）。如果未设置 `ANTHROPIC_FOUNDRY_BASE_URL`，则为必需（请参阅[Microsoft Foundry](/zh-CN/microsoft-foundry)）                                                                                                                                                            |     |
| `ANTHROPIC_MODEL`                              | 要使用的模型设置的名称（请参阅[模型配置](/zh-CN/model-config#environment-variables)）                                                                                                                                                                                                                   |     |
| `ANTHROPIC_SMALL_FAST_MODEL`                   | \[已弃用] [Haiku 类模型用于后台任务](/zh-CN/costs)的名称                                                                                                                                                                                                                                           |     |
| `ANTHROPIC_SMALL_FAST_MODEL_AWS_REGION`        | 使用 Bedrock 时覆盖 Haiku 类模型的 AWS 区域                                                                                                                                                                                                                                                    |     |
| `AWS_BEARER_TOKEN_BEDROCK`                     | Bedrock API 密钥用于身份验证（请参阅[Bedrock API 密钥](https://aws.amazon.com/blogs/machine-learning/accelerate-ai-development-with-amazon-bedrock-api-keys/)）                                                                                                                                    |     |
| `BASH_DEFAULT_TIMEOUT_MS`                      | 长时间运行的 bash 命令的默认超时                                                                                                                                                                                                                                                                 |     |
| `BASH_MAX_OUTPUT_LENGTH`                       | bash 输出中的最大字符数，超过此数字后会进行中间截断                                                                                                                                                                                                                                                        |     |
| `BASH_MAX_TIMEOUT_MS`                          | 模型可以为长时间运行的 bash 命令设置的最大超时                                                                                                                                                                                                                                                          |     |
| `CLAUDE_AUTOCOMPACT_PCT_OVERRIDE`              | 设置触发自动压缩的上下文容量百分比（1-100）。默认情况下，自动压缩在大约 95% 容量时触发。使用较低的值（如 `50`）以更早压缩。高于默认阈值的值无效。适用于主对话和 subagents。此百分比与[状态行](/zh-CN/statusline)中可用的 `context_window.used_percentage` 字段对齐                                                                                                           |     |
| `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR`     | 在每个 Bash 命令后返回到原始工作目录                                                                                                                                                                                                                                                               |     |
| `CLAUDE_CODE_ADDITIONAL_DIRECTORIES_CLAUDE_MD` | 设置为 `1` 以从使用 `--add-dir` 指定的目录加载 CLAUDE.md 文件。默认情况下，其他目录不加载内存文件                                                                                                                                                                                                                     | `1` |
| `CLAUDE_CODE_API_KEY_HELPER_TTL_MS`            | 应刷新凭证的间隔（以毫秒为单位）（使用 `apiKeyHelper` 时）                                                                                                                                                                                                                                               |     |
| `CLAUDE_CODE_CLIENT_CERT`                      | 用于 mTLS 身份验证的客户端证书文件的路径                                                                                                                                                                                                                                                             |     |
| `CLAUDE_CODE_CLIENT_KEY`                       | 用于 mTLS 身份验证的客户端私钥文件的路径                                                                                                                                                                                                                                                             |     |
| `CLAUDE_CODE_CLIENT_KEY_PASSPHRASE`            | 加密 CLAUDE\_CODE\_CLIENT\_KEY 的密码（可选）                                                                                                                                                                                                                                                |     |
| `CLAUDE_CODE_DISABLE_1M_CONTEXT`               | 设置为 `1` 以禁用[1M 上下文窗口](/zh-CN/model-config#extended-context)支持。设置后，1M 模型变体在模型选择器中不可用。对于具有合规要求的企业环境很有用                                                                                                                                                                                |     |
| `CLAUDE_CODE_DISABLE_AUTO_MEMORY`              | 设置为 `1` 以禁用[自动内存](/zh-CN/memory#auto-memory)。设置为 `0` 以在逐步推出期间强制启用自动内存。禁用时，Claude 不创建或加载自动内存文件                                                                                                                                                                                       |     |
| `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS`         | 设置为 `1` 以禁用所有后台任务功能，包括 Bash 和 subagent 工具上的 `run_in_background` 参数、自动后台处理和 Ctrl+B 快捷键                                                                                                                                                                                               |     |
| `CLAUDE_CODE_DISABLE_EXPERIMENTAL_BETAS`       | 设置为 `1` 以禁用 Anthropic API 特定的 `anthropic-beta` 标头。如果在使用带有第三方提供商的 LLM 网关时遇到"Unexpected value(s) for the `anthropic-beta` header"之类的问题，请使用此选项                                                                                                                                         |     |
| `CLAUDE_CODE_DISABLE_FEEDBACK_SURVEY`          | 设置为 `1` 以禁用"Claude 做得怎么样？"会话质量调查。使用第三方提供商或禁用遥测时也会禁用。请参阅[会话质量调查](/zh-CN/data-usage#session-quality-surveys)                                                                                                                                                                          |     |
| `CLAUDE_CODE_DISABLE_NONESSENTIAL_TRAFFIC`     | 等同于设置 `DISABLE_AUTOUPDATER`、`DISABLE_BUG_COMMAND`、`DISABLE_ERROR_REPORTING` 和 `DISABLE_TELEMETRY`                                                                                                                                                                                   |     |
| `CLAUDE_CODE_DISABLE_TERMINAL_TITLE`           | 设置为 `1` 以禁用基于对话上下文的自动终端标题更新                                                                                                                                                                                                                                                         |     |
| `CLAUDE_CODE_EFFORT_LEVEL`                     | 为支持的模型设置努力级别。值：`low`、`medium`、`high`（默认）。较低的努力更快且更便宜，较高的努力提供更深入的推理。目前仅支持 Opus 4.6。请参阅[调整努力级别](/zh-CN/model-config#adjust-effort-level)                                                                                                                                              |     |
| `CLAUDE_CODE_ENABLE_PROMPT_SUGGESTION`         | 设置为 `false` 以禁用提示建议（`/config` 中的"提示建议"切换）。这些是在 Claude 响应后出现在您的提示输入中的灰显预测。请参阅[提示建议](/zh-CN/interactive-mode#prompt-suggestions)                                                                                                                                                      |     |
| `CLAUDE_CODE_ENABLE_TASKS`                     | 设置为 `false` 以临时恢复到之前的 TODO 列表而不是任务跟踪系统。默认：`true`。请参阅[任务列表](/zh-CN/interactive-mode#task-list)                                                                                                                                                                                       |     |
| `CLAUDE_CODE_ENABLE_TELEMETRY`                 | 设置为 `1` 以启用用于指标和日志记录的 OpenTelemetry 数据收集。在配置 OTel 导出器之前需要。请参阅[监控](/zh-CN/monitoring-usage)                                                                                                                                                                                          |     |
| `CLAUDE_CODE_EXIT_AFTER_STOP_DELAY`            | 查询循环变为空闲后自动退出前等待的时间（以毫秒为单位）。对于使用 SDK 模式的自动化工作流和脚本很有用                                                                                                                                                                                                                                |     |
| `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS`         | 设置为 `1` 以启用[agent teams](/zh-CN/agent-teams)。Agent teams 是实验性的，默认禁用                                                                                                                                                                                                                 |     |
| `CLAUDE_CODE_FILE_READ_MAX_OUTPUT_TOKENS`      | 覆盖文件读取的默认令牌限制。当您需要完整读取较大文件时很有用                                                                                                                                                                                                                                                      |     |
| `CLAUDE_CODE_HIDE_ACCOUNT_INFO`                | 设置为 `1` 以从 Claude Code UI 隐藏您的电子邮件地址和组织名称。在流式传输或录制时很有用                                                                                                                                                                                                                              |     |
| `CLAUDE_CODE_IDE_SKIP_AUTO_INSTALL`            | 跳过 IDE 扩展的自动安装                                                                                                                                                                                                                                                                      |     |
| `CLAUDE_CODE_MAX_OUTPUT_TOKENS`                | 为大多数请求设置最大输出令牌数。默认：32,000。最大：64,000。增加此值会减少在[自动压缩](/zh-CN/costs#reduce-token-usage)触发之前可用的有效上下文窗口。                                                                                                                                                                                  |     |
| `CLAUDE_CODE_OTEL_HEADERS_HELPER_DEBOUNCE_MS`  | 刷新动态 OpenTelemetry 标头的间隔（以毫秒为单位）（默认：1740000 / 29 分钟）。请参阅[动态标头](/zh-CN/monitoring-usage#dynamic-headers)                                                                                                                                                                             |     |
| `CLAUDE_CODE_PLAN_MODE_REQUIRED`               | 在需要计划批准的[agent team](/zh-CN/agent-teams)队友上自动设置为 `true`。只读：由 Claude Code 在生成队友时设置。请参阅[需要计划批准](/zh-CN/agent-teams#require-plan-approval-for-teammates)                                                                                                                               |     |
| `CLAUDE_CODE_PROXY_RESOLVES_HOSTS`             | 设置为 `true` 以允许代理执行 DNS 解析而不是调用者。对于代理应处理主机名解析的环境选择加入                                                                                                                                                                                                                                 |     |
| `CLAUDE_CODE_SHELL`                            | 覆盖自动 shell 检测。当您的登录 shell 与您的首选工作 shell 不同时很有用（例如 `bash` vs `zsh`）                                                                                                                                                                                                                  |     |
| `CLAUDE_CODE_SHELL_PREFIX`                     | 包装所有 bash 命令的命令前缀（例如用于日志记录或审计）。示例：`/path/to/logger.sh` 将执行 `/path/to/logger.sh <command>`                                                                                                                                                                                           |     |
| `CLAUDE_CODE_SIMPLE`                           | 设置为 `1` 以使用最小系统提示和仅 Bash、文件读取和文件编辑工具运行。禁用 MCP 工具、附件、hooks 和 CLAUDE.md 文件                                                                                                                                                                                                            |     |
| `CLAUDE_CODE_SKIP_BEDROCK_AUTH`                | 跳过 Bedrock 的 AWS 身份验证（例如使用 LLM 网关时）                                                                                                                                                                                                                                                 |     |
| `CLAUDE_CODE_SKIP_FOUNDRY_AUTH`                | 跳过 Microsoft Foundry 的 Azure 身份验证（例如使用 LLM 网关时）                                                                                                                                                                                                                                     |     |
| `CLAUDE_CODE_SKIP_VERTEX_AUTH`                 | 跳过 Vertex 的 Google 身份验证（例如使用 LLM 网关时）                                                                                                                                                                                                                                               |     |
| `CLAUDE_CODE_SUBAGENT_MODEL`                   | 请参阅[模型配置](/zh-CN/model-config)                                                                                                                                                                                                                                                      |     |
| `CLAUDE_CODE_TASK_LIST_ID`                     | 跨会话共享任务列表。在多个 Claude Code 实例中设置相同的 ID 以协调共享任务列表。请参阅[任务列表](/zh-CN/interactive-mode#task-list)                                                                                                                                                                                        |     |
| `CLAUDE_CODE_TEAM_NAME`                        | 此队友所属的 agent team 的名称。在[agent team](/zh-CN/agent-teams)成员上自动设置                                                                                                                                                                                                                      |     |
| `CLAUDE_CODE_TMPDIR`                           | 覆盖用于内部临时文件的临时目录。Claude Code 将 `/claude/` 附加到此路径。默认：Unix/macOS 上的 `/tmp`，Windows 上的 `os.tmpdir()`                                                                                                                                                                                    |     |
| `CLAUDE_CODE_USE_BEDROCK`                      | 使用[Bedrock](/zh-CN/amazon-bedrock)                                                                                                                                                                                                                                                  |     |
| `CLAUDE_CODE_USE_FOUNDRY`                      | 使用[Microsoft Foundry](/zh-CN/microsoft-foundry)                                                                                                                                                                                                                                     |     |
| `CLAUDE_CODE_USE_VERTEX`                       | 使用[Vertex](/zh-CN/google-vertex-ai)                                                                                                                                                                                                                                                 |     |
| `CLAUDE_CONFIG_DIR`                            | 自定义 Claude Code 存储其配置和数据文件的位置                                                                                                                                                                                                                                                       |     |
| `DISABLE_AUTOUPDATER`                          | 设置为 `1` 以禁用自动更新。                                                                                                                                                                                                                                                                    |     |
| `DISABLE_BUG_COMMAND`                          | 设置为 `1` 以禁用 `/bug` 命令                                                                                                                                                                                                                                                               |     |
| `DISABLE_COST_WARNINGS`                        | 设置为 `1` 以禁用成本警告消息                                                                                                                                                                                                                                                                   |     |
| `DISABLE_ERROR_REPORTING`                      | 设置为 `1` 以选择退出 Sentry 错误报告                                                                                                                                                                                                                                                           |     |
| `DISABLE_INSTALLATION_CHECKS`                  | 设置为 `1` 以禁用安装警告。仅在手动管理安装位置时使用，因为这可能会掩盖标准安装的问题                                                                                                                                                                                                                                       |     |
| `DISABLE_NON_ESSENTIAL_MODEL_CALLS`            | 设置为 `1` 以禁用非关键路径（如风味文本）的模型调用                                                                                                                                                                                                                                                        |     |
| `DISABLE_PROMPT_CACHING`                       | 设置为 `1` 以禁用所有模型的提示缓存（优先于每个模型的设置）                                                                                                                                                                                                                                                    |     |
| `DISABLE_PROMPT_CACHING_HAIKU`                 | 设置为 `1` 以禁用 Haiku 模型的提示缓存                                                                                                                                                                                                                                                           |     |
| `DISABLE_PROMPT_CACHING_OPUS`                  | 设置为 `1` 以禁用 Opus 模型的提示缓存                                                                                                                                                                                                                                                            |     |
| `DISABLE_PROMPT_CACHING_SONNET`                | 设置为 `1` 以禁用 Sonnet 模型的提示缓存                                                                                                                                                                                                                                                          |     |
| `DISABLE_TELEMETRY`                            | 设置为 `1` 以选择退出 Statsig 遥测（注意 Statsig 事件不包括用户数据，如代码、文件路径或 bash 命令）                                                                                                                                                                                                                    |     |
| `ENABLE_TOOL_SEARCH`                           | 控制[MCP 工具搜索](/zh-CN/mcp#scale-with-mcp-tool-search)。值：`auto`（默认，在 10% 上下文处启用）、`auto:N`（自定义阈值，例如 `auto:5` 表示 5%）、`true`（始终开启）、`false`（禁用）                                                                                                                                            |     |
| `FORCE_AUTOUPDATE_PLUGINS`                     | 设置为 `true` 以强制 plugin 自动更新，即使主自动更新器通过 `DISABLE_AUTOUPDATER` 禁用                                                                                                                                                                                                                      |     |
| `HTTP_PROXY`                                   | 为网络连接指定 HTTP 代理服务器                                                                                                                                                                                                                                                                  |     |
| `HTTPS_PROXY`                                  | 为网络连接指定 HTTPS 代理服务器                                                                                                                                                                                                                                                                 |     |
| `IS_DEMO`                                      | 设置为 `true` 以启用演示模式：从 UI 隐藏电子邮件和组织，跳过入职，隐藏内部命令。对于流式传输或录制会话很有用                                                                                                                                                                                                                        |     |
| `MAX_MCP_OUTPUT_TOKENS`                        | MCP 工具响应中允许的最大令牌数。当输出超过 10,000 个令牌时，Claude Code 显示警告（默认：25000）                                                                                                                                                                                                                      |     |
| `MAX_THINKING_TOKENS`                          | 覆盖[扩展思考](https://platform.claude.com/docs/en/build-with-claude/extended-thinking)令牌预算。思考默认在最大预算（31,999 个令牌）处启用。使用此选项限制预算（例如 `MAX_THINKING_TOKENS=10000`）或完全禁用思考（`MAX_THINKING_TOKENS=0`）。对于 Opus 4.6，思考深度由[努力级别](/zh-CN/model-config#adjust-effort-level)控制，此变量被忽略，除非设置为 `0` 以禁用思考。 |     |
| `MCP_CLIENT_SECRET`                            | 需要[预配置凭证](/zh-CN/mcp#use-pre-configured-oauth-credentials)的 MCP servers 的 OAuth 客户端密钥。在添加带有 `--client-secret` 的服务器时避免交互式提示                                                                                                                                                          |     |
| `MCP_OAUTH_CALLBACK_PORT`                      | OAuth 重定向回调的固定端口，作为添加具有[预配置凭证](/zh-CN/mcp#use-pre-configured-oauth-credentials)的 MCP 服务器时 `--callback-port` 的替代方案                                                                                                                                                                   |     |
| `MCP_TIMEOUT`                                  | MCP 服务器启动的超时（以毫秒为单位）                                                                                                                                                                                                                                                                |     |
| `MCP_TOOL_TIMEOUT`                             | MCP 工具执行的超时（以毫秒为单位）                                                                                                                                                                                                                                                                 |     |
| `NO_PROXY`                                     | 将直接发出请求的域和 IP 列表，绕过代理                                                                                                                                                                                                                                                               |     |
| `SLASH_COMMAND_TOOL_CHAR_BUDGET`               | 覆盖为[Skill 工具](/zh-CN/skills#control-who-invokes-a-skill)显示的 skill 元数据的字符预算。预算以上下文窗口的 2% 动态缩放，回退为 16,000 个字符。为了向后兼容而保留的旧名称                                                                                                                                                           |     |
| `USE_BUILTIN_RIPGREP`                          | 设置为 `0` 以使用系统安装的 `rg` 而不是 Claude Code 附带的 `rg`                                                                                                                                                                                                                                      |     |
| `VERTEX_REGION_CLAUDE_3_5_HAIKU`               | 使用 Vertex AI 时覆盖 Claude 3.5 Haiku 的区域                                                                                                                                                                                                                                               |     |
| `VERTEX_REGION_CLAUDE_3_7_SONNET`              | 使用 Vertex AI 时覆盖 Claude 3.7 Sonnet 的区域                                                                                                                                                                                                                                              |     |
| `VERTEX_REGION_CLAUDE_4_0_OPUS`                | 使用 Vertex AI 时覆盖 Claude 4.0 Opus 的区域                                                                                                                                                                                                                                                |     |
| `VERTEX_REGION_CLAUDE_4_0_SONNET`              | 使用 Vertex AI 时覆盖 Claude 4.0 Sonnet 的区域                                                                                                                                                                                                                                              |     |
| `VERTEX_REGION_CLAUDE_4_1_OPUS`                | 使用 Vertex AI 时覆盖 Claude 4.1 Opus 的区域                                                                                                                                                                                                                                                |     |

## Claude 可用的工具

Claude Code 可以访问一组强大的工具，帮助它理解和修改您的代码库：

| 工具                  | 描述                                                                                                                                                 | 需要权限 |
| :------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------- | :--- |
| **AskUserQuestion** | 提出多选问题以收集需求或澄清歧义                                                                                                                                   | 否    |
| **Bash**            | 在您的环境中执行 shell 命令（请参阅下面的[Bash 工具行为](#bash-tool-behavior)）                                                                                          | 是    |
| **TaskOutput**      | 从后台任务（bash shell 或 subagent）检索输出                                                                                                                   | 否    |
| **Edit**            | 对特定文件进行有针对性的编辑                                                                                                                                     | 是    |
| **ExitPlanMode**    | 提示用户退出 Plan Mode 并开始编码                                                                                                                             | 是    |
| **Glob**            | 基于模式匹配查找文件                                                                                                                                         | 否    |
| **Grep**            | 在文件内容中搜索模式                                                                                                                                         | 否    |
| **KillShell**       | 按其 ID 杀死正在运行的后台 bash shell                                                                                                                         | 否    |
| **MCPSearch**       | 当启用[工具搜索](/zh-CN/mcp#scale-with-mcp-tool-search)时搜索和加载 MCP 工具                                                                                      | 否    |
| **NotebookEdit**    | 修改 Jupyter notebook 单元格                                                                                                                            | 是    |
| **Read**            | 读取文件的内容                                                                                                                                            | 否    |
| **Skill**           | 在主对话中执行[skill](/zh-CN/skills#control-who-invokes-a-skill)                                                                                          | 是    |
| **Task**            | 运行 sub-agent 以处理复杂的多步骤任务                                                                                                                           | 否    |
| **TaskCreate**      | 在任务列表中创建新任务                                                                                                                                        | 否    |
| **TaskGet**         | 检索特定任务的完整详细信息                                                                                                                                      | 否    |
| **TaskList**        | 列出所有任务及其当前状态                                                                                                                                       | 否    |
| **TaskUpdate**      | 更新任务状态、依赖项、详细信息或删除任务                                                                                                                               | 否    |
| **WebFetch**        | 从指定的 URL 获取内容                                                                                                                                      | 是    |
| **WebSearch**       | 执行带有域过滤的网络搜索                                                                                                                                       | 是    |
| **Write**           | 创建或覆盖文件                                                                                                                                            | 是    |
| **LSP**             | 通过语言服务器的代码智能。在文件编辑后自动报告类型错误和警告。还支持导航操作：跳转到定义、查找引用、获取类型信息、列出符号、查找实现、跟踪调用层次结构。需要[代码智能 plugin](/zh-CN/discover-plugins#code-intelligence)及其语言服务器二进制文件 | 否    |

权限规则可以使用 `/allowed-tools` 或在[权限设置](/zh-CN/settings#available-settings)中配置。另请参阅[工具特定权限规则](/zh-CN/permissions#tool-specific-permission-rules)。

### Bash 工具行为

Bash 工具执行 shell 命令，具有以下持久性行为：

* **工作目录持久化**：当 Claude 更改工作目录（例如 `cd /path/to/dir`）时，后续 Bash 命令将在该目录中执行。您可以使用 `CLAUDE_BASH_MAINTAIN_PROJECT_WORKING_DIR=1` 在每个命令后重置为项目目录。
* **环境变量不持久化**：在一个 Bash 命令中设置的环境变量（例如 `export MY_VAR=value`）**不**在后续 Bash 命令中可用。每个 Bash 命令在新的 shell 环境中运行。

要使环境变量在 Bash 命令中可用，您有**三个选项**：

**选项 1：在启动 Claude Code 之前激活环境**（最简单的方法）

在启动 Claude Code 之前在您的终端中激活您的虚拟环境：

```bash  theme={null}
conda activate myenv
# 或：source /path/to/venv/bin/activate
claude
```

这适用于 shell 环境，但在 Claude 的 Bash 命令中设置的环境变量不会在命令之间持久化。

**选项 2：在启动 Claude Code 之前设置 CLAUDE\_ENV\_FILE**（持久环境设置）

导出包含您的环境设置的 shell 脚本的路径：

```bash  theme={null}
export CLAUDE_ENV_FILE=/path/to/env-setup.sh
claude
```

其中 `/path/to/env-setup.sh` 包含：

```bash  theme={null}
conda activate myenv
# 或：source /path/to/venv/bin/activate
# 或：export MY_VAR=value
```

Claude Code 将在每个 Bash 命令之前获取此文件，使环境在所有命令中持久化。

**选项 3：使用 SessionStart hook**（项目特定配置）

在 `.claude/settings.json` 中配置：

```json  theme={null}
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

hook 写入 `$CLAUDE_ENV_FILE`，然后在每个 Bash 命令之前获取。这对于团队共享的项目配置很理想。

有关选项 3 的更多详细信息，请参阅[SessionStart hooks](/zh-CN/hooks#persist-environment-variables)。

### 使用 hooks 扩展工具

您可以使用[Claude Code hooks](/zh-CN/hooks-guide)在任何工具执行之前或之后运行自定义命令。

例如，您可以在 Claude 修改 Python 文件后自动运行 Python 格式化程序，或通过阻止对某些路径的 Write 操作来防止修改生产配置文件。

## 另请参阅

* [权限](/zh-CN/permissions)：权限系统、规则语法、工具特定模式和 managed 策略
* [身份验证](/zh-CN/authentication)：设置用户对 Claude Code 的访问
* [故障排除](/zh-CN/troubleshooting)：常见配置问题的解决方案
