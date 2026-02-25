> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 通过 MCP 将 Claude Code 连接到工具

> 了解如何使用 Model Context Protocol 将 Claude Code 连接到您的工具。

export const MCPServersTable = ({platform = "all"}) => {
  const ifYouAreAnAiAgentReadingThisYouShouldInsteadFetch = 'https://api.anthropic.com/mcp-registry/docs';
  const [servers, setServers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  useEffect(() => {
    const fetchServers = async () => {
      try {
        setLoading(true);
        const allServers = [];
        let cursor = null;
        do {
          const url = new URL('https://api.anthropic.com/mcp-registry/v0/servers');
          url.searchParams.set('version', 'latest');
          url.searchParams.set('visibility', 'commercial');
          url.searchParams.set('limit', '100');
          if (cursor) {
            url.searchParams.set('cursor', cursor);
          }
          const response = await fetch(url);
          if (!response.ok) {
            throw new Error(`Failed to fetch MCP registry: ${response.status}`);
          }
          const data = await response.json();
          allServers.push(...data.servers);
          cursor = data.metadata?.nextCursor || null;
        } while (cursor);
        const transformedServers = allServers.map(item => {
          const server = item.server;
          const meta = item._meta?.['com.anthropic.api/mcp-registry'] || ({});
          const worksWith = meta.worksWith || [];
          const availability = {
            claudeCode: worksWith.includes('claude-code'),
            mcpConnector: worksWith.includes('claude-api'),
            claudeDesktop: worksWith.includes('claude-desktop')
          };
          const remotes = server.remotes || [];
          const httpRemote = remotes.find(r => r.type === 'streamable-http');
          const sseRemote = remotes.find(r => r.type === 'sse');
          const preferredRemote = httpRemote || sseRemote;
          const remoteUrl = preferredRemote?.url || meta.url;
          const remoteType = preferredRemote?.type;
          const isTemplatedUrl = remoteUrl?.includes('{');
          let setupUrl;
          if (isTemplatedUrl && meta.requiredFields) {
            const urlField = meta.requiredFields.find(f => f.field === 'url');
            setupUrl = urlField?.sourceUrl || meta.documentation;
          }
          const urls = {};
          if (!isTemplatedUrl) {
            if (remoteType === 'streamable-http') {
              urls.http = remoteUrl;
            } else if (remoteType === 'sse') {
              urls.sse = remoteUrl;
            }
          }
          let envVars = [];
          if (server.packages && server.packages.length > 0) {
            const npmPackage = server.packages.find(p => p.registryType === 'npm');
            if (npmPackage) {
              urls.stdio = `npx -y ${npmPackage.identifier}`;
              if (npmPackage.environmentVariables) {
                envVars = npmPackage.environmentVariables;
              }
            }
          }
          return {
            name: meta.displayName || server.title || server.name,
            description: meta.oneLiner || server.description,
            documentation: meta.documentation,
            urls: urls,
            envVars: envVars,
            availability: availability,
            customCommands: meta.claudeCodeCopyText ? {
              claudeCode: meta.claudeCodeCopyText
            } : undefined,
            setupUrl: setupUrl
          };
        });
        setServers(transformedServers);
        setError(null);
      } catch (err) {
        setError(err.message);
        console.error('Error fetching MCP registry:', err);
      } finally {
        setLoading(false);
      }
    };
    fetchServers();
  }, []);
  const generateClaudeCodeCommand = server => {
    if (server.customCommands && server.customCommands.claudeCode) {
      return server.customCommands.claudeCode;
    }
    const serverSlug = server.name.toLowerCase().replace(/[^a-z0-9]/g, '-');
    if (server.urls.http) {
      return `claude mcp add ${serverSlug} --transport http ${server.urls.http}`;
    }
    if (server.urls.sse) {
      return `claude mcp add ${serverSlug} --transport sse ${server.urls.sse}`;
    }
    if (server.urls.stdio) {
      const envFlags = server.envVars && server.envVars.length > 0 ? server.envVars.map(v => `--env ${v.name}=YOUR_${v.name}`).join(' ') : '';
      const baseCommand = `claude mcp add ${serverSlug} --transport stdio`;
      return envFlags ? `${baseCommand} ${envFlags} -- ${server.urls.stdio}` : `${baseCommand} -- ${server.urls.stdio}`;
    }
    return null;
  };
  if (loading) {
    return <div>Loading MCP servers...</div>;
  }
  if (error) {
    return <div>Error loading MCP servers: {error}</div>;
  }
  const filteredServers = servers.filter(server => {
    if (platform === "claudeCode") {
      return server.availability.claudeCode;
    } else if (platform === "mcpConnector") {
      return server.availability.mcpConnector;
    } else if (platform === "claudeDesktop") {
      return server.availability.claudeDesktop;
    } else if (platform === "all") {
      return true;
    } else {
      throw new Error(`Unknown platform: ${platform}`);
    }
  });
  return <>
      <style jsx>{`
        .cards-container {
          display: grid;
          gap: 1rem;
          margin-bottom: 2rem;
        }
        .server-card {
          border: 1px solid var(--border-color, #e5e7eb);
          border-radius: 6px;
          padding: 1rem;
        }
        .command-row {
          display: flex;
          align-items: center;
          gap: 0.25rem;
        }
        .command-row code {
          font-size: 0.75rem;
          overflow-x: auto;
        }
      `}</style>

      <div className="cards-container">
        {filteredServers.map(server => {
    const claudeCodeCommand = generateClaudeCodeCommand(server);
    const mcpUrl = server.urls.http || server.urls.sse;
    const commandToShow = platform === "claudeCode" ? claudeCodeCommand : mcpUrl;
    return <div key={server.name} className="server-card">
              <div>
                {server.documentation ? <a href={server.documentation}>
                    <strong>{server.name}</strong>
                  </a> : <strong>{server.name}</strong>}
              </div>

              <p style={{
      margin: '0.5rem 0',
      fontSize: '0.9rem'
    }}>
                {server.description}
              </p>

              {server.setupUrl && <p style={{
      margin: '0.25rem 0',
      fontSize: '0.8rem',
      fontStyle: 'italic',
      opacity: 0.7
    }}>
                  Requires user-specific URL.{' '}
                  <a href={server.setupUrl} style={{
      textDecoration: 'underline'
    }}>
                    Get your URL here
                  </a>.
                </p>}

              {commandToShow && !server.setupUrl && <>
                <p style={{
      display: 'block',
      fontSize: '0.75rem',
      fontWeight: 500,
      minWidth: 'fit-content',
      marginTop: '0.5rem',
      marginBottom: 0
    }}>
                  {platform === "claudeCode" ? "Command" : "URL"}
                </p>
                <div className="command-row">
                  <code>
                    {commandToShow}
                  </code>
                </div>
              </>}
            </div>;
  })}
      </div>
    </>;
};

Claude Code 可以通过 [Model Context Protocol (MCP)](https://modelcontextprotocol.io/introduction)（一个用于 AI 工具集成的开源标准）连接到数百个外部工具和数据源。MCP 服务器为 Claude Code 提供对您的工具、数据库和 API 的访问权限。

## 使用 MCP 可以做什么

连接 MCP 服务器后，您可以要求 Claude Code：

* **从问题跟踪器实现功能**："添加 JIRA 问题 ENG-4521 中描述的功能，并在 GitHub 上创建 PR。"
* **分析监控数据**："检查 Sentry 和 Statsig 以检查 ENG-4521 中描述的功能的使用情况。"
* **查询数据库**："根据我们的 PostgreSQL 数据库，找到使用功能 ENG-4521 的 10 个随机用户的电子邮件。"
* **集成设计**："根据在 Slack 中发布的新 Figma 设计更新我们的标准电子邮件模板"
* **自动化工作流**："创建 Gmail 草稿，邀请这 10 个用户参加关于新功能的反馈会议。"

## 流行的 MCP 服务器

以下是一些您可以连接到 Claude Code 的常用 MCP 服务器：

<Warning>
  使用第三方 MCP 服务器需自担风险 - Anthropic 尚未验证
  所有这些服务器的正确性或安全性。
  确保您信任正在安装的 MCP 服务器。
  使用可能获取不受信任内容的 MCP 服务器时要特别小心，
  因为这些可能会使您面临提示注入风险。
</Warning>

<MCPServersTable platform="claudeCode" />

<Note>
  **需要特定的集成？** [在 GitHub 上查找数百个更多 MCP 服务器](https://github.com/modelcontextprotocol/servers)，或使用 [MCP SDK](https://modelcontextprotocol.io/quickstart/server) 构建您自己的服务器。
</Note>

## 安装 MCP 服务器

MCP 服务器可以根据您的需求以三种不同的方式进行配置：

### 选项 1：添加远程 HTTP 服务器

HTTP 服务器是连接到远程 MCP 服务器的推荐选项。这是云服务最广泛支持的传输方式。

```bash  theme={null}
# 基本语法
claude mcp add --transport http <name> <url>

# 真实示例：连接到 Notion
claude mcp add --transport http notion https://mcp.notion.com/mcp

# 带有 Bearer 令牌的示例
claude mcp add --transport http secure-api https://api.example.com/mcp \
  --header "Authorization: Bearer your-token"
```

### 选项 2：添加远程 SSE 服务器

<Warning>
  SSE (Server-Sent Events) 传输已弃用。请改用 HTTP 服务器（如果可用）。
</Warning>

```bash  theme={null}
# 基本语法
claude mcp add --transport sse <name> <url>

# 真实示例：连接到 Asana
claude mcp add --transport sse asana https://mcp.asana.com/sse

# 带有身份验证标头的示例
claude mcp add --transport sse private-api https://api.company.com/sse \
  --header "X-API-Key: your-key-here"
```

### 选项 3：添加本地 stdio 服务器

Stdio 服务器作为您机器上的本地进程运行。它们非常适合需要直接系统访问或自定义脚本的工具。

```bash  theme={null}
# 基本语法
claude mcp add [options] <name> -- <command> [args...]

# 真实示例：添加 Airtable 服务器
claude mcp add --transport stdio --env AIRTABLE_API_KEY=YOUR_KEY airtable \
  -- npx -y airtable-mcp-server
```

<Note>
  **重要：选项顺序**

  所有选项（`--transport`、`--env`、`--scope`、`--header`）必须位于服务器名称**之前**。然后 `--`（双破折号）将服务器名称与传递给 MCP 服务器的命令和参数分开。

  例如：

  * `claude mcp add --transport stdio myserver -- npx server` → 运行 `npx server`
  * `claude mcp add --transport stdio --env KEY=value myserver -- python server.py --port 8080` → 运行 `python server.py --port 8080`，环境中有 `KEY=value`

  这可以防止 Claude 的标志与服务器标志之间的冲突。
</Note>

### 管理您的服务器

配置后，您可以使用这些命令管理您的 MCP 服务器：

```bash  theme={null}
# 列出所有配置的服务器
claude mcp list

# 获取特定服务器的详细信息
claude mcp get github

# 删除服务器
claude mcp remove github

# （在 Claude Code 中）检查服务器状态
/mcp
```

### 动态工具更新

Claude Code 支持 MCP `list_changed` 通知，允许 MCP 服务器动态更新其可用工具、提示和资源，而无需您断开连接并重新连接。当 MCP 服务器发送 `list_changed` 通知时，Claude Code 会自动刷新来自该服务器的可用功能。

<Tip>
  提示：

  * 使用 `--scope` 标志指定配置的存储位置：
    * `local`（默认）：仅在当前项目中对您可用（在较旧版本中称为 `project`）
    * `project`：通过 `.mcp.json` 文件与项目中的每个人共享
    * `user`：在所有项目中对您可用（在较旧版本中称为 `global`）
  * 使用 `--env` 标志设置环境变量（例如，`--env KEY=value`）
  * 使用 MCP\_TIMEOUT 环境变量配置 MCP 服务器启动超时（例如，`MCP_TIMEOUT=10000 claude` 设置 10 秒超时）
  * 当 MCP 工具输出超过 10,000 个令牌时，Claude Code 将显示警告。要增加此限制，请设置 `MAX_MCP_OUTPUT_TOKENS` 环境变量（例如，`MAX_MCP_OUTPUT_TOKENS=50000`）
  * 使用 `/mcp` 对需要 OAuth 2.0 身份验证的远程服务器进行身份验证
</Tip>

<Warning>
  **Windows 用户**：在本机 Windows（不是 WSL）上，使用 `npx` 的本地 MCP 服务器需要 `cmd /c` 包装器以确保正确执行。

  ```bash  theme={null}
  # 这创建了 Windows 可以执行的 command="cmd"
  claude mcp add --transport stdio my-server -- cmd /c npx -y @some/package
  ```

  没有 `cmd /c` 包装器，您会遇到"连接已关闭"错误，因为 Windows 无法直接执行 `npx`。（有关 `--` 参数的说明，请参阅上面的注释。）
</Warning>

### 插件提供的 MCP 服务器

[插件](/zh-CN/plugins)可以捆绑 MCP 服务器，在启用插件时自动提供工具和集成。插件 MCP 服务器的工作方式与用户配置的服务器相同。

**插件 MCP 服务器的工作原理**：

* 插件在插件根目录的 `.mcp.json` 中或在 `plugin.json` 中内联定义 MCP 服务器
* 启用插件时，其 MCP 服务器会自动启动
* 插件 MCP 工具与手动配置的 MCP 工具一起出现
* 插件服务器通过插件安装进行管理（不是 `/mcp` 命令）

**示例插件 MCP 配置**：

在插件根目录的 `.mcp.json` 中：

```json  theme={null}
{
  "database-tools": {
    "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
    "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"],
    "env": {
      "DB_URL": "${DB_URL}"
    }
  }
}
```

或在 `plugin.json` 中内联：

```json  theme={null}
{
  "name": "my-plugin",
  "mcpServers": {
    "plugin-api": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/api-server",
      "args": ["--port", "8080"]
    }
  }
}
```

**插件 MCP 功能**：

* **自动生命周期**：服务器在插件启用时启动，但您必须重新启动 Claude Code 以应用 MCP 服务器更改（启用或禁用）
* **环境变量**：使用 `${CLAUDE_PLUGIN_ROOT}` 表示插件相对路径
* **用户环境访问**：访问与手动配置的服务器相同的环境变量
* **多种传输类型**：支持 stdio、SSE 和 HTTP 传输（传输支持可能因服务器而异）

**查看插件 MCP 服务器**：

```bash  theme={null}
# 在 Claude Code 中，查看所有 MCP 服务器，包括插件服务器
/mcp
```

插件服务器在列表中出现，并带有指示它们来自插件的指示符。

**插件 MCP 服务器的优势**：

* **捆绑分发**：工具和服务器打包在一起
* **自动设置**：无需手动 MCP 配置
* **团队一致性**：安装插件时每个人都获得相同的工具

有关使用插件捆绑 MCP 服务器的详细信息，请参阅[插件组件参考](/zh-CN/plugins-reference#mcp-servers)。

## MCP 安装范围

MCP 服务器可以在三个不同的范围级别进行配置，每个级别都用于管理服务器可访问性和共享的不同目的。了解这些范围可以帮助您确定为特定需求配置服务器的最佳方式。

### 本地范围

本地范围的服务器代表默认配置级别，存储在您项目路径下的 `~/.claude.json` 中。这些服务器对您保持私密，仅在当前项目目录中工作时可访问。此范围非常适合个人开发服务器、实验配置或包含不应共享的敏感凭据的服务器。

<Note>
  MCP 服务器的"本地范围"术语与一般本地设置不同。MCP 本地范围的服务器存储在 `~/.claude.json`（您的主目录）中，而一般本地设置使用 `.claude/settings.local.json`（在项目目录中）。有关设置文件位置的详细信息，请参阅[设置](/zh-CN/settings#settings-files)。
</Note>

```bash  theme={null}
# 添加本地范围的服务器（默认）
claude mcp add --transport http stripe https://mcp.stripe.com

# 显式指定本地范围
claude mcp add --transport http stripe --scope local https://mcp.stripe.com
```

### 项目范围

项目范围的服务器通过将配置存储在项目根目录的 `.mcp.json` 文件中来启用团队协作。此文件设计为检入版本控制，确保所有团队成员都可以访问相同的 MCP 工具和服务。添加项目范围的服务器时，Claude Code 会自动创建或更新此文件，使用适当的配置结构。

```bash  theme={null}
# 添加项目范围的服务器
claude mcp add --transport http paypal --scope project https://mcp.paypal.com/mcp
```

生成的 `.mcp.json` 文件遵循标准化格式：

```json  theme={null}
{
  "mcpServers": {
    "shared-server": {
      "command": "/path/to/server",
      "args": [],
      "env": {}
    }
  }
}
```

出于安全原因，Claude Code 在使用来自 `.mcp.json` 文件的项目范围的服务器之前会提示批准。如果您需要重置这些批准选择，请使用 `claude mcp reset-project-choices` 命令。

### 用户范围

用户范围的服务器存储在 `~/.claude.json` 中，提供跨项目可访问性，使其在您机器上的所有项目中可用，同时对您的用户帐户保持私密。此范围适用于个人实用程序服务器、开发工具或您在不同项目中经常使用的服务。

```bash  theme={null}
# 添加用户服务器
claude mcp add --transport http hubspot --scope user https://mcp.hubspot.com/anthropic
```

### 选择正确的范围

根据以下条件选择您的范围：

* **本地范围**：个人服务器、实验配置或特定于一个项目的敏感凭据
* **项目范围**：团队共享的服务器、项目特定的工具或协作所需的服务
* **用户范围**：跨多个项目需要的个人实用程序、开发工具或经常使用的服务

<Note>
  **MCP 服务器存储在哪里？**

  * **用户和本地范围**：`~/.claude.json`（在 `mcpServers` 字段或项目路径下）
  * **项目范围**：项目根目录中的 `.mcp.json`（检入源代码控制）
  * **托管**：系统目录中的 `managed-mcp.json`（请参阅[托管 MCP 配置](#managed-mcp-configuration)）
</Note>

### 范围层次结构和优先级

MCP 服务器配置遵循清晰的优先级层次结构。当具有相同名称的服务器存在于多个范围时，系统通过优先考虑本地范围的服务器来解决冲突，然后是项目范围的服务器，最后是用户范围的服务器。此设计确保个人配置可以在需要时覆盖共享配置。

### `.mcp.json` 中的环境变量扩展

Claude Code 支持 `.mcp.json` 文件中的环境变量扩展，允许团队共享配置，同时为特定于机器的路径和 API 密钥等敏感值保持灵活性。

**支持的语法：**

* `${VAR}` - 扩展为环境变量 `VAR` 的值
* `${VAR:-default}` - 如果设置了 `VAR`，则扩展为 `VAR`，否则使用 `default`

**扩展位置：**
环境变量可以在以下位置扩展：

* `command` - 服务器可执行文件路径
* `args` - 命令行参数
* `env` - 传递给服务器的环境变量
* `url` - 对于 HTTP 服务器类型
* `headers` - 对于 HTTP 服务器身份验证

**带有变量扩展的示例：**

```json  theme={null}
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

## 实际示例

{/* ### 示例：使用 Playwright 自动化浏览器测试

  ```bash
  # 1. 添加 Playwright MCP 服务器
  claude mcp add --transport stdio playwright -- npx -y @playwright/mcp@latest

  # 2. 编写并运行浏览器测试
  > "Test if the login flow works with test@example.com"
  > "Take a screenshot of the checkout page on mobile"
  > "Verify that the search feature returns results"
  ``` */}

### 示例：使用 Sentry 监控错误

```bash  theme={null}
# 1. 添加 Sentry MCP 服务器
claude mcp add --transport http sentry https://mcp.sentry.dev/mcp

# 2. 使用 /mcp 对您的 Sentry 帐户进行身份验证
> /mcp

# 3. 调试生产问题
> "What are the most common errors in the last 24 hours?"
> "Show me the stack trace for error ID abc123"
> "Which deployment introduced these new errors?"
```

### 示例：连接到 GitHub 进行代码审查

```bash  theme={null}
# 1. 添加 GitHub MCP 服务器
claude mcp add --transport http github https://api.githubcopilot.com/mcp/

# 2. 在 Claude Code 中，如果需要进行身份验证
> /mcp
# 为 GitHub 选择"身份验证"

# 3. 现在您可以要求 Claude 使用 GitHub
> "Review PR #456 and suggest improvements"
> "Create a new issue for the bug we just found"
> "Show me all open PRs assigned to me"
```

### 示例：查询您的 PostgreSQL 数据库

```bash  theme={null}
# 1. 使用您的连接字符串添加数据库服务器
claude mcp add --transport stdio db -- npx -y @bytebase/dbhub \
  --dsn "postgresql://readonly:pass@prod.db.com:5432/analytics"

# 2. 自然地查询您的数据库
> "What's our total revenue this month?"
> "Show me the schema for the orders table"
> "Find customers who haven't made a purchase in 90 days"
```

## 使用远程 MCP 服务器进行身份验证

许多基于云的 MCP 服务器需要身份验证。Claude Code 支持 OAuth 2.0 以实现安全连接。

<Steps>
  <Step title="添加需要身份验证的服务器">
    例如：

    ```bash  theme={null}
    claude mcp add --transport http sentry https://mcp.sentry.dev/mcp
    ```
  </Step>

  <Step title="在 Claude Code 中使用 /mcp 命令">
    在 Claude Code 中，使用命令：

    ```
    > /mcp
    ```

    然后按照浏览器中的步骤登录。
  </Step>
</Steps>

<Tip>
  提示：

  * 身份验证令牌存储安全且自动刷新
  * 使用 `/mcp` 菜单中的"清除身份验证"撤销访问权限
  * 如果您的浏览器没有自动打开，请复制提供的 URL
  * OAuth 身份验证适用于 HTTP 服务器
</Tip>

### 使用预配置的 OAuth 凭据

某些 MCP 服务器不支持自动 OAuth 设置。如果您看到"不兼容的身份验证服务器：不支持动态客户端注册"之类的错误，服务器需要预配置的凭据。首先通过服务器的开发者门户注册 OAuth 应用，然后在添加服务器时提供凭据。

<Steps>
  <Step title="使用服务器注册 OAuth 应用">
    通过服务器的开发者门户创建应用，并记下您的客户端 ID 和客户端密钥。

    许多服务器还需要重定向 URI。如果是这样，请选择一个端口并以 `http://localhost:PORT/callback` 的格式注册重定向 URI。在下一步中使用 `--callback-port` 时使用相同的端口。
  </Step>

  <Step title="使用您的凭据添加服务器">
    选择以下方法之一。用于 `--callback-port` 的端口可以是任何可用的端口。它只需要与您在上一步中注册的重定向 URI 匹配。

    <Tabs>
      <Tab title="claude mcp add">
        使用 `--client-id` 传递您的应用的客户端 ID。`--client-secret` 标志提示输入带有掩盖输入的密钥：

        ```bash  theme={null}
        claude mcp add --transport http \
          --client-id your-client-id --client-secret --callback-port 8080 \
          my-server https://mcp.example.com/mcp
        ```
      </Tab>

      <Tab title="claude mcp add-json">
        在 JSON 配置中包含 `oauth` 对象，并将 `--client-secret` 作为单独的标志传递：

        ```bash  theme={null}
        claude mcp add-json my-server \
          '{"type":"http","url":"https://mcp.example.com/mcp","oauth":{"clientId":"your-client-id","callbackPort":8080}}' \
          --client-secret
        ```
      </Tab>

      <Tab title="CI / env var">
        通过环境变量设置密钥以跳过交互式提示：

        ```bash  theme={null}
        MCP_CLIENT_SECRET=your-secret claude mcp add --transport http \
          --client-id your-client-id --client-secret --callback-port 8080 \
          my-server https://mcp.example.com/mcp
        ```
      </Tab>
    </Tabs>
  </Step>

  <Step title="在 Claude Code 中进行身份验证">
    在 Claude Code 中运行 `/mcp` 并按照浏览器登录流程进行操作。
  </Step>
</Steps>

<Tip>
  提示：

  * 客户端密钥安全地存储在您的系统钥匙链（macOS）或凭据文件中，而不是在您的配置中
  * 如果服务器使用没有密钥的公共 OAuth 客户端，仅使用 `--client-id` 而不使用 `--client-secret`
  * 这些标志仅适用于 HTTP 和 SSE 传输。它们对 stdio 服务器没有影响
  * 使用 `claude mcp get <name>` 验证为服务器配置了 OAuth 凭据
</Tip>

## 从 JSON 配置添加 MCP 服务器

如果您有 MCP 服务器的 JSON 配置，可以直接添加它：

<Steps>
  <Step title="从 JSON 添加 MCP 服务器">
    ```bash  theme={null}
    # 基本语法
    claude mcp add-json <name> '<json>'

    # 示例：使用 JSON 配置添加 HTTP 服务器
    claude mcp add-json weather-api '{"type":"http","url":"https://api.weather.com/mcp","headers":{"Authorization":"Bearer token"}}'

    # 示例：使用 JSON 配置添加 stdio 服务器
    claude mcp add-json local-weather '{"type":"stdio","command":"/path/to/weather-cli","args":["--api-key","abc123"],"env":{"CACHE_DIR":"/tmp"}}'

    # 示例：添加带有预配置 OAuth 凭据的 HTTP 服务器
    claude mcp add-json my-server '{"type":"http","url":"https://mcp.example.com/mcp","oauth":{"clientId":"your-client-id","callbackPort":8080}}' --client-secret
    ```
  </Step>

  <Step title="验证服务器已添加">
    ```bash  theme={null}
    claude mcp get weather-api
    ```
  </Step>
</Steps>

<Tip>
  提示：

  * 确保 JSON 在您的 shell 中正确转义
  * JSON 必须符合 MCP 服务器配置架构
  * 您可以使用 `--scope user` 将服务器添加到您的用户配置而不是项目特定的配置
</Tip>

## 从 Claude Desktop 导入 MCP 服务器

如果您已在 Claude Desktop 中配置了 MCP 服务器，可以导入它们：

<Steps>
  <Step title="从 Claude Desktop 导入服务器">
    ```bash  theme={null}
    # 基本语法 
    claude mcp add-from-claude-desktop 
    ```
  </Step>

  <Step title="选择要导入的服务器">
    运行命令后，您将看到一个交互式对话框，允许您选择要导入的服务器。
  </Step>

  <Step title="验证服务器已导入">
    ```bash  theme={null}
    claude mcp list 
    ```
  </Step>
</Steps>

<Tip>
  提示：

  * 此功能仅在 macOS 和 Windows Subsystem for Linux (WSL) 上有效
  * 它从这些平台上的标准位置读取 Claude Desktop 配置文件
  * 使用 `--scope user` 标志将服务器添加到您的用户配置
  * 导入的服务器将具有与 Claude Desktop 中相同的名称
  * 如果已存在具有相同名称的服务器，它们将获得数字后缀（例如，`server_1`）
</Tip>

## 使用来自 Claude.ai 的 MCP 服务器

如果您已使用 [Claude.ai](https://claude.ai) 帐户登录 Claude Code，您在 Claude.ai 中添加的 MCP 服务器会自动在 Claude Code 中可用：

<Steps>
  <Step title="在 Claude.ai 中配置 MCP 服务器">
    在 [claude.ai/settings/connectors](https://claude.ai/settings/connectors) 添加服务器。在 Team 和 Enterprise 计划中，只有管理员可以添加服务器。
  </Step>

  <Step title="对 MCP 服务器进行身份验证">
    在 Claude.ai 中完成任何必需的身份验证步骤。
  </Step>

  <Step title="在 Claude Code 中查看和管理服务器">
    在 Claude Code 中，使用命令：

    ```
    # 在 Claude Code 中，查看所有 MCP 服务器，包括 Claude.ai 服务器
    > /mcp
    ```

    Claude.ai 服务器在列表中出现，并带有指示它们来自 Claude.ai 的指示符。
  </Step>
</Steps>

## 将 Claude Code 用作 MCP 服务器

您可以将 Claude Code 本身用作 MCP 服务器，其他应用程序可以连接到它：

```bash  theme={null}
# 启动 Claude 作为 stdio MCP 服务器
claude mcp serve
```

您可以通过将此配置添加到 claude\_desktop\_config.json 在 Claude Desktop 中使用它：

```json  theme={null}
{
  "mcpServers": {
    "claude-code": {
      "type": "stdio",
      "command": "claude",
      "args": ["mcp", "serve"],
      "env": {}
    }
  }
}
```

<Warning>
  **配置可执行文件路径**：`command` 字段必须引用 Claude Code 可执行文件。如果 `claude` 命令不在您的系统 PATH 中，您需要指定可执行文件的完整路径。

  要查找完整路径：

  ```bash  theme={null}
  which claude
  ```

  然后在您的配置中使用完整路径：

  ```json  theme={null}
  {
    "mcpServers": {
      "claude-code": {
        "type": "stdio",
        "command": "/full/path/to/claude",
        "args": ["mcp", "serve"],
        "env": {}
      }
    }
  }
  ```

  没有正确的可执行文件路径，您会遇到 `spawn claude ENOENT` 之类的错误。
</Warning>

<Tip>
  提示：

  * 服务器提供对 Claude 的工具（如 View、Edit、LS 等）的访问权限。
  * 在 Claude Desktop 中，尝试要求 Claude 读取目录中的文件、进行编辑等。
  * 请注意，此 MCP 服务器仅向您的 MCP 客户端公开 Claude Code 的工具，因此您自己的客户端负责为各个工具调用实现用户确认。
</Tip>

## MCP 输出限制和警告

当 MCP 工具产生大量输出时，Claude Code 有助于管理令牌使用，以防止压倒您的对话上下文：

* **输出警告阈值**：当任何 MCP 工具输出超过 10,000 个令牌时，Claude Code 显示警告
* **可配置限制**：您可以使用 `MAX_MCP_OUTPUT_TOKENS` 环境变量调整最大允许的 MCP 输出令牌
* **默认限制**：默认最大值为 25,000 个令牌

要增加产生大量输出的工具的限制：

```bash  theme={null}
# 为 MCP 工具输出设置更高的限制
export MAX_MCP_OUTPUT_TOKENS=50000
claude
```

这在使用以下 MCP 服务器时特别有用：

* 查询大型数据集或数据库
* 生成详细的报告或文档
* 处理大量日志文件或调试信息

<Warning>
  如果您经常遇到特定 MCP 服务器的输出警告，请考虑增加限制或配置服务器以分页或过滤其响应。
</Warning>

## 使用 MCP 资源

MCP 服务器可以公开资源，您可以使用 @ 提及来引用这些资源，类似于引用文件的方式。

### 引用 MCP 资源

<Steps>
  <Step title="列出可用资源">
    在您的提示中键入 `@` 以查看来自所有连接的 MCP 服务器的可用资源。资源与文件一起出现在自动完成菜单中。
  </Step>

  <Step title="引用特定资源">
    使用格式 `@server:protocol://resource/path` 引用资源：

    ```
    > Can you analyze @github:issue://123 and suggest a fix?
    ```

    ```
    > Please review the API documentation at @docs:file://api/authentication
    ```
  </Step>

  <Step title="多个资源引用">
    您可以在单个提示中引用多个资源：

    ```
    > Compare @postgres:schema://users with @docs:file://database/user-model
    ```
  </Step>
</Steps>

<Tip>
  提示：

  * 引用时会自动获取资源并将其作为附件包含
  * 资源路径在 @ 提及自动完成中可进行模糊搜索
  * 当服务器支持时，Claude Code 会自动提供列出和读取 MCP 资源的工具
  * 资源可以包含 MCP 服务器提供的任何类型的内容（文本、JSON、结构化数据等）
</Tip>

## 使用 MCP 工具搜索进行扩展

当您配置了许多 MCP 服务器时，工具定义可能会消耗您的上下文窗口的很大一部分。MCP 工具搜索通过动态按需加载工具而不是预加载所有工具来解决此问题。

### 工作原理

当您的 MCP 工具描述会消耗超过 10% 的上下文窗口时，Claude Code 会自动启用工具搜索。您可以[调整此阈值](#configure-tool-search)或完全禁用工具搜索。触发时：

1. MCP 工具被延迟而不是预先加载到上下文中
2. Claude 使用搜索工具在需要时发现相关的 MCP 工具
3. 只有 Claude 实际需要的工具才会加载到上下文中
4. MCP 工具从您的角度继续完全按照之前的方式工作

### 对于 MCP 服务器作者

如果您正在构建 MCP 服务器，启用工具搜索时服务器说明字段会变得更有用。服务器说明帮助 Claude 理解何时搜索您的工具，类似于 [skills](/zh-CN/skills) 的工作方式。

添加清晰、描述性的服务器说明，说明：

* 您的工具处理的任务类别
* Claude 何时应搜索您的工具
* 您的服务器提供的关键功能

### 配置工具搜索

工具搜索默认以自动模式运行，这意味着它仅在您的 MCP 工具定义超过上下文阈值时激活。如果您的工具很少，它们会正常加载，无需工具搜索。此功能需要支持 `tool_reference` 块的模型：Sonnet 4 及更高版本，或 Opus 4 及更高版本。Haiku 模型不支持工具搜索。

使用 `ENABLE_TOOL_SEARCH` 环境变量控制工具搜索行为：

| 值          | 行为                                        |
| :--------- | :---------------------------------------- |
| `auto`     | 当 MCP 工具超过 10% 的上下文时激活（默认）                |
| `auto:<N>` | 在自定义阈值激活，其中 `<N>` 是百分比（例如，`auto:5` 表示 5%） |
| `true`     | 始终启用                                      |
| `false`    | 禁用，所有 MCP 工具预先加载                          |

```bash  theme={null}
# 使用自定义 5% 阈值
ENABLE_TOOL_SEARCH=auto:5 claude

# 完全禁用工具搜索
ENABLE_TOOL_SEARCH=false claude
```

或在您的[settings.json `env` 字段](/zh-CN/settings#available-settings)中设置值。

您还可以使用 `disallowedTools` 设置专门禁用 MCPSearch 工具：

```json  theme={null}
{
  "permissions": {
    "deny": ["MCPSearch"]
  }
}
```

## 将 MCP 提示用作命令

MCP 服务器可以公开提示，这些提示在 Claude Code 中作为命令可用。

### 执行 MCP 提示

<Steps>
  <Step title="发现可用的提示">
    键入 `/` 以查看所有可用的命令，包括来自 MCP 服务器的命令。MCP 提示以 `/mcp__servername__promptname` 的格式出现。
  </Step>

  <Step title="执行不带参数的提示">
    ```
    > /mcp__github__list_prs
    ```
  </Step>

  <Step title="执行带参数的提示">
    许多提示接受参数。在命令后以空格分隔的方式传递它们：

    ```
    > /mcp__github__pr_review 456
    ```

    ```
    > /mcp__jira__create_issue "Bug in login flow" high
    ```
  </Step>
</Steps>

<Tip>
  提示：

  * MCP 提示从连接的服务器动态发现
  * 参数根据提示的定义参数进行解析
  * 提示结果直接注入到对话中
  * 服务器和提示名称被规范化（空格变为下划线）
</Tip>

## 托管 MCP 配置

对于需要对 MCP 服务器进行集中控制的组织，Claude Code 支持两个配置选项：

1. **使用 `managed-mcp.json` 进行独占控制**：部署用户无法修改或扩展的固定 MCP 服务器集
2. **使用允许列表/拒绝列表进行基于策略的控制**：允许用户添加自己的服务器，但限制允许的服务器

这些选项允许 IT 管理员：

* **控制 MCP 服务器员工可以访问的**：在整个组织中部署一组标准化的已批准 MCP 服务器
* **防止未授权的 MCP 服务器**：限制用户添加未批准的 MCP 服务器
* **完全禁用 MCP**：如果需要，完全删除 MCP 功能

### 选项 1：使用 managed-mcp.json 进行独占控制

部署 `managed-mcp.json` 文件时，它对所有 MCP 服务器进行**独占控制**。用户无法添加、修改或使用除此文件中定义的 MCP 服务器之外的任何 MCP 服务器。这是希望完全控制的组织的最简单方法。

系统管理员将配置文件部署到系统范围的目录：

* macOS：`/Library/Application Support/ClaudeCode/managed-mcp.json`
* Linux 和 WSL：`/etc/claude-code/managed-mcp.json`
* Windows：`C:\Program Files\ClaudeCode\managed-mcp.json`

<Note>
  这些是系统范围的路径（不是像 `~/Library/...` 这样的用户主目录），需要管理员权限。它们设计为由 IT 管理员部署。
</Note>

`managed-mcp.json` 文件使用与标准 `.mcp.json` 文件相同的格式：

```json  theme={null}
{
  "mcpServers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp/"
    },
    "sentry": {
      "type": "http",
      "url": "https://mcp.sentry.dev/mcp"
    },
    "company-internal": {
      "type": "stdio",
      "command": "/usr/local/bin/company-mcp-server",
      "args": ["--config", "/etc/company/mcp-config.json"],
      "env": {
        "COMPANY_API_URL": "https://internal.company.com"
      }
    }
  }
}
```

### 选项 2：使用允许列表和拒绝列表进行基于策略的控制

管理员可以允许用户配置自己的 MCP 服务器，同时对允许的服务器强制限制，而不是进行独占控制。此方法在[托管设置文件](/zh-CN/settings#settings-files)中使用 `allowedMcpServers` 和 `deniedMcpServers`。

<Note>
  **在选项之间选择**：当您想要部署一组固定的服务器而不进行用户自定义时，使用选项 1（`managed-mcp.json`）。当您想要允许用户在策略约束内添加自己的服务器时，使用选项 2（允许列表/拒绝列表）。
</Note>

#### 限制选项

允许列表或拒绝列表中的每个条目可以通过三种方式限制服务器：

1. **按服务器名称** (`serverName`)：匹配服务器的配置名称
2. **按命令** (`serverCommand`)：匹配用于启动 stdio 服务器的确切命令和参数
3. **按 URL 模式** (`serverUrl`)：匹配带有通配符支持的远程服务器 URL

**重要**：每个条目必须恰好具有 `serverName`、`serverCommand` 或 `serverUrl` 之一。

#### 示例配置

```json  theme={null}
{
  "allowedMcpServers": [
    // 按服务器名称允许
    { "serverName": "github" },
    { "serverName": "sentry" },

    // 按确切命令允许（对于 stdio 服务器）
    { "serverCommand": ["npx", "-y", "@modelcontextprotocol/server-filesystem"] },
    { "serverCommand": ["python", "/usr/local/bin/approved-server.py"] },

    // 按 URL 模式允许（对于远程服务器）
    { "serverUrl": "https://mcp.company.com/*" },
    { "serverUrl": "https://*.internal.corp/*" }
  ],
  "deniedMcpServers": [
    // 按服务器名称阻止
    { "serverName": "dangerous-server" },

    // 按确切命令阻止（对于 stdio 服务器）
    { "serverCommand": ["npx", "-y", "unapproved-package"] },

    // 按 URL 模式阻止（对于远程服务器）
    { "serverUrl": "https://*.untrusted.com/*" }
  ]
}
```

#### 基于命令的限制如何工作

**精确匹配**：

* 命令数组必须**精确**匹配 - 命令和所有参数的顺序正确
* 示例：`["npx", "-y", "server"]` 将**不**匹配 `["npx", "server"]` 或 `["npx", "-y", "server", "--flag"]`

**Stdio 服务器行为**：

* 当允许列表包含**任何** `serverCommand` 条目时，stdio 服务器**必须**匹配其中一个命令
* Stdio 服务器在存在命令限制时无法仅按名称通过
* 这确保管理员可以强制执行允许运行的命令

**非 stdio 服务器行为**：

* 远程服务器（HTTP、SSE、WebSocket）在允许列表中存在 `serverUrl` 条目时使用基于 URL 的匹配
* 如果不存在 URL 条目，远程服务器回退到基于名称的匹配
* 命令限制不适用于远程服务器

#### 基于 URL 的限制如何工作

URL 模式使用 `*` 支持通配符以匹配任何字符序列。这对于允许整个域或子域很有用。

**通配符示例**：

* `https://mcp.company.com/*` - 允许特定域上的所有路径
* `https://*.example.com/*` - 允许 example.com 的任何子域
* `http://localhost:*/*` - 允许 localhost 上的任何端口

**远程服务器行为**：

* 当允许列表包含**任何** `serverUrl` 条目时，远程服务器**必须**匹配其中一个 URL 模式
* 远程服务器在存在 URL 限制时无法仅按名称通过
* 这确保管理员可以强制执行允许的远程端点

<Accordion title="示例：仅 URL 允许列表">
  ```json  theme={null}
  {
    "allowedMcpServers": [
      { "serverUrl": "https://mcp.company.com/*" },
      { "serverUrl": "https://*.internal.corp/*" }
    ]
  }
  ```

  **结果**：

  * `https://mcp.company.com/api` 处的 HTTP 服务器：✅ 允许（匹配 URL 模式）
  * `https://api.internal.corp/mcp` 处的 HTTP 服务器：✅ 允许（匹配通配符子域）
  * `https://external.com/mcp` 处的 HTTP 服务器：❌ 阻止（不匹配任何 URL 模式）
  * 任何命令的 Stdio 服务器：❌ 阻止（没有名称或命令条目可匹配）
</Accordion>

<Accordion title="示例：仅命令允许列表">
  ```json  theme={null}
  {
    "allowedMcpServers": [
      { "serverCommand": ["npx", "-y", "approved-package"] }
    ]
  }
  ```

  **结果**：

  * 使用 `["npx", "-y", "approved-package"]` 的 Stdio 服务器：✅ 允许（匹配命令）
  * 使用 `["node", "server.js"]` 的 Stdio 服务器：❌ 阻止（不匹配命令）
  * 名为"my-api"的 HTTP 服务器：❌ 阻止（没有名称条目可匹配）
</Accordion>

<Accordion title="示例：混合名称和命令允许列表">
  ```json  theme={null}
  {
    "allowedMcpServers": [
      { "serverName": "github" },
      { "serverCommand": ["npx", "-y", "approved-package"] }
    ]
  }
  ```

  **结果**：

  * 名为"local-tool"的 Stdio 服务器，使用 `["npx", "-y", "approved-package"]`：✅ 允许（匹配命令）
  * 名为"local-tool"的 Stdio 服务器，使用 `["node", "server.js"]`：❌ 阻止（命令条目存在但不匹配）
  * 名为"github"的 Stdio 服务器，使用 `["node", "server.js"]`：❌ 阻止（stdio 服务器在存在命令条目时必须匹配命令）
  * 名为"github"的 HTTP 服务器：✅ 允许（匹配名称）
  * 名为"other-api"的 HTTP 服务器：❌ 阻止（名称不匹配）
</Accordion>

<Accordion title="示例：仅名称允许列表">
  ```json  theme={null}
  {
    "allowedMcpServers": [
      { "serverName": "github" },
      { "serverName": "internal-tool" }
    ]
  }
  ```

  **结果**：

  * 名为"github"的 Stdio 服务器，任何命令：✅ 允许（没有命令限制）
  * 名为"internal-tool"的 Stdio 服务器，任何命令：✅ 允许（没有命令限制）
  * 名为"github"的 HTTP 服务器：✅ 允许（匹配名称）
  * 任何名为"other"的服务器：❌ 阻止（名称不匹配）
</Accordion>

#### 允许列表行为 (`allowedMcpServers`)

* `undefined`（默认）：无限制 - 用户可以配置任何 MCP 服务器
* 空数组 `[]`：完全锁定 - 用户无法配置任何 MCP 服务器
* 条目列表：用户只能配置按名称、命令或 URL 模式匹配的服务器

#### 拒绝列表行为 (`deniedMcpServers`)

* `undefined`（默认）：没有服务器被阻止
* 空数组 `[]`：没有服务器被阻止
* 条目列表：指定的服务器在所有范围内被显式阻止

#### 重要说明

* **选项 1 和选项 2 可以组合**：如果 `managed-mcp.json` 存在，它具有独占控制，用户无法添加服务器。允许列表/拒绝列表仍然适用于托管服务器本身。
* **拒绝列表具有绝对优先级**：如果服务器匹配拒绝列表条目（按名称、命令或 URL），即使它在允许列表上也会被阻止
* 基于名称、基于命令和基于 URL 的限制一起工作：如果服务器匹配**任何**名称条目、命令条目或 URL 模式，它就会通过（除非被拒绝列表阻止）

<Note>
  **使用 `managed-mcp.json` 时**：用户无法通过 `claude mcp add` 或配置文件添加 MCP 服务器。`allowedMcpServers` 和 `deniedMcpServers` 设置仍然适用于过滤实际加载的托管服务器。
</Note>
