> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# 创建和分发 plugin marketplace

> 构建和托管 plugin marketplace，以在团队和社区中分发 Claude Code 扩展。

plugin marketplace 是一个目录，让你能够将 plugin 分发给他人。Marketplace 提供集中式发现、版本跟踪、自动更新以及对多种源类型（git 仓库、本地路径等）的支持。本指南展示了如何创建自己的 marketplace，与你的团队或社区共享 plugin。

想要从现有 marketplace 安装 plugin？请参阅[发现和安装预构建 plugin](/zh-CN/discover-plugins)。

## 概述

创建和分发 marketplace 涉及：

1. **创建 plugin**：使用命令、agents、hooks、MCP servers 或 LSP servers 构建一个或多个 plugin。本指南假设你已经有要分发的 plugin；有关如何创建 plugin 的详细信息，请参阅[创建 plugin](/zh-CN/plugins)。
2. **创建 marketplace 文件**：定义一个 `marketplace.json`，列出你的 plugin 及其位置（请参阅[创建 marketplace 文件](#create-the-marketplace-file)）。
3. **托管 marketplace**：推送到 GitHub、GitLab 或其他 git 主机（请参阅[托管和分发 marketplace](#host-and-distribute-marketplaces)）。
4. **与用户共享**：用户使用 `/plugin marketplace add` 添加你的 marketplace 并安装单个 plugin（请参阅[发现和安装 plugin](/zh-CN/discover-plugins)）。

一旦你的 marketplace 上线，你可以通过推送更改到你的仓库来更新它。用户使用 `/plugin marketplace update` 刷新他们的本地副本。

## 演练：创建本地 marketplace

此示例创建一个包含一个 plugin 的 marketplace：用于代码审查的 `/review` skill。你将创建目录结构、添加 skill、创建 plugin manifest 和 marketplace 目录，然后安装并测试它。

<Steps>
  <Step title="创建目录结构">
    ```bash  theme={null}
    mkdir -p my-marketplace/.claude-plugin
    mkdir -p my-marketplace/plugins/review-plugin/.claude-plugin
    mkdir -p my-marketplace/plugins/review-plugin/skills/review
    ```
  </Step>

  <Step title="创建 skill">
    创建一个 `SKILL.md` 文件，定义 `/review` skill 的功能。

    ```markdown my-marketplace/plugins/review-plugin/skills/review/SKILL.md theme={null}
    ---
    description: Review code for bugs, security, and performance
    disable-model-invocation: true
    ---

    Review the code I've selected or the recent changes for:
    - Potential bugs or edge cases
    - Security concerns
    - Performance issues
    - Readability improvements

    Be concise and actionable.
    ```
  </Step>

  <Step title="创建 plugin manifest">
    创建一个 `plugin.json` 文件，描述该 plugin。manifest 位于 `.claude-plugin/` 目录中。

    ```json my-marketplace/plugins/review-plugin/.claude-plugin/plugin.json theme={null}
    {
      "name": "review-plugin",
      "description": "Adds a /review skill for quick code reviews",
      "version": "1.0.0"
    }
    ```
  </Step>

  <Step title="创建 marketplace 文件">
    创建列出你的 plugin 的 marketplace 目录。

    ```json my-marketplace/.claude-plugin/marketplace.json theme={null}
    {
      "name": "my-plugins",
      "owner": {
        "name": "Your Name"
      },
      "plugins": [
        {
          "name": "review-plugin",
          "source": "./plugins/review-plugin",
          "description": "Adds a /review skill for quick code reviews"
        }
      ]
    }
    ```
  </Step>

  <Step title="添加和安装">
    添加 marketplace 并安装 plugin。

    ```shell  theme={null}
    /plugin marketplace add ./my-marketplace
    /plugin install review-plugin@my-plugins
    ```
  </Step>

  <Step title="尝试一下">
    在编辑器中选择一些代码并运行你的新命令。

    ```shell  theme={null}
    /review
    ```
  </Step>
</Steps>

要了解更多关于 plugin 可以做什么的信息，包括 hooks、agents、MCP servers 和 LSP servers，请参阅 [Plugins](/zh-CN/plugins)。

<Note>
  **plugin 如何安装**：当用户安装 plugin 时，Claude Code 会将 plugin 目录复制到缓存位置。这意味着 plugin 无法使用 `../shared-utils` 之类的路径引用其目录外的文件，因为这些文件不会被复制。

  如果你需要在 plugin 之间共享文件，请使用符号链接（在复制期间会被跟踪）或重新构造你的 marketplace，使共享目录位于 plugin 源路径内。有关详细信息，请参阅 [Plugin 缓存和文件解析](/zh-CN/plugins-reference#plugin-caching-and-file-resolution)。
</Note>

## 创建 marketplace 文件

在你的仓库根目录中创建 `.claude-plugin/marketplace.json`。此文件定义你的 marketplace 的名称、所有者信息以及包含其源的 plugin 列表。

每个 plugin 条目至少需要一个 `name` 和 `source`（从哪里获取它）。有关所有可用字段，请参阅下面的[完整架构](#marketplace-schema)。

```json  theme={null}
{
  "name": "company-tools",
  "owner": {
    "name": "DevTools Team",
    "email": "devtools@example.com"
  },
  "plugins": [
    {
      "name": "code-formatter",
      "source": "./plugins/formatter",
      "description": "Automatic code formatting on save",
      "version": "2.1.0",
      "author": {
        "name": "DevTools Team"
      }
    },
    {
      "name": "deployment-tools",
      "source": {
        "source": "github",
        "repo": "company/deploy-plugin"
      },
      "description": "Deployment automation tools"
    }
  ]
}
```

## Marketplace 架构

### 必需字段

| 字段        | 类型     | 描述                                                                                                         | 示例             |
| :-------- | :----- | :--------------------------------------------------------------------------------------------------------- | :------------- |
| `name`    | string | Marketplace 标识符（kebab-case，无空格）。这是面向公众的：用户在安装 plugin 时会看到它（例如，`/plugin install my-tool@your-marketplace`）。 | `"acme-tools"` |
| `owner`   | object | Marketplace 维护者信息（[见下面的字段](#owner-fields)）                                                                 |                |
| `plugins` | array  | 可用 plugin 列表                                                                                               | 见下文            |

<Note>
  **保留名称**：以下 marketplace 名称为 Anthropic 官方使用保留，第三方 marketplace 无法使用：`claude-code-marketplace`、`claude-code-plugins`、`claude-plugins-official`、`anthropic-marketplace`、`anthropic-plugins`、`agent-skills`、`life-sciences`。冒充官方 marketplace 的名称（如 `official-claude-plugins` 或 `anthropic-tools-v2`）也被阻止。
</Note>

### 所有者字段

| 字段      | 类型     | 必需 | 描述         |
| :------ | :----- | :- | :--------- |
| `name`  | string | 是  | 维护者或团队的名称  |
| `email` | string | 否  | 维护者的联系电子邮件 |

### 可选元数据

| 字段                     | 类型     | 描述                                                                                                         |
| :--------------------- | :----- | :--------------------------------------------------------------------------------------------------------- |
| `metadata.description` | string | 简短的 marketplace 描述                                                                                         |
| `metadata.version`     | string | Marketplace 版本                                                                                             |
| `metadata.pluginRoot`  | string | 添加到相对 plugin 源路径的基目录（例如，`"./plugins"` 让你可以写 `"source": "formatter"` 而不是 `"source": "./plugins/formatter"`） |

## Plugin 条目

`plugins` 数组中的每个 plugin 条目描述一个 plugin 及其位置。你可以包含来自 [plugin manifest 架构](/zh-CN/plugins-reference#plugin-manifest-schema) 的任何字段（如 `description`、`version`、`author`、`commands`、`hooks` 等），加上这些 marketplace 特定字段：`source`、`category`、`tags` 和 `strict`。

### 必需字段

| 字段       | 类型             | 描述                                                                                         |
| :------- | :------------- | :----------------------------------------------------------------------------------------- |
| `name`   | string         | Plugin 标识符（kebab-case，无空格）。这是面向公众的：用户在安装时会看到它（例如，`/plugin install my-plugin@marketplace`）。 |
| `source` | string\|object | 从哪里获取 plugin（见下面的 [Plugin 源](#plugin-sources)）                                             |

### 可选 plugin 字段

**标准元数据字段：**

| 字段            | 类型      | 描述                                                                                                      |
| :------------ | :------ | :------------------------------------------------------------------------------------------------------ |
| `description` | string  | 简短的 plugin 描述                                                                                           |
| `version`     | string  | Plugin 版本                                                                                               |
| `author`      | object  | Plugin 作者信息（`name` 必需，`email` 可选）                                                                       |
| `homepage`    | string  | Plugin 主页或文档 URL                                                                                        |
| `repository`  | string  | 源代码仓库 URL                                                                                               |
| `license`     | string  | SPDX 许可证标识符（例如，MIT、Apache-2.0）                                                                          |
| `keywords`    | array   | 用于 plugin 发现和分类的标签                                                                                      |
| `category`    | string  | Plugin 类别以供组织                                                                                           |
| `tags`        | array   | 用于可搜索性的标签                                                                                               |
| `strict`      | boolean | 当为 true（默认）时，marketplace 组件字段与 plugin.json 合并。当为 false 时，marketplace 条目完全定义 plugin，plugin.json 不得也声明组件。 |

**组件配置字段：**

| 字段           | 类型             | 描述                         |
| :----------- | :------------- | :------------------------- |
| `commands`   | string\|array  | 命令文件或目录的自定义路径              |
| `agents`     | string\|array  | agent 文件的自定义路径             |
| `hooks`      | string\|object | 自定义 hooks 配置或 hooks 文件的路径  |
| `mcpServers` | string\|object | MCP server 配置或 MCP 配置文件的路径 |
| `lspServers` | string\|object | LSP server 配置或 LSP 配置文件的路径 |

## Plugin 源

### 相对路径

对于同一仓库中的 plugin：

```json  theme={null}
{
  "name": "my-plugin",
  "source": "./plugins/my-plugin"
}
```

<Note>
  相对路径仅在用户通过 Git（GitHub、GitLab 或 git URL）添加你的 marketplace 时有效。如果用户通过直接 URL 添加你的 marketplace 到 `marketplace.json` 文件，相对路径将无法正确解析。对于基于 URL 的分发，请改用 GitHub、npm 或 git URL 源。有关详细信息，请参阅[故障排除](#plugins-with-relative-paths-fail-in-url-based-marketplaces)。
</Note>

### GitHub 仓库

```json  theme={null}
{
  "name": "github-plugin",
  "source": {
    "source": "github",
    "repo": "owner/plugin-repo"
  }
}
```

你可以固定到特定的分支、标签或提交：

```json  theme={null}
{
  "name": "github-plugin",
  "source": {
    "source": "github",
    "repo": "owner/plugin-repo",
    "ref": "v2.0.0",
    "sha": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0"
  }
}
```

| 字段     | 类型     | 描述                               |
| :----- | :----- | :------------------------------- |
| `repo` | string | 必需。`owner/repo` 格式的 GitHub 仓库    |
| `ref`  | string | 可选。Git 分支或标签（默认为仓库默认分支）          |
| `sha`  | string | 可选。完整的 40 字符 git 提交 SHA 以固定到确切版本 |

### Git 仓库

```json  theme={null}
{
  "name": "git-plugin",
  "source": {
    "source": "url",
    "url": "https://gitlab.com/team/plugin.git"
  }
}
```

你可以固定到特定的分支、标签或提交：

```json  theme={null}
{
  "name": "git-plugin",
  "source": {
    "source": "url",
    "url": "https://gitlab.com/team/plugin.git",
    "ref": "main",
    "sha": "a1b2c3d4e5f6a7b8c9d0e1f2a3b4c5d6e7f8a9b0"
  }
}
```

| 字段    | 类型     | 描述                               |
| :---- | :----- | :------------------------------- |
| `url` | string | 必需。完整的 git 仓库 URL（必须以 `.git` 结尾） |
| `ref` | string | 可选。Git 分支或标签（默认为仓库默认分支）          |
| `sha` | string | 可选。完整的 40 字符 git 提交 SHA 以固定到确切版本 |

### 高级 plugin 条目

此示例显示了使用许多可选字段的 plugin 条目，包括命令、agents、hooks 和 MCP servers 的自定义路径：

```json  theme={null}
{
  "name": "enterprise-tools",
  "source": {
    "source": "github",
    "repo": "company/enterprise-plugin"
  },
  "description": "Enterprise workflow automation tools",
  "version": "2.1.0",
  "author": {
    "name": "Enterprise Team",
    "email": "enterprise@example.com"
  },
  "homepage": "https://docs.example.com/plugins/enterprise-tools",
  "repository": "https://github.com/company/enterprise-plugin",
  "license": "MIT",
  "keywords": ["enterprise", "workflow", "automation"],
  "category": "productivity",
  "commands": [
    "./commands/core/",
    "./commands/enterprise/",
    "./commands/experimental/preview.md"
  ],
  "agents": ["./agents/security-reviewer.md", "./agents/compliance-checker.md"],
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate.sh"
          }
        ]
      }
    ]
  },
  "mcpServers": {
    "enterprise-db": {
      "command": "${CLAUDE_PLUGIN_ROOT}/servers/db-server",
      "args": ["--config", "${CLAUDE_PLUGIN_ROOT}/config.json"]
    }
  },
  "strict": false
}
```

需要注意的关键事项：

* **`commands` 和 `agents`**：你可以指定多个目录或单个文件。路径相对于 plugin 根目录。
* **`${CLAUDE_PLUGIN_ROOT}`**：在 hooks 和 MCP server 配置中使用此变量来引用 plugin 安装目录中的文件。这是必要的，因为 plugin 在安装时被复制到缓存位置。
* **`strict: false`**：由于这设置为 false，plugin 不需要自己的 `plugin.json`。marketplace 条目定义了一切。

## 托管和分发 marketplace

### 在 GitHub 上托管（推荐）

GitHub 提供最简单的分发方法：

1. **创建仓库**：为你的 marketplace 设置一个新仓库
2. **添加 marketplace 文件**：使用你的 plugin 定义创建 `.claude-plugin/marketplace.json`
3. **与团队共享**：用户使用 `/plugin marketplace add owner/repo` 添加你的 marketplace

**优势**：内置版本控制、问题跟踪和团队协作功能。

### 在其他 git 服务上托管

任何 git 托管服务都可以工作，例如 GitLab、Bitbucket 和自托管服务器。用户使用完整的仓库 URL 添加：

```shell  theme={null}
/plugin marketplace add https://gitlab.com/company/plugins.git
```

### 私有仓库

Claude Code 支持从私有仓库安装 plugin。对于手动安装和更新，Claude Code 使用你现有的 git 凭证助手。如果 `git clone` 对你终端中的私有仓库有效，它在 Claude Code 中也有效。常见的凭证助手包括用于 GitHub 的 `gh auth login`、macOS Keychain 和 `git-credential-store`。

后台自动更新在启动时运行，不使用凭证助手，因为交互式提示会阻止 Claude Code 启动。要为私有 marketplace 启用自动更新，请在你的环境中设置适当的身份验证令牌：

| 提供商       | 环境变量                        | 注释                    |
| :-------- | :-------------------------- | :-------------------- |
| GitHub    | `GITHUB_TOKEN` 或 `GH_TOKEN` | 个人访问令牌或 GitHub App 令牌 |
| GitLab    | `GITLAB_TOKEN` 或 `GL_TOKEN` | 个人访问令牌或项目令牌           |
| Bitbucket | `BITBUCKET_TOKEN`           | 应用密码或仓库访问令牌           |

在你的 shell 配置中设置令牌（例如，`.bashrc`、`.zshrc`）或在运行 Claude Code 时传递它：

```bash  theme={null}
export GITHUB_TOKEN=ghp_xxxxxxxxxxxxxxxxxxxx
```

<Note>
  对于 CI/CD 环境，将令牌配置为秘密环境变量。GitHub Actions 自动为同一组织中的仓库提供 `GITHUB_TOKEN`。
</Note>

### 在分发前本地测试

在共享前本地测试你的 marketplace：

```shell  theme={null}
/plugin marketplace add ./my-local-marketplace
/plugin install test-plugin@my-local-marketplace
```

有关完整的添加命令范围（GitHub、Git URL、本地路径、远程 URL），请参阅[添加 marketplace](/zh-CN/discover-plugins#add-marketplaces)。

### 为你的团队要求 marketplace

你可以配置你的仓库，以便当团队成员信任项目文件夹时，他们会自动被提示安装你的 marketplace。将你的 marketplace 添加到 `.claude/settings.json`：

```json  theme={null}
{
  "extraKnownMarketplaces": {
    "company-tools": {
      "source": {
        "source": "github",
        "repo": "your-org/claude-plugins"
      }
    }
  }
}
```

你也可以指定默认应启用哪些 plugin：

```json  theme={null}
{
  "enabledPlugins": {
    "code-formatter@company-tools": true,
    "deployment-tools@company-tools": true
  }
}
```

有关完整的配置选项，请参阅 [Plugin 设置](/zh-CN/settings#plugin-settings)。

### 托管 marketplace 限制

对于需要严格控制 plugin 源的组织，管理员可以使用托管设置中的 [`strictKnownMarketplaces`](/zh-CN/settings#strictknownmarketplaces) 设置限制用户允许添加哪些 plugin marketplace。

当在托管设置中配置 `strictKnownMarketplaces` 时，限制行为取决于该值：

| 值        | 行为                           |
| -------- | ---------------------------- |
| 未定义（默认）  | 无限制。用户可以添加任何 marketplace     |
| 空数组 `[]` | 完全锁定。用户无法添加任何新 marketplace   |
| 源列表      | 用户只能添加与允许列表完全匹配的 marketplace |

#### 常见配置

禁用所有 marketplace 添加：

```json  theme={null}
{
  "strictKnownMarketplaces": []
}
```

仅允许特定 marketplace：

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
    }
  ]
}
```

使用正则表达式模式匹配允许来自内部 git 服务器的所有 marketplace：

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

#### 限制如何工作

限制在 plugin 安装过程的早期进行验证，在任何网络请求或文件系统操作发生之前。这可以防止未授权的 marketplace 访问尝试。

允许列表对大多数源类型使用精确匹配。要允许 marketplace，所有指定的字段必须完全匹配：

* 对于 GitHub 源：`repo` 是必需的，如果在允许列表中指定，`ref` 或 `path` 也必须匹配
* 对于 URL 源：完整 URL 必须完全匹配
* 对于 `hostPattern` 源：marketplace 主机与正则表达式模式匹配

因为 `strictKnownMarketplaces` 在[托管设置](/zh-CN/settings#settings-files)中设置，个人用户和项目配置无法覆盖这些限制。

有关完整的配置详细信息，包括所有支持的源类型和与 `extraKnownMarketplaces` 的比较，请参阅 [strictKnownMarketplaces 参考](/zh-CN/settings#strictknownmarketplaces)。

## 验证和测试

在共享前测试你的 marketplace。

验证你的 marketplace JSON 语法：

```bash  theme={null}
claude plugin validate .
```

或从 Claude Code 内：

```shell  theme={null}
/plugin validate .
```

添加 marketplace 进行测试：

```shell  theme={null}
/plugin marketplace add ./path/to/marketplace
```

安装测试 plugin 以验证一切正常：

```shell  theme={null}
/plugin install test-plugin@marketplace-name
```

有关完整的 plugin 测试工作流，请参阅[本地测试你的 plugin](/zh-CN/plugins#test-your-plugins-locally)。有关技术故障排除，请参阅 [Plugins 参考](/zh-CN/plugins-reference)。

## 故障排除

### Marketplace 未加载

**症状**：无法添加 marketplace 或看不到其中的 plugin

**解决方案**：

* 验证 marketplace URL 是否可访问
* 检查 `.claude-plugin/marketplace.json` 是否存在于指定路径
* 使用 `claude plugin validate` 或 `/plugin validate` 确保 JSON 语法有效
* 对于私有仓库，确认你有访问权限

### Marketplace 验证错误

从你的 marketplace 目录运行 `claude plugin validate .` 或 `/plugin validate .` 以检查问题。常见错误：

| 错误                                                | 原因               | 解决方案                                       |
| :------------------------------------------------ | :--------------- | :----------------------------------------- |
| `File not found: .claude-plugin/marketplace.json` | 缺少 manifest      | 使用必需字段创建 `.claude-plugin/marketplace.json` |
| `Invalid JSON syntax: Unexpected token...`        | JSON 语法错误        | 检查缺少的逗号、多余的逗号或未引用的字符串                      |
| `Duplicate plugin name "x" found in marketplace`  | 两个 plugin 共享相同名称 | 为每个 plugin 提供唯一的 `name` 值                  |
| `plugins[0].source: Path traversal not allowed`   | 源路径包含 `..`       | 使用相对于 marketplace 根目录的路径，不包含 `..`          |

**警告**（非阻止性）：

* `Marketplace has no plugins defined`：将至少一个 plugin 添加到 `plugins` 数组
* `No marketplace description provided`：添加 `metadata.description` 以帮助用户了解你的 marketplace
* `Plugin "x" uses npm source which is not yet fully implemented`：改用 `github` 或本地路径源

### Plugin 安装失败

**症状**：Marketplace 出现但 plugin 安装失败

**解决方案**：

* 验证 plugin 源 URL 是否可访问
* 检查 plugin 目录是否包含必需的文件
* 对于 GitHub 源，确保仓库是公开的或你有访问权限
* 通过手动克隆/下载来测试 plugin 源

### 私有仓库身份验证失败

**症状**：从私有仓库安装 plugin 时出现身份验证错误

**解决方案**：

对于手动安装和更新：

* 验证你已使用你的 git 提供商进行身份验证（例如，为 GitHub 运行 `gh auth status`）
* 检查你的凭证助手是否配置正确：`git config --global credential.helper`
* 尝试手动克隆仓库以验证你的凭证是否有效

对于后台自动更新：

* 在你的环境中设置适当的令牌：`echo $GITHUB_TOKEN`
* 检查令牌是否具有所需的权限（对仓库的读取访问权限）
* 对于 GitHub，确保令牌对私有仓库具有 `repo` 范围
* 对于 GitLab，确保令牌至少具有 `read_repository` 范围
* 验证令牌未过期

### 相对路径 plugin 在基于 URL 的 marketplace 中失败

**症状**：通过 URL（如 `https://example.com/marketplace.json`）添加了 marketplace，但具有相对路径源（如 `"./plugins/my-plugin"`）的 plugin 安装失败，出现"路径未找到"错误。

**原因**：基于 URL 的 marketplace 仅下载 `marketplace.json` 文件本身。它们不从服务器下载 plugin 文件。marketplace 条目中的相对路径引用远程服务器上未下载的文件。

**解决方案**：

* **使用外部源**：将 plugin 条目更改为使用 GitHub、npm 或 git URL 源，而不是相对路径：
  ```json  theme={null}
  { "name": "my-plugin", "source": { "source": "github", "repo": "owner/repo" } }
  ```
* **使用基于 Git 的 marketplace**：在 Git 仓库中托管你的 marketplace 并使用 git URL 添加它。基于 Git 的 marketplace 克隆整个仓库，使相对路径正常工作。

### 安装后文件未找到

**症状**：Plugin 安装但对文件的引用失败，特别是 plugin 目录外的文件

**原因**：Plugin 被复制到缓存目录而不是就地使用。引用 plugin 目录外文件的路径（如 `../shared-utils`）将无法工作，因为这些文件不会被复制。

**解决方案**：有关解决方法（包括符号链接和目录重组），请参阅 [Plugin 缓存和文件解析](/zh-CN/plugins-reference#plugin-caching-and-file-resolution)。

有关其他调试工具和常见问题，请参阅[调试和开发工具](/zh-CN/plugins-reference#debugging-and-development-tools)。

## 另请参阅

* [发现和安装预构建 plugin](/zh-CN/discover-plugins) - 从现有 marketplace 安装 plugin
* [Plugins](/zh-CN/plugins) - 创建你自己的 plugin
* [Plugins 参考](/zh-CN/plugins-reference) - 完整的技术规范和架构
* [Plugin 设置](/zh-CN/settings#plugin-settings) - Plugin 配置选项
* [strictKnownMarketplaces 参考](/zh-CN/settings#strictknownmarketplaces) - 托管 marketplace 限制
