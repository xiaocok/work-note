## LSP 协议接口完整参考

本章节系统整理 Language Server Protocol 的接口。第一部分是覆盖编辑器核心交互的 **24 个核心接口**（生命周期、文档同步、诊断、语言功能、格式化、工作区六大类）；第二部分是**扩展接口**，覆盖窗口消息（`window/*`）、基础协议（`$/*`）、扩展导航、语义高亮、层级、文件操作、动态注册等。每个接口均包含：**功能说明**、**消息类型**、**请求参数及说明**、**响应参数及说明**、**请求示例**、**响应示例**（Request 类接口）或**调用效果**（Notification 类接口，展示调用后服务器的反应及触发的后续通知）。

> 说明：
>
> - **消息类型**分为 **Request（请求）** 和 **Notification（通知）**。请求有 `id` 且必须返回响应；通知无 `id`，**无响应**。
> - **关于 `id`**：`id` 是请求的标识，由**发起请求的一方**自增分配；响应必须**原样带回**该 `id`，不自行产生新值。客户端与服务器各自维护一套独立计数器（如客户端 1、2、3…，服务器主动请求 100、101…，号段重叠无妨）。本文示例中客户端核心请求用 1~15、扩展请求 16~35、服务器主动请求 100+，仅为阅读时区分方向的约定，非协议强制。`id` 的取值约束见文末**附录**。
> - **响应示例**展示 Request 接口返回的 `result`；**调用效果**段落用于 Notification 接口，说明该通知发出后服务器会产生什么反应、回送哪些后续消息（含字段与示例）。
> - **方向**分 **C→S**（客户端→服务器）和 **S→C**（服务器→客户端）。默认是 C→S，S→C 会单独标注。
> - 文中位置坐标（`Position`）均为 LSP 0-based：`line` 行号从 0 开始，`character` 字符位置从 0 开始。
> - `Range` 表示一个区间：`{ start: Position, end: Position }`。
> - `Location` 为 `{ uri, range }`；`TextEdit` 为 `{ range, newText }`。
> - `DocumentUri` 是 `file://` 形式的文档 URI 字符串。
> - 多个接口共用的字段（位置、区间、编辑、能力、枚举等）统一收录到文末**附录**，正文中只标注"参考附录 X"，不重复展开。

### 接口总目录（59 个）

> 全部接口一览，可作为目录速查。核心 24 个（#1–#24）+ 扩展 35 个（#25–#59）。点击下方分部分章节查看每个接口的完整说明。

| #    | 方法名                                | 类型         | 方向 | 分类       | 有无响应 | 功能简述                          |
| ---- | ------------------------------------- | ------------ | ---- | ---------- | -------- | --------------------------------- |
| 1    | `initialize`                          | Request      | C→S  | 生命周期   | 有响应   | 初始化握手，交换客户端/服务器能力 |
| 2    | `initialized`                         | Notification | C→S  | 生命周期   | 无响应   | 通知服务器初始化完成              |
| 3    | `shutdown`                            | Request      | C→S  | 生命周期   | 有响应   | 请求关闭服务器（不退出进程）      |
| 4    | `exit`                                | Notification | C→S  | 生命周期   | 无响应   | 通知服务器退出进程                |
| 5    | `textDocument/didOpen`                | Notification | C→S  | 文档同步   | 无响应   | 通知文档已打开                    |
| 6    | `textDocument/didChange`              | Notification | C→S  | 文档同步   | 无响应   | 通知文档内容变更                  |
| 7    | `textDocument/willSave`               | Notification | C→S  | 文档同步   | 无响应   | 通知文档即将保存                  |
| 8    | `textDocument/didSave`                | Notification | C→S  | 文档同步   | 无响应   | 通知文档已保存                    |
| 9    | `textDocument/didClose`               | Notification | C→S  | 文档同步   | 无响应   | 通知文档已关闭                    |
| 10   | `textDocument/publishDiagnostics`     | Notification | S→C  | 诊断       | 无响应   | 服务器 → 客户端推送诊断           |
| 11   | `textDocument/completion`             | Request      | C→S  | 语言功能   | 有响应   | 请求代码补全项                    |
| 12   | `completionItem/resolve`              | Request      | C→S  | 语言功能   | 有响应   | 补全项详情懒加载                  |
| 13   | `textDocument/hover`                  | Request      | C→S  | 语言功能   | 有响应   | 请求悬停提示信息                  |
| 14   | `textDocument/signatureHelp`          | Request      | C→S  | 语言功能   | 有响应   | 请求函数签名提示                  |
| 15   | `textDocument/definition`             | Request      | C→S  | 语言功能   | 有响应   | 跳转到定义                        |
| 16   | `textDocument/references`             | Request      | C→S  | 语言功能   | 有响应   | 查找所有引用                      |
| 17   | `textDocument/documentHighlight`      | Request      | C→S  | 语言功能   | 有响应   | 文档内同名符号高亮                |
| 18   | `textDocument/documentSymbol`         | Request      | C→S  | 语言功能   | 有响应   | 获取文档符号大纲                  |
| 19   | `textDocument/codeAction`             | Request      | C→S  | 语言功能   | 有响应   | 获取快速修复/重构动作             |
| 20   | `textDocument/rename`                 | Request      | C→S  | 语言功能   | 有响应   | 重命名符号                        |
| 21   | `textDocument/formatting`             | Request      | C→S  | 格式化     | 有响应   | 格式化整个文档                    |
| 22   | `textDocument/rangeFormatting`        | Request      | C→S  | 格式化     | 有响应   | 格式化选中区域                    |
| 23   | `workspace/symbol`                    | Request      | C→S  | 工作区     | 有响应   | 全局符号搜索                      |
| 24   | `workspace/didChangeConfiguration`    | Notification | C→S  | 工作区     | 无响应   | 通知配置变更                      |
| 25   | `window/showMessage`                  | Notification | S→C  | 窗口消息   | 无响应   | 在客户端弹出提示消息              |
| 26   | `window/showMessageRequest`           | Request      | S→C  | 窗口消息   | 有响应   | 弹出带按钮的消息并返回用户选择    |
| 27   | `window/logMessage`                   | Notification | S→C  | 窗口消息   | 无响应   | 写入客户端日志输出                |
| 28   | `window/showDocument`                 | Request      | S→C  | 窗口消息   | 有响应   | 请求客户端打开文档或外部 URL      |
| 29   | `telemetry/event`                     | Notification | S→C  | 窗口消息   | 无响应   | 上报遥测事件                      |
| 30   | `$/cancelRequest`                     | Notification | 双向 | 基础协议   | 无响应   | 取消进行中的请求                  |
| 31   | `$/progress`                          | Notification | 双向 | 基础协议   | 无响应   | 上报进度                          |
| 32   | `window/workDoneProgress/create`      | Request      | S→C  | 基础协议   | 有响应   | 创建进度令牌                      |
| 33   | `window/workDoneProgress/cancel`      | Notification | C→S  | 基础协议   | 无响应   | 取消服务器发起的进度              |
| 34   | `textDocument/declaration`            | Request      | C→S  | 扩展导航   | 有响应   | 跳转到声明                        |
| 35   | `textDocument/typeDefinition`         | Request      | C→S  | 扩展导航   | 有响应   | 跳转到类型定义                    |
| 36   | `textDocument/implementation`         | Request      | C→S  | 扩展导航   | 有响应   | 跳转到实现                        |
| 37   | `textDocument/codeLens`               | Request      | C→S  | 装饰       | 有响应   | 获取 CodeLens                     |
| 38   | `codeLens/resolve`                    | Request      | C→S  | 装饰       | 有响应   | CodeLens 详情懒加载               |
| 39   | `textDocument/documentLink`           | Request      | C→S  | 装饰       | 有响应   | 获取文档内可点击链接              |
| 40   | `documentLink/resolve`                | Request      | C→S  | 装饰       | 有响应   | 文档链接详情懒加载                |
| 41   | `textDocument/documentColor`          | Request      | C→S  | 装饰       | 有响应   | 获取文档中的颜色引用              |
| 42   | `textDocument/colorPresentation`      | Request      | C→S  | 装饰       | 有响应   | 获取颜色的文本表示                |
| 43   | `textDocument/foldingRange`           | Request      | C→S  | 编辑辅助   | 有响应   | 获取代码折叠区间                  |
| 44   | `textDocument/selectionRange`         | Request      | C→S  | 编辑辅助   | 有响应   | 获取智能选区扩展                  |
| 45   | `textDocument/prepareRename`          | Request      | C→S  | 编辑辅助   | 有响应   | 重命名前校验可重命名范围          |
| 46   | `textDocument/onTypeFormatting`       | Request      | C→S  | 编辑辅助   | 有响应   | 输入时即时格式化                  |
| 47   | `textDocument/willSaveWaitUntil`      | Request      | C→S  | 编辑辅助   | 有响应   | 保存前返回需应用的编辑            |
| 48   | `textDocument/semanticTokens/full`    | Request      | C→S  | 语义高亮   | 有响应   | 获取整篇语义高亮标记              |
| 49   | `textDocument/inlayHint`              | Request      | C→S  | 语义高亮   | 有响应   | 获取内联提示（参数名、类型）      |
| 50   | `textDocument/prepareCallHierarchy`   | Request      | C→S  | 调用层级   | 有响应   | 准备调用层级                      |
| 51   | `callHierarchy/incomingCalls`         | Request      | C→S  | 调用层级   | 有响应   | 获取调用者                        |
| 52   | `callHierarchy/outgoingCalls`         | Request      | C→S  | 调用层级   | 有响应   | 获取被调用者                      |
| 53   | `workspace/executeCommand`            | Request      | C→S  | 工作区管理 | 有响应   | 执行服务器注册的命令              |
| 54   | `workspace/applyEdit`                 | Request      | S→C  | 工作区管理 | 有响应   | 请求客户端应用工作区编辑          |
| 55   | `workspace/configuration`             | Request      | S→C  | 工作区管理 | 有响应   | 拉取客户端配置                    |
| 56   | `workspace/didChangeWatchedFiles`     | Notification | C→S  | 工作区管理 | 无响应   | 通知磁盘文件变化                  |
| 57   | `workspace/didChangeWorkspaceFolders` | Notification | C→S  | 工作区管理 | 无响应   | 通知工作区文件夹变化              |
| 58   | `client/registerCapability`           | Request      | S→C  | 动态注册   | 有响应   | 动态注册服务器能力                |
| 59   | `client/unregisterCapability`         | Request      | S→C  | 动态注册   | 有响应   | 注销动态能力                      |

---

### 第一部分：核心接口总览

| #    | 方法名                             | 类型         | 方向 | 分类     | 有无响应 | 功能简述                          |
| ---- | ---------------------------------- | ------------ | ---- | -------- | -------- | --------------------------------- |
| 1    | `initialize`                       | Request      | C→S  | 生命周期 | 有响应   | 初始化握手，交换客户端/服务器能力 |
| 2    | `initialized`                      | Notification | C→S  | 生命周期 | 无响应   | 通知服务器初始化完成              |
| 3    | `shutdown`                         | Request      | C→S  | 生命周期 | 有响应   | 请求关闭服务器（不退出进程）      |
| 4    | `exit`                             | Notification | C→S  | 生命周期 | 无响应   | 通知服务器退出进程                |
| 5    | `textDocument/didOpen`             | Notification | C→S  | 文档同步 | 无响应   | 通知文档已打开                    |
| 6    | `textDocument/didChange`           | Notification | C→S  | 文档同步 | 无响应   | 通知文档内容变更                  |
| 7    | `textDocument/willSave`            | Notification | C→S  | 文档同步 | 无响应   | 通知文档即将保存                  |
| 8    | `textDocument/didSave`             | Notification | C→S  | 文档同步 | 无响应   | 通知文档已保存                    |
| 9    | `textDocument/didClose`            | Notification | C→S  | 文档同步 | 无响应   | 通知文档已关闭                    |
| 10   | `textDocument/publishDiagnostics`  | Notification | S→C  | 诊断     | 无响应   | 服务器 → 客户端推送诊断           |
| 11   | `textDocument/completion`          | Request      | C→S  | 语言功能 | 有响应   | 请求代码补全项                    |
| 12   | `completionItem/resolve`           | Request      | C→S  | 语言功能 | 有响应   | 补全项详情懒加载                  |
| 13   | `textDocument/hover`               | Request      | C→S  | 语言功能 | 有响应   | 请求悬停提示信息                  |
| 14   | `textDocument/signatureHelp`       | Request      | C→S  | 语言功能 | 有响应   | 请求函数签名提示                  |
| 15   | `textDocument/definition`          | Request      | C→S  | 语言功能 | 有响应   | 跳转到定义                        |
| 16   | `textDocument/references`          | Request      | C→S  | 语言功能 | 有响应   | 查找所有引用                      |
| 17   | `textDocument/documentHighlight`   | Request      | C→S  | 语言功能 | 有响应   | 文档内同名符号高亮                |
| 18   | `textDocument/documentSymbol`      | Request      | C→S  | 语言功能 | 有响应   | 获取文档符号大纲                  |
| 19   | `textDocument/codeAction`          | Request      | C→S  | 语言功能 | 有响应   | 获取快速修复/重构动作             |
| 20   | `textDocument/rename`              | Request      | C→S  | 语言功能 | 有响应   | 重命名符号                        |
| 21   | `textDocument/formatting`          | Request      | C→S  | 格式化   | 有响应   | 格式化整个文档                    |
| 22   | `textDocument/rangeFormatting`     | Request      | C→S  | 格式化   | 有响应   | 格式化选中区域                    |
| 23   | `workspace/symbol`                 | Request      | C→S  | 工作区   | 有响应   | 全局符号搜索                      |
| 24   | `workspace/didChangeConfiguration` | Notification | C→S  | 工作区   | 无响应   | 通知配置变更                      |

---

### 一、生命周期接口

#### 1. `initialize`（Request，C→S）

**功能**：客户端建立连接后发送的**第一个**请求。客户端借此声明自身能力（capabilities）和工作区根目录，服务器返回它支持的能力。所有其它请求都必须在 `initialize` 成功之后才能发送。

**请求参数（InitializeParams）**：

| 字段                    | 类型                             | 说明                                                     |
| ----------------------- | -------------------------------- | -------------------------------------------------------- |
| `processId`             | number \| null                   | 客户端进程 ID。浏览器环境无真实进程，传 `null`           |
| `rootUri`               | DocumentUri \| null              | 工作区根目录 URI，如 `file:///workspace`                 |
| `rootPath`              | string \| null                   | （已废弃，被 `rootUri` 取代）工作区根路径                |
| `clientInfo`            | { name, version? }               | 可选，客户端名称和版本                                   |
| `locale`                | string                           | 可选，客户端语言区域，如 `zh-cn`                         |
| `initializationOptions` | any                              | 可选，传给服务器的自定义初始化选项                       |
| `capabilities`          | ClientCapabilities               | 客户端能力声明（是否支持 completion、hover、snippet 等） |
| `trace`                 | 'off' \| 'messages' \| 'verbose' | 可选，服务器日志追踪级别                                 |
| `workspaceFolders`      | WorkspaceFolder[] \| null        | 可选，多根工作区文件夹列表                               |

上表中 `trace`、`workspaceFolders`、`initializationOptions`、`capabilities` 四个字段较为笼统，下面逐一展开。

**① `trace` 枚举值**：控制服务器通过 `$/logTrace` 通知向客户端上报的追踪日志详细程度，仅影响调试日志，不影响功能。

| 枚举值     | 含义                                                                       |
| ---------- | -------------------------------------------------------------------------- |
| `off`      | 关闭追踪。服务器不发送任何 `$/logTrace` 追踪日志（默认值）                  |
| `messages` | 简略追踪。服务器对每次收发的请求/通知输出一行概要（方法名、耗时等），不含参数正文 |
| `verbose`  | 详细追踪。在 `messages` 基础上额外附带消息的完整参数/结果正文，用于深度排错 |

> 运行期可通过 `$/setTrace` 通知动态修改该级别，无需重新 `initialize`。

**② `workspaceFolders` 子字段（WorkspaceFolder）**：数组元素结构。传 `null` 表示未打开任何文件夹（如仅打开单个游离文件）；传数组表示多根工作区。

| 字段   | 类型        | 说明                                                            |
| ------ | ----------- | --------------------------------------------------------------- |
| `uri`  | DocumentUri | 文件夹根目录 URI，如 `file:///workspace`                        |
| `name` | string      | 文件夹显示名称，用于界面展示与日志区分，如 `workspace`、`backend` |

> 当 `workspaceFolders` 与 `rootUri` 同时存在时，以 `workspaceFolders` 为准；`rootUri` 为单根时代的兼容字段。

**③ `initializationOptions` 说明**：类型为 `any`，是**与具体语言服务器强相关的自定义配置**，LSP 协议本身不规定其结构，由各服务器自行定义。客户端需查阅对应服务器文档填写。典型示例（gopls）：

```json
"initializationOptions": {
  "buildFlags": ["-tags=integration"],
  "usePlaceholders": true,
  "completeUnimported": true,
  "staticcheck": false
}
```

| 字段（gopls 示例）   | 类型     | 说明                                       |
| -------------------- | -------- | ------------------------------------------ |
| `buildFlags`         | string[] | 传给 Go 构建工具的额外参数                  |
| `usePlaceholders`    | boolean  | 补全函数时是否填充参数占位符               |
| `completeUnimported` | boolean  | 是否补全尚未 import 的包                    |
| `staticcheck`        | boolean  | 是否启用 staticcheck 静态检查              |

> 不同服务器（gopls、pyright、rust-analyzer 等）字段完全不同。该字段为空时传 `null`。

**④ `capabilities` 子字段（ClientCapabilities）**：声明**客户端自身支持哪些特性**，服务器据此裁剪返回内容（如客户端不支持 snippet，服务器就不返回带占位符的补全）。顶层分三大类：

| 顶层字段       | 类型                     | 说明                                                       |
| -------------- | ------------------------ | ---------------------------------------------------------- |
| `textDocument` | TextDocumentClientCapabilities | 文档级特性能力（补全、悬停、定义、诊断等，按方法分组）   |
| `workspace`    | WorkspaceClientCapabilities    | 工作区级特性能力（配置、文件操作、符号搜索、动态注册等） |
| `window`       | WindowClientCapabilities       | 窗口级特性能力（进度提示、消息弹窗、日志等）             |
| `general`      | GeneralClientCapabilities      | 通用特性能力（位置编码、正则引擎、Markdown 解析器等）     |

常用子字段（节选，完整结构极庞大，此处列编辑器最常声明的项）：

| 路径                                                     | 类型     | 说明                                                          |
| -------------------------------------------------------- | -------- | ------------------------------------------------------------- |
| `textDocument.completion.completionItem.snippetSupport`  | boolean  | 是否支持 snippet 占位符语法（如 `func($1)`）                  |
| `textDocument.completion.completionItem.documentationFormat` | string[] | 补全项文档支持的格式，如 `["markdown","plaintext"]`        |
| `textDocument.completion.contextSupport`                 | boolean  | 是否在补全请求中携带 `context`（触发字符等）                 |
| `textDocument.hover.contentFormat`                       | string[] | 悬停内容支持的格式，优先 `markdown` 后 `plaintext`           |
| `textDocument.signatureHelp.signatureInformation.documentationFormat` | string[] | 签名帮助文档格式                                    |
| `textDocument.definition.linkSupport`                    | boolean  | 是否支持 `LocationLink`（带跳转高亮区间）而非纯 `Location`   |
| `textDocument.publishDiagnostics.relatedInformation`     | boolean  | 是否支持诊断的关联信息（跨文件提示）                         |
| `textDocument.publishDiagnostics.tagSupport.valueSet`    | number[] | 支持的诊断标签，如 `[1,2]`（1=Unnecessary，2=Deprecated）   |
| `workspace.configuration`                                | boolean  | 是否支持服务器反向拉取配置（`workspace/configuration`）      |
| `workspace.workspaceFolders`                             | boolean  | 是否支持多根工作区动态增删                                   |
| `workspace.didChangeConfiguration.dynamicRegistration`   | boolean  | 是否支持配置变更的动态注册                                   |
| `workspace.applyEdit`                                    | boolean  | 是否支持服务器反向应用编辑（`workspace/applyEdit`）          |
| `window.workDoneProgress`                                | boolean  | 是否支持 `$/progress` 进度提示                               |
| `general.positionEncodings`                              | string[] | 支持的位置编码，如 `["utf-16","utf-8"]`                     |

> 完整字段见 LSP 规范的 `ClientCapabilities`。未声明的能力默认按"不支持"处理，服务器应做降级。

**响应参数（InitializeResult）**：

| 字段           | 类型               | 说明                                                         |
| -------------- | ------------------ | ------------------------------------------------------------ |
| `capabilities` | ServerCapabilities | 服务器支持的能力，如 `completionProvider.triggerCharacters`、`textDocumentSync`、`hoverProvider` 等 |
| `serverInfo`   | { name, version? } | 可选，服务器名称和版本                                       |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "initialize",
  "params": {
    "processId": null,
    "rootUri": "file:///workspace",
    "clientInfo": { "name": "monaco-lsp-client", "version": "1.0.0" },
    "locale": "zh-cn",
    "initializationOptions": null,
    "capabilities": {
      "textDocument": {
        "completion": {
          "completionItem": { "snippetSupport": true }
        },
        "hover": { "contentFormat": ["markdown", "plaintext"] }
      }
    },
    "trace": "off",
    "workspaceFolders": [
      { "uri": "file:///workspace", "name": "workspace" }
    ]
  }
}
```

**响应示例**（服务器返回它支持的能力）：

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "capabilities": {
      "textDocumentSync": 2,
      "completionProvider": {
        "triggerCharacters": ["."],
        "resolveProvider": true
      },
      "hoverProvider": true,
      "signatureHelpProvider": { "triggerCharacters": ["(", ","] },
      "definitionProvider": true,
      "referencesProvider": true,
      "documentSymbolProvider": true,
      "renameProvider": { "prepareProvider": true },
      "documentFormattingProvider": true
    },
    "serverInfo": { "name": "gopls", "version": "0.15.0" }
  }
}
```

#### 2. `initialized`（Notification，C→S）

**功能**：客户端在**收到 `initialize` 响应之后**发送的通知，告知服务器初始化流程已完成，服务器此后可以发送动态注册等请求。

**请求参数（InitializedParams）**：空对象 `{}`，无字段。

**响应参数**：**无响应**（通知类消息）。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "method": "initialized",
  "params": {}
}
```

**调用效果**：该通知本身无返回。它解除服务器的"初始化中"状态，此后服务器可主动发起 `client/registerCapability`（动态注册能力）、`window/logMessage`、`$/progress`（如开始索引工作区）等消息。典型地，服务器会回送一条日志通知：

```json
{
  "jsonrpc": "2.0",
  "method": "window/logMessage",
  "params": { "type": 3, "message": "gopls 已完成初始化，开始索引工作区" }
}
```

#### 3. `shutdown`（Request，C→S）

**功能**：请求服务器关闭（释放资源、停止分析），但**不退出进程**。服务器应在收到后停止接受除 `exit` 外的新请求。

**请求参数**：无参数（`params` 可省略或为 `null`）。

**响应参数**：

| 字段     | 类型 | 说明                                      |
| -------- | ---- | ----------------------------------------- |
| `result` | null | 固定返回 `null`，仅表示服务器已准备好退出 |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "method": "shutdown"
}
```

**响应示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "result": null
}
```

#### 4. `exit`（Notification，C→S）

**功能**：通知服务器**立即退出进程**。通常在 `shutdown` 成功响应之后发送。

**请求参数**：无参数。

**响应参数**：**无响应**（通知类消息）。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "method": "exit"
}
```

**调用效果**：该通知无返回。服务器收到后立即终止进程：若此前收到过 `shutdown` 则以退出码 `0` 退出，否则以退出码 `1` 退出。此后连接关闭，不再有任何消息往来。

---

### 二、文档同步接口

#### 5. `textDocument/didOpen`（Notification，C→S）

**功能**：通知服务器某个文档已在编辑器中打开。服务器以此为权威内容来源开始管理该文档（覆盖磁盘内容），并可触发诊断。

**请求参数（DidOpenTextDocumentParams）**：

| 字段                      | 类型        | 说明                                      |
| ------------------------- | ----------- | ----------------------------------------- |
| `textDocument.uri`        | DocumentUri | 文档 URI，后续所有文档消息必须保持一致    |
| `textDocument.languageId` | string      | 语言标识，如 `go`、`python`、`typescript` |
| `textDocument.version`    | number      | 文档版本号，打开时通常为 `1`              |
| `textDocument.text`       | string      | 文档完整初始内容                          |

**响应参数**：**无响应**（通知类消息）。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "method": "textDocument/didOpen",
  "params": {
    "textDocument": {
      "uri": "file:///workspace/main.go",
      "languageId": "go",
      "version": 1,
      "text": "package main\n\nfunc main() {\n}\n"
    }
  }
}
```

**调用效果**：该通知无返回。服务器开始以此内容为权威来源管理该文档，解析后**主动回送** `textDocument/publishDiagnostics` 通知推送诊断（无错误时 `diagnostics` 为空数组，用于清空旧标记）：

```json
{
  "jsonrpc": "2.0",
  "method": "textDocument/publishDiagnostics",
  "params": {
    "uri": "file:///workspace/main.go",
    "version": 1,
    "diagnostics": []
  }
}
```

#### 6. `textDocument/didChange`（Notification，C→S）

**功能**：通知服务器文档内容发生变化。可发送增量变更或全文替换，取决于服务器声明的 `textDocumentSync` 模式。

**请求参数（DidChangeTextDocumentParams）**：

| 字段                           | 类型                             | 说明                                                 |
| ------------------------------ | -------------------------------- | ---------------------------------------------------- |
| `textDocument.uri`             | DocumentUri                      | 发生变化的文档 URI                                   |
| `textDocument.version`         | number                           | 变更后的版本号，必须**严格递增**                     |
| `contentChanges`               | TextDocumentContentChangeEvent[] | 变更数组                                             |
| `contentChanges[].range`       | Range                            | 可选。增量模式下表示被替换的区间；省略时表示全文替换 |
| `contentChanges[].rangeLength` | number                           | 可选（已废弃），被替换区间的长度                     |
| `contentChanges[].text`        | string                           | 新文本。增量模式下是替换文本；全文模式下是整篇内容   |

**响应参数**：**无响应**（通知类消息）。

**请求示例**（增量变更：在第 2 行第 0 列插入一行内容）：

```json
{
  "jsonrpc": "2.0",
  "method": "textDocument/didChange",
  "params": {
    "textDocument": {
      "uri": "file:///workspace/main.go",
      "version": 2
    },
    "contentChanges": [
      {
        "range": {
          "start": { "line": 3, "character": 0 },
          "end": { "line": 3, "character": 0 }
        },
        "text": "\tfmt.Println(\"hello\")\n"
      }
    ]
  }
}
```

**请求示例**（全文替换，省略 `range`）：

```json
{
  "jsonrpc": "2.0",
  "method": "textDocument/didChange",
  "params": {
    "textDocument": {
      "uri": "file:///workspace/main.go",
      "version": 3
    },
    "contentChanges": [
      { "text": "package main\n\nfunc main() {\n\tprintln(\"hi\")\n}\n" }
    ]
  }
}
```

**调用效果**：该通知无返回。服务器更新其内存中的文档内容与版本号，重新分析后**主动回送** `textDocument/publishDiagnostics`（通常带与本次变更一致的 `version`）。例如引用了未导入的 `fmt` 时会推送一条错误诊断：

```json
{
  "jsonrpc": "2.0",
  "method": "textDocument/publishDiagnostics",
  "params": {
    "uri": "file:///workspace/main.go",
    "version": 3,
    "diagnostics": [
      {
        "range": {
          "start": { "line": 3, "character": 1 },
          "end": { "line": 3, "character": 4 }
        },
        "severity": 1,
        "code": "UndeclaredName",
        "source": "gopls",
        "message": "undefined: fmt"
      }
    ]
  }
}
```

#### 7. `textDocument/willSave`（Notification，C→S）

**功能**：通知服务器文档**即将**保存（保存动作发生前）。允许服务器在保存前做准备工作。

**请求参数（WillSaveTextDocumentParams）**：

| 字段               | 类型                   | 说明                                                         |
| ------------------ | ---------------------- | ------------------------------------------------------------ |
| `textDocument.uri` | DocumentUri            | 即将保存的文档 URI                                           |
| `reason`           | TextDocumentSaveReason | 保存原因：`1=Manual`（手动）、`2=AfterDelay`（延时自动）、`3=FocusOut`（失焦） |

**响应参数**：**无响应**（通知类消息）。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "method": "textDocument/willSave",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "reason": 1
  }
}
```

**调用效果**：该通知无返回，服务器仅收到"即将保存"的信号用于内部准备，**不会**回送任何消息、也不会修改文档。若需要在保存前修改文档内容，应改用请求版本 `textDocument/willSaveWaitUntil`（见接口 47），它会返回 `TextEdit[]`。

> 提示：还有一个对应的请求版本 `textDocument/willSaveWaitUntil`（Request），它会返回 `TextEdit[]`，允许服务器在保存前修改文档（如自动格式化）。本表聚焦无响应的通知版本。

#### 8. `textDocument/didSave`（Notification，C→S）

**功能**：通知服务器文档**已经**保存到磁盘。注意：`didSave` 不负责落盘，仅通知保存事件，真实落盘由编辑器/业务接口完成。

**请求参数（DidSaveTextDocumentParams）**：

| 字段               | 类型        | 说明                                                         |
| ------------------ | ----------- | ------------------------------------------------------------ |
| `textDocument.uri` | DocumentUri | 已保存文档 URI                                               |
| `text`             | string      | 可选。保存后的完整内容，仅当服务器声明 `save.includeText` 时才需携带 |

**响应参数**：**无响应**（通知类消息）。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "method": "textDocument/didSave",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "text": "package main\n\nfunc main() {\n}\n"
  }
}
```

**调用效果**：该通知无返回。服务器据此触发与"保存"绑定的逻辑（如运行 lint、整理 import、构建），随后通常**主动回送** `textDocument/publishDiagnostics` 刷新诊断。部分服务器还会借此机会发送 `window/logMessage` 报告保存后任务的进展。例如保存后 lint 通过、清空诊断：

```json
{
  "jsonrpc": "2.0",
  "method": "textDocument/publishDiagnostics",
  "params": {
    "uri": "file:///workspace/main.go",
    "diagnostics": []
  }
}
```

#### 9. `textDocument/didClose`（Notification，C→S）

**功能**：通知服务器文档已关闭。服务器此后不再以编辑器内容为权威，应回退到磁盘文件内容，并通常清空该文档的诊断。

**请求参数（DidCloseTextDocumentParams）**：

| 字段               | 类型        | 说明           |
| ------------------ | ----------- | -------------- |
| `textDocument.uri` | DocumentUri | 已关闭文档 URI |

**响应参数**：**无响应**（通知类消息）。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "method": "textDocument/didClose",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" }
  }
}
```

**调用效果**：该通知无返回。服务器不再以编辑器内容为该文档的权威来源，回退到磁盘文件内容，并通常**主动回送**一条空 `diagnostics` 的 `textDocument/publishDiagnostics` 以清除编辑器中该文档的所有诊断标记：

```json
{
  "jsonrpc": "2.0",
  "method": "textDocument/publishDiagnostics",
  "params": {
    "uri": "file:///workspace/main.go",
    "diagnostics": []
  }
}
```

---

### 三、诊断接口

#### 10. `textDocument/publishDiagnostics`（Notification，服务器 → 客户端）

**功能**：服务器**主动**向客户端推送某个文档的诊断信息（错误、警告、提示）。客户端将其转换为编辑器标记（如 Monaco markers）。这是少数由服务器发起的通知。

**请求参数（PublishDiagnosticsParams）**：

| 字段                     | 类型             | 说明                                                        |
| ------------------------ | ---------------- | ----------------------------------------------------------- |
| `uri`                    | DocumentUri      | 诊断所属文档 URI                                            |
| `version`                | number           | 可选，诊断对应的文档版本号                                  |
| `diagnostics`            | Diagnostic[]     | 诊断数组。空数组 `[]` 表示清空该文档的旧诊断                |
| `diagnostics[].range`    | Range            | 诊断对应的代码区间                                          |
| `diagnostics[].severity` | number           | 严重级别：`1=Error`、`2=Warning`、`3=Information`、`4=Hint` |
| `diagnostics[].code`     | string \| number | 可选，诊断代码                                              |
| `diagnostics[].source`   | string           | 可选，诊断来源，如 `gopls`、`pyright`                       |
| `diagnostics[].message`  | string           | 诊断文本                                                    |
| `diagnostics[].tags`     | number[]         | 可选，标签：`1=Unnecessary`（冗余）、`2=Deprecated`（弃用） |
| `diagnostics[].relatedInformation` | DiagnosticRelatedInformation[] | 可选，关联信息数组，每项为 `{ location: Location, message: string }`，用于指向相关代码位置 |
| `diagnostics[].codeDescription`    | { href: URI }    | 可选，诊断代码的说明文档链接                                |
| `diagnostics[].data`     | any              | 可选，服务器私有数据，`codeAction` 解析快速修复时回传      |

**响应参数**：**无响应**（通知类消息）。

**请求示例**（由服务器发往客户端）：

```json
{
  "jsonrpc": "2.0",
  "method": "textDocument/publishDiagnostics",
  "params": {
    "uri": "file:///workspace/main.go",
    "version": 2,
    "diagnostics": [
      {
        "range": {
          "start": { "line": 3, "character": 1 },
          "end": { "line": 3, "character": 4 }
        },
        "severity": 1,
        "code": "UndeclaredName",
        "source": "gopls",
        "message": "undefined: fmt",
        "tags": []
      }
    ]
  }
}
```

**调用效果**：这是一条 S→C 通知，**客户端无需回复**。客户端收到后将 `diagnostics` 转换为编辑器标记（如 Monaco 的 markers、VS Code 的波浪线与"问题"面板条目）：`severity=1` 渲染为红色错误标记，`2` 为黄色警告等。收到空数组 `[]` 时清空该文档的所有标记。同一文档后续每次 `publishDiagnostics` 都是**全量替换**，而非增量追加。

---

### 四、语言功能接口

#### 11. `textDocument/completion`（Request，C→S）

**功能**：请求指定位置可用的代码补全项。

**请求参数（CompletionParams）**：

| 字段                       | 类型        | 说明                                                         |
| -------------------------- | ----------- | ------------------------------------------------------------ |
| `textDocument.uri`         | DocumentUri | 目标文档 URI，必须是当前已打开文档                           |
| `position.line`            | number      | 光标所在行，0-based                                          |
| `position.character`       | number      | 光标所在字符位置，0-based                                    |
| `context.triggerKind`      | number      | 可选，触发方式：`1=Invoked`（主动触发）、`2=TriggerCharacter`、`3=TriggerForIncompleteCompletions` |
| `context.triggerCharacter` | string      | 可选，触发字符（如 `.`），仅当 `triggerKind=2` 时存在        |

**响应参数（CompletionItem[] \| CompletionList \| null）**：

| 字段                       | 类型                    | 说明                                                         |
| -------------------------- | ----------------------- | ------------------------------------------------------------ |
| `isIncomplete`             | boolean                 | （CompletionList）`true` 表示列表不完整，输入更多字符需重新请求 |
| `items`                    | CompletionItem[]        | 补全项数组（也可直接返回 CompletionItem[]）                  |
| `items[].label`            | string                  | 补全列表显示名称，通常也是默认插入文本                       |
| `items[].kind`             | number                  | 补全项类型枚举：如 `3=Function`、`6=Variable`、`7=Class`、`9=Module`、`14=Keyword` |
| `items[].detail`           | string                  | 可选，右侧补充说明（如函数签名）                             |
| `items[].documentation`    | string \| MarkupContent | 可选，文档说明，可能是字符串或 `{ kind, value }`             |
| `items[].insertText`       | string                  | 可选，实际插入文本，缺省回退到 `label`                       |
| `items[].insertTextFormat` | number                  | 可选，`1=PlainText`、`2=Snippet`（带占位符）                 |
| `items[].preselect`        | boolean                 | 可选，`true` 表示默认预选中该项                              |
| `items[].tags`             | number[]                | 可选，补全项标签，如 `1=Deprecated`（划掉显示）             |
| `items[].textEdit`         | TextEdit                | 可选，精确替换范围 + 文本，优先于 `insertText`               |
| `items[].sortText`         | string                  | 可选，排序用键                                               |
| `items[].filterText`       | string                  | 可选，过滤用键                                               |
| `items[].additionalTextEdits` | TextEdit[]           | 可选，应用该项时附带的其它编辑（如自动补 import）            |
| `items[].commitCharacters` | string[]                | 可选，触发提交该补全项的字符                                 |
| `items[].command`          | Command                 | 可选，插入补全项后要执行的命令                               |
| `items[].data`             | any                     | 可选，`completionItem/resolve` 时原样回传的私有数据         |
| `result` 为 `null`         | null                    | 表示无补全项                                                 |

上表中 `items[].kind`、`items[].documentation`、`items[].command` 三个字段较为笼统，下面逐一展开。

**① `items[].kind` 枚举值（CompletionItemKind）**：决定补全项在列表中显示的图标，整数 `1~25`。客户端据此渲染对应类型图标。

| 值 | 名称        | 值 | 名称       | 值 | 名称          | 值 | 名称          | 值 | 名称          |
| -- | ----------- | -- | ---------- | -- | ------------- | -- | ------------- | -- | ------------- |
| 1  | Text        | 6  | Variable   | 11 | Unit          | 16 | Color         | 21 | Constant      |
| 2  | Method      | 7  | Class      | 12 | Value         | 17 | File          | 22 | Struct        |
| 3  | Function    | 8  | Interface  | 13 | Enum          | 18 | Reference     | 23 | Event         |
| 4  | Constructor | 9  | Module     | 14 | Keyword       | 19 | Folder        | 24 | Operator      |
| 5  | Field       | 10 | Property   | 15 | Snippet       | 20 | EnumMember    | 25 | TypeParameter |

**② `items[].documentation` 子结构（string \| MarkupContent）**：可为纯字符串，或 `MarkupContent` 对象。`MarkupContent` 是 LSP 中通用的富文本结构，`hover`、`signatureHelp` 等多个接口复用。

| 字段    | 类型   | 说明                                                |
| ------- | ------ | --------------------------------------------------- |
| `kind`  | string | 内容格式：`plaintext`（纯文本）或 `markdown`（Markdown） |
| `value` | string | 内容正文，按 `kind` 解析                            |

**③ `items[].command` 子结构（Command）**：补全插入后要执行的命令。`Command` 也是通用结构，`codeAction`、`codeLens` 等接口复用。

| 字段        | 类型   | 说明                                                       |
| ----------- | ------ | ---------------------------------------------------------- |
| `title`     | string | 命令显示标题                                               |
| `command`   | string | 命令标识符，须为服务器/客户端已注册的命令                  |
| `arguments` | any[]  | 可选，传给命令的参数数组                                    |

**请求示例**（在 `.` 后触发补全）：

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "method": "textDocument/completion",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "position": { "line": 3, "character": 5 },
    "context": {
      "triggerKind": 2,
      "triggerCharacter": "."
    }
  }
}
```

**响应示例**（返回补全列表）：

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "result": {
    "isIncomplete": false,
    "items": [
      {
        "label": "Println",
        "kind": 3,
        "detail": "func(a ...any) (n int, err error)",
        "documentation": "Println formats using the default formats...",
        "insertText": "Println(${1:a})",
        "insertTextFormat": 2,
        "sortText": "00000",
        "data": { "uri": "file:///workspace/main.go", "id": 42 }
      },
      {
        "label": "Printf",
        "kind": 3,
        "detail": "func(format string, a ...any) (n int, err error)",
        "insertText": "Printf(${1:format})",
        "insertTextFormat": 2,
        "sortText": "00001"
      }
    ]
  }
}
```

#### 12. `completionItem/resolve`（Request，C→S）

**功能**：对单个补全项做**详情懒加载**。当 `completion` 返回的项缺少 `detail`、`documentation` 等较重字段时，编辑器在用户选中该项时再请求补全这些信息，以提升首次补全性能。

**请求参数（CompletionItem）**：传入一个待补全详情的 `CompletionItem`（即 `completion` 返回的某一项原样回传）。

**响应参数（CompletionItem）**：返回补全了 `detail`、`documentation`、`additionalTextEdits` 等字段后的同一个 `CompletionItem`。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "method": "completionItem/resolve",
  "params": {
    "label": "Println",
    "kind": 3,
    "data": { "uri": "file:///workspace/main.go", "id": 42 }
  }
}
```

**响应示例**（补全了 `detail`、`documentation`、`additionalTextEdits` 等较重字段）：

```json
{
  "jsonrpc": "2.0",
  "id": 4,
  "result": {
    "label": "Println",
    "kind": 3,
    "detail": "func fmt.Println(a ...any) (n int, err error)",
    "documentation": {
      "kind": "markdown",
      "value": "Println formats using the default formats for its operands and writes to standard output."
    },
    "additionalTextEdits": [
      {
        "range": {
          "start": { "line": 1, "character": 0 },
          "end": { "line": 1, "character": 0 }
        },
        "newText": "import \"fmt\"\n"
      }
    ]
  }
}
```

#### 13. `textDocument/hover`（Request，C→S）

**功能**：请求指定位置的悬停提示信息（类型、文档注释等）。

**请求参数（HoverParams）**：

| 字段                 | 类型        | 说明                  |
| -------------------- | ----------- | --------------------- |
| `textDocument.uri`   | DocumentUri | 目标文档 URI          |
| `position.line`      | number      | 光标行，0-based       |
| `position.character` | number      | 光标字符位置，0-based |

**响应参数（Hover \| null）**：

| 字段               | 类型                                            | 说明                                           |
| ------------------ | ----------------------------------------------- | ---------------------------------------------- |
| `contents`         | MarkupContent \| MarkedString \| MarkedString[] | 悬停内容，常见为 `{ kind: 'markdown', value }` |
| `range`            | Range                                           | 可选，悬停信息覆盖的代码区间                   |
| `result` 为 `null` | null                                            | 该位置无悬停信息                               |

`contents` 字段类型笼统，可取以下三种形式之一：

**① `MarkupContent`（推荐）**：通用富文本结构，同 `completion` 接口 `items[].documentation`（见接口 #11）。

| 字段    | 类型   | 说明                                            |
| ------- | ------ | ----------------------------------------------- |
| `kind`  | string | `plaintext` 或 `markdown`                       |
| `value` | string | 内容正文                                        |

**② `MarkedString`（已废弃）**：旧式结构，为字符串，或 `{ language, value }` 对象（其中 `language` 用于代码块语法高亮，`value` 为代码正文）。新实现应改用 `MarkupContent`。

**③ `MarkedString[]`**：上述 `MarkedString` 的数组，用于拼接多段内容。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 5,
  "method": "textDocument/hover",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "position": { "line": 3, "character": 6 }
  }
}
```

**响应示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 5,
  "result": {
    "contents": {
      "kind": "markdown",
      "value": "```go\nfunc fmt.Println(a ...any) (n int, err error)\n```\n\nPrintln 使用默认格式输出并换行。"
    },
    "range": {
      "start": { "line": 3, "character": 5 },
      "end": { "line": 3, "character": 12 }
    }
  }
}
```

#### 14. `textDocument/signatureHelp`（Request，C→S）

**功能**：在函数调用处请求签名提示（参数列表、当前高亮参数）。通常由 `(` 或 `,` 触发。

**请求参数（SignatureHelpParams）**：

| 字段                          | 类型          | 说明                                                         |
| ----------------------------- | ------------- | ------------------------------------------------------------ |
| `textDocument.uri`            | DocumentUri   | 目标文档 URI                                                 |
| `position`                    | Position      | 光标位置，0-based                                            |
| `context.triggerKind`         | number        | 可选，触发方式：`1=Invoked`、`2=TriggerCharacter`、`3=ContentChange` |
| `context.triggerCharacter`    | string        | 可选，触发字符，如 `(`、`,`                                  |
| `context.isRetrigger`         | boolean       | 可选，是否为重新触发                                         |
| `context.activeSignatureHelp` | SignatureHelp | 可选，上一次的签名提示                                       |

**响应参数（SignatureHelp \| null）**：

| 字段                         | 类型                    | 说明                                      |
| ---------------------------- | ----------------------- | ----------------------------------------- |
| `signatures`                 | SignatureInformation[]  | 候选签名数组                              |
| `signatures[].label`         | string                  | 签名文本，如 `func add(a int, b int) int` |
| `signatures[].documentation` | string \| MarkupContent | 可选，签名说明                            |
| `signatures[].parameters`    | ParameterInformation[]  | 参数列表，每项含 `label`、`documentation` |
| `signatures[].activeParameter` | number                | 可选，本签名内当前高亮参数索引（优先于顶层 `activeParameter`） |
| `activeSignature`            | number                  | 当前高亮的签名索引                        |
| `activeParameter`            | number                  | 当前高亮的参数索引                        |
| `result` 为 `null`           | null                    | 无签名提示                                |

**请求示例**（在 `(` 后触发）：

```json
{
  "jsonrpc": "2.0",
  "id": 6,
  "method": "textDocument/signatureHelp",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "position": { "line": 3, "character": 12 },
    "context": {
      "triggerKind": 2,
      "triggerCharacter": "(",
      "isRetrigger": false
    }
  }
}
```

**响应示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 6,
  "result": {
    "signatures": [
      {
        "label": "func add(a int, b int) int",
        "documentation": "返回两数之和",
        "parameters": [
          { "label": "a int" },
          { "label": "b int" }
        ],
        "activeParameter": 0
      }
    ],
    "activeSignature": 0,
    "activeParameter": 0
  }
}
```

#### 15. `textDocument/definition`（Request，C→S）

**功能**：请求某符号的定义位置（跳转到定义）。

**请求参数（DefinitionParams）**：

| 字段               | 类型        | 说明                      |
| ------------------ | ----------- | ------------------------- |
| `textDocument.uri` | DocumentUri | 目标文档 URI              |
| `position`         | Position    | 符号所在光标位置，0-based |

**响应参数（Location \| Location[] \| LocationLink[] \| null）**：

| 字段                   | 类型        | 说明                                    |
| ---------------------- | ----------- | --------------------------------------- |
| `uri`                  | DocumentUri | （Location）定义所在文档 URI            |
| `range`                | Range       | （Location）定义所在区间                |
| `targetUri`            | DocumentUri | （LocationLink）目标文档 URI            |
| `targetRange`          | Range       | （LocationLink）目标完整区间            |
| `targetSelectionRange` | Range       | （LocationLink）目标中应选中/高亮的区间 |
| `result` 为 `null`     | null        | 未找到定义                              |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 7,
  "method": "textDocument/definition",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "position": { "line": 10, "character": 8 }
  }
}
```

**响应示例**（返回单个 `Location`）：

```json
{
  "jsonrpc": "2.0",
  "id": 7,
  "result": {
    "uri": "file:///workspace/util.go",
    "range": {
      "start": { "line": 4, "character": 5 },
      "end": { "line": 4, "character": 12 }
    }
  }
}
```

#### 16. `textDocument/references`（Request，C→S）

**功能**：查找某符号在工作区中的所有引用位置。

**请求参数（ReferenceParams）**：

| 字段                         | 类型        | 说明                          |
| ---------------------------- | ----------- | ----------------------------- |
| `textDocument.uri`           | DocumentUri | 目标文档 URI                  |
| `position`                   | Position    | 符号位置，0-based             |
| `context.includeDeclaration` | boolean     | 是否在结果中包含定义/声明本身 |

**响应参数（Location[] \| null）**：

| 字段               | 类型        | 说明             |
| ------------------ | ----------- | ---------------- |
| `[].uri`           | DocumentUri | 引用所在文档 URI |
| `[].range`         | Range       | 引用所在区间     |
| `result` 为 `null` | null        | 无引用           |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 8,
  "method": "textDocument/references",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "position": { "line": 10, "character": 8 },
    "context": { "includeDeclaration": true }
  }
}
```

**响应示例**（返回 `Location[]`）：

```json
{
  "jsonrpc": "2.0",
  "id": 8,
  "result": [
    {
      "uri": "file:///workspace/util.go",
      "range": {
        "start": { "line": 4, "character": 5 },
        "end": { "line": 4, "character": 12 }
      }
    },
    {
      "uri": "file:///workspace/main.go",
      "range": {
        "start": { "line": 10, "character": 8 },
        "end": { "line": 10, "character": 15 }
      }
    }
  ]
}
```

#### 17. `textDocument/documentHighlight`（Request，C→S）

**功能**：在**当前文档内**高亮与光标处符号相关的所有出现位置（如同名变量的读/写）。

**请求参数（DocumentHighlightParams）**：

| 字段               | 类型        | 说明              |
| ------------------ | ----------- | ----------------- |
| `textDocument.uri` | DocumentUri | 目标文档 URI      |
| `position`         | Position    | 光标位置，0-based |

**响应参数（DocumentHighlight[] \| null）**：

| 字段               | 类型   | 说明                                          |
| ------------------ | ------ | --------------------------------------------- |
| `[].range`         | Range  | 高亮区间                                      |
| `[].kind`          | number | 可选，高亮类型：`1=Text`、`2=Read`、`3=Write` |
| `result` 为 `null` | null   | 无高亮                                        |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 9,
  "method": "textDocument/documentHighlight",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "position": { "line": 5, "character": 4 }
  }
}
```

**响应示例**（同名符号的读/写出现位置）：

```json
{
  "jsonrpc": "2.0",
  "id": 9,
  "result": [
    {
      "range": {
        "start": { "line": 5, "character": 4 },
        "end": { "line": 5, "character": 9 }
      },
      "kind": 3
    },
    {
      "range": {
        "start": { "line": 7, "character": 12 },
        "end": { "line": 7, "character": 17 }
      },
      "kind": 2
    }
  ]
}
```

#### 18. `textDocument/documentSymbol`（Request，C→S）

**功能**：获取当前文档的符号大纲（函数、类、变量等），用于面包屑、大纲视图（如 `Ctrl+Shift+O`）。

**请求参数（DocumentSymbolParams）**：

| 字段               | 类型        | 说明         |
| ------------------ | ----------- | ------------ |
| `textDocument.uri` | DocumentUri | 目标文档 URI |

**响应参数（DocumentSymbol[] \| SymbolInformation[] \| null）**：

| 字段                | 类型             | 说明                                                         |
| ------------------- | ---------------- | ------------------------------------------------------------ |
| `[].name`           | string           | 符号名称                                                     |
| `[].kind`           | number           | 符号类型枚举：如 `5=Class`、`6=Method`、`12=Function`、`13=Variable` |
| `[].detail`         | string           | 可选，（DocumentSymbol）符号详情                             |
| `[].tags`           | number[]         | 可选，符号标签，如 `1=Deprecated`                            |
| `[].range`          | Range            | （DocumentSymbol）符号完整区间                               |
| `[].selectionRange` | Range            | （DocumentSymbol）符号名称区间                               |
| `[].children`       | DocumentSymbol[] | 可选，子符号，可构成层级树                                   |
| `[].location`       | Location         | （SymbolInformation）符号位置                                |
| `[].containerName`  | string           | 可选，（SymbolInformation）所属容器名                        |
| `result` 为 `null`  | null             | 无符号                                                       |

其中 `[].kind` 字段（SymbolKind）取整数 `1~26`，决定符号在大纲中显示的图标。该枚举被 `workspace/symbol`（#23）、`prepareCallHierarchy`（#50）等接口复用：

| 值 | 名称      | 值 | 名称        | 值 | 名称       | 值 | 名称          | 值 | 名称          |
| -- | --------- | -- | ----------- | -- | ---------- | -- | ------------- | -- | ------------- |
| 1  | File      | 7  | Property    | 13 | Variable   | 19 | Object        | 25 | Operator      |
| 2  | Module    | 8  | Field       | 14 | Constant   | 20 | Key           | 26 | TypeParameter |
| 3  | Namespace | 9  | Constructor | 15 | String     | 21 | Null          |    |               |
| 4  | Package   | 10 | Enum        | 16 | Number     | 22 | EnumMember    |    |               |
| 5  | Class     | 11 | Interface   | 17 | Boolean    | 23 | Struct        |    |               |
| 6  | Method    | 12 | Function    | 18 | Array      | 24 | Event         |    |               |

> `[].tags` 取值同诊断标签思路，目前仅 `1=Deprecated`（在大纲中以删除线显示）。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 10,
  "method": "textDocument/documentSymbol",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" }
  }
}
```

**响应示例**（返回 `DocumentSymbol[]`，含层级）：

```json
{
  "jsonrpc": "2.0",
  "id": 10,
  "result": [
    {
      "name": "main",
      "kind": 12,
      "range": {
        "start": { "line": 2, "character": 0 },
        "end": { "line": 6, "character": 1 }
      },
      "selectionRange": {
        "start": { "line": 2, "character": 5 },
        "end": { "line": 2, "character": 9 }
      },
      "children": []
    }
  ]
}
```

#### 19. `textDocument/codeAction`（Request，C→S）

**功能**：获取指定区间内可用的代码动作（快速修复、重构、源代码操作，如自动导入、提取函数）。

**请求参数（CodeActionParams）**：

| 字段                  | 类型         | 说明                                                |
| --------------------- | ------------ | --------------------------------------------------- |
| `textDocument.uri`    | DocumentUri  | 目标文档 URI                                        |
| `range`               | Range        | 触发代码动作的区间（通常是选区或诊断位置）          |
| `context.diagnostics` | Diagnostic[] | 该区间相关的诊断，供生成快速修复                    |
| `context.only`        | string[]     | 可选，限定返回的动作类型，如 `quickfix`、`refactor` |
| `context.triggerKind` | number       | 可选，`1=Invoked`、`2=Automatic`                    |

**响应参数（(Command \| CodeAction)[] \| null）**：

| 字段               | 类型          | 说明                                              |
| ------------------ | ------------- | ------------------------------------------------- |
| `[].title`         | string        | 动作显示标题                                      |
| `[].kind`          | string        | 可选，动作类型，如 `quickfix`、`refactor.extract` |
| `[].diagnostics`   | Diagnostic[]  | 可选，该动作要解决的诊断                          |
| `[].edit`          | WorkspaceEdit | 可选，应用该动作产生的文档编辑                    |
| `[].command`       | Command       | 可选，要执行的命令                                |
| `[].isPreferred`   | boolean       | 可选，是否为首选动作                              |
| `[].disabled`      | { reason: string } | 可选，该动作当前不可用及其原因                |
| `[].data`          | any           | 可选，`codeAction/resolve` 时回传的私有数据       |
| `result` 为 `null` | null          | 无可用动作                                        |

上表中 `context.only`、`[].kind` 取值（CodeActionKind）及 `[].command` 较为笼统，下面展开。

**① CodeActionKind 取值**：`context.only`（请求侧过滤）与 `[].kind`（响应侧标注）共用同一组**点分层级字符串**，支持前缀匹配（如 `refactor` 匹配所有 `refactor.*`）。

| 值                       | 含义                                             |
| ------------------------ | ------------------------------------------------ |
| `quickfix`               | 快速修复（针对某诊断的修复，如自动导入）         |
| `refactor`               | 重构（总类）                                     |
| `refactor.extract`       | 提取（提取函数、变量、常量等）                   |
| `refactor.inline`        | 内联（与提取相反）                               |
| `refactor.rewrite`       | 重写（其它形式的等价改写）                       |
| `source`                 | 源代码操作（作用于整个文件，非特定位置）         |
| `source.organizeImports` | 整理导入                                         |
| `source.fixAll`          | 修复所有可自动修复的问题                         |

**② `[].command` 子结构（Command）**：同 `completion` 的 `items[].command`（见接口 #11），为 `{ title, command, arguments? }`。`edit` 与 `command` 可同时存在，此时先应用 `edit` 再执行 `command`。

**请求示例**（针对一处诊断请求快速修复）：

```json
{
  "jsonrpc": "2.0",
  "id": 11,
  "method": "textDocument/codeAction",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "range": {
      "start": { "line": 3, "character": 1 },
      "end": { "line": 3, "character": 4 }
    },
    "context": {
      "diagnostics": [
        {
          "range": {
            "start": { "line": 3, "character": 1 },
            "end": { "line": 3, "character": 4 }
          },
          "severity": 1,
          "message": "undefined: fmt",
          "source": "gopls"
        }
      ],
      "only": ["quickfix"],
      "triggerKind": 1
    }
  }
}
```

**响应示例**（返回带 `edit` 的快速修复动作）：

```json
{
  "jsonrpc": "2.0",
  "id": 11,
  "result": [
    {
      "title": "添加 import \"fmt\"",
      "kind": "quickfix",
      "diagnostics": [
        {
          "range": {
            "start": { "line": 3, "character": 1 },
            "end": { "line": 3, "character": 4 }
          },
          "severity": 1,
          "message": "undefined: fmt",
          "source": "gopls"
        }
      ],
      "isPreferred": true,
      "edit": {
        "changes": {
          "file:///workspace/main.go": [
            {
              "range": {
                "start": { "line": 1, "character": 0 },
                "end": { "line": 1, "character": 0 }
              },
              "newText": "import \"fmt\"\n"
            }
          ]
        }
      }
    }
  ]
}
```

#### 20. `textDocument/rename`（Request，C→S）

**功能**：对某符号执行重命名，返回需要在整个工作区应用的编辑集合。

**请求参数（RenameParams）**：

| 字段               | 类型        | 说明                          |
| ------------------ | ----------- | ----------------------------- |
| `textDocument.uri` | DocumentUri | 目标文档 URI                  |
| `position`         | Position    | 被重命名符号所在位置，0-based |
| `newName`          | string      | 新名称                        |

**响应参数（WorkspaceEdit \| null）**：

| 字段               | 类型                      | 说明                                       |
| ------------------ | ------------------------- | ------------------------------------------ |
| `changes`          | { [uri]: TextEdit[] }     | 按文档 URI 分组的文本编辑（旧式格式）      |
| `documentChanges`  | TextDocumentEdit[] \| ... | 可选，带版本号的文档编辑（新式格式，推荐） |
| `result` 为 `null` | null                      | 无法重命名                                 |

> 其中 `TextEdit` 为 `{ range: Range, newText: string }`，表示在某区间替换为新文本。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 12,
  "method": "textDocument/rename",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "position": { "line": 10, "character": 8 },
    "newName": "newFuncName"
  }
}
```

**响应示例**（返回 `WorkspaceEdit`，按文档分组的编辑）：

```json
{
  "jsonrpc": "2.0",
  "id": 12,
  "result": {
    "changes": {
      "file:///workspace/main.go": [
        {
          "range": {
            "start": { "line": 10, "character": 8 },
            "end": { "line": 10, "character": 15 }
          },
          "newText": "newFuncName"
        }
      ],
      "file:///workspace/util.go": [
        {
          "range": {
            "start": { "line": 4, "character": 5 },
            "end": { "line": 4, "character": 12 }
          },
          "newText": "newFuncName"
        }
      ]
    }
  }
}
```

#### 21. `textDocument/formatting`（Request，C→S）

**功能**：格式化**整个文档**，返回需要应用的文本编辑。

**请求参数（DocumentFormattingParams）**：

| 字段                             | 类型        | 说明                           |
| -------------------------------- | ----------- | ------------------------------ |
| `textDocument.uri`               | DocumentUri | 目标文档 URI                   |
| `options.tabSize`                | number      | 一个制表符等于多少空格         |
| `options.insertSpaces`           | boolean     | 是否用空格代替制表符           |
| `options.trimTrailingWhitespace` | boolean     | 可选，是否删除行尾空白         |
| `options.insertFinalNewline`     | boolean     | 可选，是否在文件末尾插入换行   |
| `options.trimFinalNewlines`      | boolean     | 可选，是否删除文件末尾多余空行 |

**响应参数（TextEdit[] \| null）**：

| 字段               | 类型   | 说明           |
| ------------------ | ------ | -------------- |
| `[].range`         | Range  | 被替换的区间   |
| `[].newText`       | string | 替换后的新文本 |
| `result` 为 `null` | null   | 无需格式化     |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 13,
  "method": "textDocument/formatting",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "options": {
      "tabSize": 4,
      "insertSpaces": false,
      "trimTrailingWhitespace": true,
      "insertFinalNewline": true
    }
  }
}
```

**响应示例**（返回 `TextEdit[]`）：

```json
{
  "jsonrpc": "2.0",
  "id": 13,
  "result": [
    {
      "range": {
        "start": { "line": 3, "character": 0 },
        "end": { "line": 3, "character": 8 }
      },
      "newText": "\tfmt.Println(\"hello\")"
    }
  ]
}
```

#### 22. `textDocument/rangeFormatting`（Request，C→S）

**功能**：仅格式化文档中**选中的区域**，返回需要应用的文本编辑。

**请求参数（DocumentRangeFormattingParams）**：

| 字段               | 类型              | 说明                                                         |
| ------------------ | ----------------- | ------------------------------------------------------------ |
| `textDocument.uri` | DocumentUri       | 目标文档 URI                                                 |
| `range`            | Range             | 要格式化的区间                                               |
| `options`          | FormattingOptions | 同 `formatting` 的 `options`（`tabSize`、`insertSpaces` 等） |

**响应参数（TextEdit[] \| null）**：

| 字段               | 类型   | 说明           |
| ------------------ | ------ | -------------- |
| `[].range`         | Range  | 被替换的区间   |
| `[].newText`       | string | 替换后的新文本 |
| `result` 为 `null` | null   | 无需格式化     |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 14,
  "method": "textDocument/rangeFormatting",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "range": {
      "start": { "line": 2, "character": 0 },
      "end": { "line": 5, "character": 0 }
    },
    "options": {
      "tabSize": 4,
      "insertSpaces": false
    }
  }
}
```

**响应示例**（返回选区内的 `TextEdit[]`）：

```json
{
  "jsonrpc": "2.0",
  "id": 14,
  "result": [
    {
      "range": {
        "start": { "line": 3, "character": 0 },
        "end": { "line": 3, "character": 2 }
      },
      "newText": "\t"
    }
  ]
}
```

---

### 五、工作区接口

#### 23. `workspace/symbol`（Request，C→S）

**功能**：在**整个工作区**按名称搜索符号（函数、类、变量等），对应 VS Code 的 `Ctrl+T`。

**请求参数（WorkspaceSymbolParams）**：

| 字段    | 类型   | 说明                                     |
| ------- | ------ | ---------------------------------------- |
| `query` | string | 搜索关键字，空字符串通常表示请求全部符号 |

**响应参数（SymbolInformation[] \| WorkspaceSymbol[] \| null）**：

| 字段               | 类型     | 说明                                      |
| ------------------ | -------- | ----------------------------------------- |
| `[].name`          | string   | 符号名称                                  |
| `[].kind`          | number   | 符号类型枚举，如 `12=Function`、`5=Class` |
| `[].location`      | Location | 符号位置（`{ uri, range }`）              |
| `[].containerName` | string   | 可选，所属容器名（如类名、包名）          |
| `result` 为 `null` | null     | 无匹配符号                                |

> `[].kind` 为 SymbolKind 枚举（整数 `1~26`），完整取值见接口 #18 `documentSymbol`。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 15,
  "method": "workspace/symbol",
  "params": {
    "query": "main"
  }
}
```

**响应示例**（返回 `SymbolInformation[]`）：

```json
{
  "jsonrpc": "2.0",
  "id": 15,
  "result": [
    {
      "name": "main",
      "kind": 12,
      "location": {
        "uri": "file:///workspace/main.go",
        "range": {
          "start": { "line": 2, "character": 0 },
          "end": { "line": 6, "character": 1 }
        }
      },
      "containerName": "main"
    }
  ]
}
```

#### 24. `workspace/didChangeConfiguration`（Notification，C→S）

**功能**：通知服务器客户端配置发生变化，服务器据此调整行为（如启用/禁用某些诊断）。

**请求参数（DidChangeConfigurationParams）**：

| 字段       | 类型 | 说明                                       |
| ---------- | ---- | ------------------------------------------ |
| `settings` | any  | 变更后的配置对象，结构由具体语言服务器约定 |

**响应参数**：**无响应**（通知类消息）。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "method": "workspace/didChangeConfiguration",
  "params": {
    "settings": {
      "go": {
        "lintTool": "golangci-lint",
        "formatTool": "gofmt"
      }
    }
  }
}
```

**调用效果**：该通知无返回。服务器据新配置调整行为（启用/禁用诊断、切换工具链等）。常见后续反应有两类：一是服务器**反向发起** `workspace/configuration` 请求向客户端按 section 拉取更细的配置；二是因规则变化而对已打开文档重新分析并回送 `textDocument/publishDiagnostics`。例如服务器回拉配置：

```json
{
  "jsonrpc": "2.0",
  "id": 104,
  "method": "workspace/configuration",
  "params": {
    "items": [{ "section": "go" }]
  }
}
```

---

### 接口分类速查

| 分类     | 接口                                                         | 数量   |
| -------- | ------------------------------------------------------------ | ------ |
| 生命周期 | `initialize`、`initialized`、`shutdown`、`exit`              | 4      |
| 文档同步 | `didOpen`、`didChange`、`willSave`、`didSave`、`didClose`    | 5      |
| 诊断     | `publishDiagnostics`                                         | 1      |
| 语言功能 | `completion`、`completionItem/resolve`、`hover`、`signatureHelp`、`definition`、`references`、`documentHighlight`、`documentSymbol`、`codeAction`、`rename` | 10     |
| 格式化   | `formatting`、`rangeFormatting`                              | 2      |
| 工作区   | `workspace/symbol`、`workspace/didChangeConfiguration`       | 2      |
| **合计** |                                                              | **24** |

> 备注：以上 24 个为覆盖编辑器核心交互的常用接口。本项目当前实际用到的是其中的 `initialize`、`initialized`、`didOpen`、`didChange`、`didSave`、`completion`、`publishDiagnostics` 等子集。其余接口见下方"第二部分：扩展接口"。

---

## 第二部分：扩展接口

除核心 24 个接口外，LSP 还定义了大量扩展接口。本部分按用途分组，逐个给出功能、消息类型、请求/响应参数说明（通知类明确标注无响应）。注意这里有不少是 **服务器 → 客户端（S→C）** 方向的消息。

### 扩展接口总览

| #    | 方法名                                | 类型         | 方向 | 分类     | 有无响应 | 功能简述                       |
| ---- | ------------------------------------- | ------------ | ---- | -------- | -------- | ------------------------------ |
| 25   | `window/showMessage`                  | Notification | S→C  | 窗口消息 | 无响应   | 在客户端弹出提示消息           |
| 26   | `window/showMessageRequest`           | Request      | S→C  | 窗口消息 | 有响应   | 弹出带按钮的消息并返回用户选择 |
| 27   | `window/logMessage`                   | Notification | S→C  | 窗口消息 | 无响应   | 写入客户端日志输出             |
| 28   | `window/showDocument`                 | Request      | S→C  | 窗口消息 | 有响应   | 请求客户端打开文档或外部 URL   |
| 29   | `telemetry/event`                     | Notification | S→C  | 窗口消息 | 无响应   | 上报遥测事件                   |
| 30   | `$/cancelRequest`                     | Notification | 双向 | 基础协议 | 无响应   | 取消进行中的请求               |
| 31   | `$/progress`                          | Notification | 双向 | 基础协议 | 无响应   | 上报进度                       |
| 32   | `window/workDoneProgress/create`      | Request      | S→C  | 基础协议 | 有响应   | 创建进度令牌                   |
| 33   | `window/workDoneProgress/cancel`      | Notification | C→S  | 基础协议 | 无响应   | 取消服务器发起的进度           |
| 34   | `textDocument/declaration`            | Request      | C→S  | 扩展导航 | 有响应   | 跳转到声明                     |
| 35   | `textDocument/typeDefinition`         | Request      | C→S  | 扩展导航 | 有响应   | 跳转到类型定义                 |
| 36   | `textDocument/implementation`         | Request      | C→S  | 扩展导航 | 有响应   | 跳转到实现                     |
| 37   | `textDocument/codeLens`               | Request      | C→S  | 装饰     | 有响应   | 获取 CodeLens                  |
| 38   | `codeLens/resolve`                    | Request      | C→S  | 装饰     | 有响应   | CodeLens 详情懒加载            |
| 39   | `textDocument/documentLink`           | Request      | C→S  | 装饰     | 有响应   | 获取文档内可点击链接           |
| 40   | `documentLink/resolve`                | Request      | C→S  | 装饰     | 有响应   | 文档链接详情懒加载             |
| 41   | `textDocument/documentColor`          | Request      | C→S  | 装饰     | 有响应   | 获取文档中的颜色引用           |
| 42   | `textDocument/colorPresentation`      | Request      | C→S  | 装饰     | 有响应   | 获取颜色的文本表示             |
| 43   | `textDocument/foldingRange`           | Request      | C→S  | 编辑辅助 | 有响应   | 获取代码折叠区间               |
| 44   | `textDocument/selectionRange`         | Request      | C→S  | 编辑辅助 | 有响应   | 获取智能选区扩展               |
| 45   | `textDocument/prepareRename`          | Request      | C→S  | 编辑辅助 | 有响应   | 重命名前校验可重命名范围       |
| 46   | `textDocument/onTypeFormatting`       | Request      | C→S  | 编辑辅助 | 有响应   | 输入时即时格式化               |
| 47   | `textDocument/willSaveWaitUntil`      | Request      | C→S  | 编辑辅助 | 有响应   | 保存前返回需应用的编辑         |
| 48   | `textDocument/semanticTokens/full`    | Request      | C→S  | 语义高亮 | 有响应   | 获取整篇语义高亮标记           |
| 49   | `textDocument/inlayHint`              | Request      | C→S  | 语义高亮 | 有响应   | 获取内联提示（参数名、类型）   |
| 50   | `textDocument/prepareCallHierarchy`   | Request      | C→S  | 层级     | 有响应   | 准备调用层级                   |
| 51   | `callHierarchy/incomingCalls`         | Request      | C→S  | 层级     | 有响应   | 获取调用者                     |
| 52   | `callHierarchy/outgoingCalls`         | Request      | C→S  | 层级     | 有响应   | 获取被调用者                   |
| 53   | `workspace/executeCommand`            | Request      | C→S  | 工作区   | 有响应   | 执行服务器注册的命令           |
| 54   | `workspace/applyEdit`                 | Request      | S→C  | 工作区   | 有响应   | 请求客户端应用工作区编辑       |
| 55   | `workspace/configuration`             | Request      | S→C  | 工作区   | 有响应   | 拉取客户端配置                 |
| 56   | `workspace/didChangeWatchedFiles`     | Notification | C→S  | 工作区   | 无响应   | 通知磁盘文件变化               |
| 57   | `workspace/didChangeWorkspaceFolders` | Notification | C→S  | 工作区   | 无响应   | 通知工作区文件夹变化           |
| 58   | `client/registerCapability`           | Request      | S→C  | 动态注册 | 有响应   | 动态注册服务器能力             |
| 59   | `client/unregisterCapability`         | Request      | S→C  | 动态注册 | 有响应   | 注销动态能力                   |

---

### 六、窗口消息（`window/*`，服务器 → 客户端）

#### 25. `window/showMessage`（Notification，S→C）

**功能**：服务器请求客户端在 UI 上弹出一条提示消息（如右下角通知）。仅展示，不需要用户响应。

**请求参数（ShowMessageParams）**：

| 字段      | 类型   | 说明                                                |
| --------- | ------ | --------------------------------------------------- |
| `type`    | number | 消息类型：`1=Error`、`2=Warning`、`3=Info`、`4=Log` |
| `message` | string | 要显示的消息文本                                    |

**响应参数**：**无响应**（通知类消息）。

**请求示例**（由服务器发往客户端）：

```json
{
  "jsonrpc": "2.0",
  "method": "window/showMessage",
  "params": {
    "type": 3,
    "message": "Go language server 已就绪"
  }
}
```

**调用效果**：S→C 通知，**客户端无需回复**。客户端按 `type` 在 UI 弹出对应级别的提示（如 VS Code 右下角通知、Monaco 自定义提示条）：`1` 红色错误、`2` 黄色警告、`3` 信息、`4` 日志。仅展示，不收集用户反馈；若需要用户点击按钮并回传结果，应使用请求版本 `window/showMessageRequest`（接口 26）。

#### 26. `window/showMessageRequest`（Request，S→C）

**功能**：与 `showMessage` 类似，但带可选操作按钮，并等待用户选择后返回结果（如"是否重新加载？[是] [否]"）。

**请求参数（ShowMessageRequestParams）**：

| 字段      | 类型                | 说明                                                |
| --------- | ------------------- | --------------------------------------------------- |
| `type`    | number              | 消息类型：`1=Error`、`2=Warning`、`3=Info`、`4=Log` |
| `message` | string              | 消息文本                                            |
| `actions` | MessageActionItem[] | 可选，按钮列表，每项为 `{ title: string }`          |

**响应参数（MessageActionItem \| null）**：

| 字段               | 类型   | 说明                           |
| ------------------ | ------ | ------------------------------ |
| `title`            | string | 用户点击的按钮标题             |
| `result` 为 `null` | null   | 用户未选择任何按钮（关闭消息） |

**请求示例**（由服务器发往客户端）：

```json
{
  "jsonrpc": "2.0",
  "id": 100,
  "method": "window/showMessageRequest",
  "params": {
    "type": 2,
    "message": "配置已变更，是否重新加载？",
    "actions": [
      { "title": "重新加载" },
      { "title": "稍后" }
    ]
  }
}
```

**响应示例**（由客户端返回用户的选择）：

```json
{
  "jsonrpc": "2.0",
  "id": 100,
  "result": { "title": "重新加载" }
}
```

#### 27. `window/logMessage`（Notification，S→C）

**功能**：服务器请求客户端把一条消息写入日志输出通道（通常不弹窗，仅记录到输出面板）。

**请求参数（LogMessageParams）**：

| 字段      | 类型   | 说明                                                |
| --------- | ------ | --------------------------------------------------- |
| `type`    | number | 日志级别：`1=Error`、`2=Warning`、`3=Info`、`4=Log` |
| `message` | string | 日志文本                                            |

**响应参数**：**无响应**（通知类消息）。

**请求示例**（由服务器发往客户端）：

```json
{
  "jsonrpc": "2.0",
  "method": "window/logMessage",
  "params": {
    "type": 4,
    "message": "[trace] 开始解析 main.go"
  }
}
```

**调用效果**：S→C 通知，**客户端无需回复**。客户端将消息按 `type` 级别写入日志/输出通道（如 VS Code 的 Output 面板对应通道），通常**不弹窗打扰用户**，仅供排查问题时查看。与 `showMessage` 的区别在于：`logMessage` 进日志、`showMessage` 进通知 UI。

#### 28. `window/showDocument`（Request，S→C，3.16+）

**功能**：服务器请求客户端打开某个文档或外部 URL（如打开浏览器查看文档、跳转到某文件某位置）。

**请求参数（ShowDocumentParams）**：

| 字段        | 类型    | 说明                                      |
| ----------- | ------- | ----------------------------------------- |
| `uri`       | URI     | 要打开的文档或外部链接 URI                |
| `external`  | boolean | 可选，`true` 表示用外部程序（浏览器）打开 |
| `takeFocus` | boolean | 可选，打开后是否聚焦                      |
| `selection` | Range   | 可选，打开文档后要选中的区间              |

**响应参数（ShowDocumentResult）**：

| 字段      | 类型    | 说明         |
| --------- | ------- | ------------ |
| `success` | boolean | 是否成功打开 |

**请求示例**（由服务器发往客户端，打开外部文档）：

```json
{
  "jsonrpc": "2.0",
  "id": 101,
  "method": "window/showDocument",
  "params": {
    "uri": "https://pkg.go.dev/fmt",
    "external": true,
    "takeFocus": true
  }
}
```

**响应示例**（由客户端返回是否成功打开）：

```json
{
  "jsonrpc": "2.0",
  "id": 101,
  "result": { "success": true }
}
```

#### 29. `telemetry/event`（Notification，S→C）

**功能**：服务器向客户端上报遥测/统计事件，由客户端决定如何处理（通常转发给遥测系统）。

**请求参数**：

| 字段     | 类型 | 说明                             |
| -------- | ---- | -------------------------------- |
| `params` | any  | 任意遥测载荷，结构由服务器自定义 |

**响应参数**：**无响应**（通知类消息）。

**请求示例**（由服务器发往客户端）：

```json
{
  "jsonrpc": "2.0",
  "method": "telemetry/event",
  "params": {
    "name": "completionLatency",
    "durationMs": 42,
    "itemCount": 18
  }
}
```

**调用效果**：S→C 通知，**客户端无需回复**。客户端把该遥测载荷转发给自身的遥测系统（如 VS Code 的 telemetry 通道）；若用户关闭了遥测或客户端未实现该能力，则**静默丢弃**。不影响编辑器 UI，也不会触发任何后续消息。

---

### 七、基础协议（`$/*` 与进度）

#### 30. `$/cancelRequest`（Notification，双向）

**功能**：取消一个仍在处理中的请求。发起请求的一方在不再需要结果时发送，接收方应尽量提前结束该请求。

**请求参数（CancelParams）**：

| 字段 | 类型             | 说明                |
| ---- | ---------------- | ------------------- |
| `id` | number \| string | 要取消的请求的 `id` |

**响应参数**：**无响应**（通知类消息）。被取消的请求本身会以错误码 `RequestCancelled` 返回。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "method": "$/cancelRequest",
  "params": {
    "id": 3
  }
}
```

**调用效果**：该通知本身无返回。接收方应尽量提前结束 `id=3` 的请求处理；被取消的那个**原请求**会以一个错误响应结束（错误码 `-32800 RequestCancelled`），而非正常 `result`：

```json
{
  "jsonrpc": "2.0",
  "id": 3,
  "error": { "code": -32800, "message": "Request cancelled" }
}
```

#### 31. `$/progress`（Notification，双向）

**功能**：通用进度上报通道，配合进度令牌（token）使用，可用于工作进度条、流式部分结果等。

**请求参数（ProgressParams）**：

| 字段    | 类型             | 说明                                                         |
| ------- | ---------------- | ------------------------------------------------------------ |
| `token` | number \| string | 进度令牌，标识本次进度流                                     |
| `value` | any              | 进度载荷。对 WorkDoneProgress 为 `{ kind: 'begin'\|'report'\|'end', title?, message?, percentage? }` |

`value` 字段笼统，最常见的载荷是 **WorkDoneProgress**，由 `kind` 区分三个阶段，字段各异：

**① `kind: "begin"`（开始）**：

| 字段          | 类型    | 说明                                                       |
| ------------- | ------- | ---------------------------------------------------------- |
| `kind`        | string  | 固定为 `begin`                                             |
| `title`       | string  | 进度标题，如"正在索引工作区"（必填）                       |
| `message`     | string  | 可选，详细文案，如"0/120 文件"                             |
| `percentage`  | number  | 可选，进度百分比 `0~100`                                   |
| `cancellable` | boolean | 可选，是否显示取消按钮（用户取消会触发 `window/workDoneProgress/cancel`） |

**② `kind: "report"`（更新）**：

| 字段          | 类型    | 说明                                   |
| ------------- | ------- | -------------------------------------- |
| `kind`        | string  | 固定为 `report`                        |
| `message`     | string  | 可选，更新后的文案                     |
| `percentage`  | number  | 可选，更新后的百分比                   |
| `cancellable` | boolean | 可选，更新是否可取消                   |

**③ `kind: "end"`（结束）**：

| 字段      | 类型   | 说明                       |
| --------- | ------ | -------------------------- |
| `kind`    | string | 固定为 `end`               |
| `message` | string | 可选，收尾文案，如"索引完成" |

**响应参数**：**无响应**（通知类消息）。

**请求示例**（进度开始，由服务器发往客户端）：

```json
{
  "jsonrpc": "2.0",
  "method": "$/progress",
  "params": {
    "token": "indexing-1",
    "value": {
      "kind": "begin",
      "title": "正在索引工作区",
      "message": "0/120 文件",
      "percentage": 0,
      "cancellable": true
    }
  }
}
```

**调用效果**：该通知无返回。接收方（通常是客户端）按 `token` 找到对应的进度 UI 并更新。一次完整的进度流由同一 `token` 的多条 `$/progress` 组成：`begin` → 若干 `report`（更新百分比/文案）→ `end`（结束并关闭进度条）。后续 `report` 与 `end` 示例：

```json
{
  "jsonrpc": "2.0",
  "method": "$/progress",
  "params": {
    "token": "indexing-1",
    "value": { "kind": "report", "message": "60/120 文件", "percentage": 50 }
  }
}
```

```json
{
  "jsonrpc": "2.0",
  "method": "$/progress",
  "params": {
    "token": "indexing-1",
    "value": { "kind": "end", "message": "索引完成" }
  }
}
```

#### 32. `window/workDoneProgress/create`（Request，S→C）

**功能**：服务器请求客户端创建一个工作进度令牌，随后服务器用该令牌通过 `$/progress` 发送进度。

**请求参数（WorkDoneProgressCreateParams）**：

| 字段    | 类型             | 说明                     |
| ------- | ---------------- | ------------------------ |
| `token` | number \| string | 服务器希望使用的进度令牌 |

**响应参数**：

| 字段     | 类型 | 说明                          |
| -------- | ---- | ----------------------------- |
| `result` | null | 成功返回 `null`；失败返回错误 |

**请求示例**（由服务器发往客户端）：

```json
{
  "jsonrpc": "2.0",
  "id": 102,
  "method": "window/workDoneProgress/create",
  "params": {
    "token": "indexing-1"
  }
}
```

**响应示例**（由客户端确认创建成功）：

```json
{
  "jsonrpc": "2.0",
  "id": 102,
  "result": null
}
```

#### 33. `window/workDoneProgress/cancel`（Notification，C→S）

**功能**：客户端通知服务器，用户取消了某个由服务器发起的进度操作。

**请求参数（WorkDoneProgressCancelParams）**：

| 字段    | 类型             | 说明             |
| ------- | ---------------- | ---------------- |
| `token` | number \| string | 要取消的进度令牌 |

**响应参数**：**无响应**（通知类消息）。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "method": "window/workDoneProgress/cancel",
  "params": {
    "token": "indexing-1"
  }
}
```

**调用效果**：该通知无返回。服务器收到后应中止 `token` 对应的后台操作，并以一条 `kind: "end"` 的 `$/progress` 收尾、关闭进度条：

```json
{
  "jsonrpc": "2.0",
  "method": "$/progress",
  "params": {
    "token": "indexing-1",
    "value": { "kind": "end", "message": "已取消" }
  }
}
```

---

### 八、扩展导航接口

#### 34. `textDocument/declaration`（Request）

**功能**：跳转到符号的**声明**位置（区别于定义，某些语言声明与定义分离，如 C 头文件）。

**请求参数（DeclarationParams）**：

| 字段               | 类型        | 说明              |
| ------------------ | ----------- | ----------------- |
| `textDocument.uri` | DocumentUri | 目标文档 URI      |
| `position`         | Position    | 符号位置，0-based |

**响应参数（Location \| Location[] \| LocationLink[] \| null）**：

| 字段               | 类型                | 说明               |
| ------------------ | ------------------- | ------------------ |
| `uri` / `range`    | DocumentUri / Range | 声明所在文档与区间 |
| `result` 为 `null` | null                | 未找到声明         |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 16,
  "method": "textDocument/declaration",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "position": { "line": 8, "character": 10 }
  }
}
```

**响应示例**（返回声明所在 `Location`）：

```json
{
  "jsonrpc": "2.0",
  "id": 16,
  "result": {
    "uri": "file:///workspace/types.go",
    "range": {
      "start": { "line": 2, "character": 5 },
      "end": { "line": 2, "character": 11 }
    }
  }
}
```

#### 35. `textDocument/typeDefinition`（Request）

**功能**：跳转到某符号的**类型定义**（如变量跳转到其类型的定义处）。

**请求参数（TypeDefinitionParams）**：

| 字段               | 类型        | 说明              |
| ------------------ | ----------- | ----------------- |
| `textDocument.uri` | DocumentUri | 目标文档 URI      |
| `position`         | Position    | 符号位置，0-based |

**响应参数（Location \| Location[] \| LocationLink[] \| null）**：

| 字段               | 类型                | 说明                   |
| ------------------ | ------------------- | ---------------------- |
| `uri` / `range`    | DocumentUri / Range | 类型定义所在文档与区间 |
| `result` 为 `null` | null                | 未找到类型定义         |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 17,
  "method": "textDocument/typeDefinition",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "position": { "line": 12, "character": 6 }
  }
}
```

**响应示例**（返回类型定义所在 `Location`）：

```json
{
  "jsonrpc": "2.0",
  "id": 17,
  "result": {
    "uri": "file:///workspace/types.go",
    "range": {
      "start": { "line": 5, "character": 5 },
      "end": { "line": 5, "character": 9 }
    }
  }
}
```

#### 36. `textDocument/implementation`（Request）

**功能**：跳转到接口/抽象方法的**具体实现**位置（可能有多个）。

**请求参数（ImplementationParams）**：

| 字段               | 类型        | 说明              |
| ------------------ | ----------- | ----------------- |
| `textDocument.uri` | DocumentUri | 目标文档 URI      |
| `position`         | Position    | 符号位置，0-based |

**响应参数（Location \| Location[] \| LocationLink[] \| null）**：

| 字段               | 类型                | 说明               |
| ------------------ | ------------------- | ------------------ |
| `uri` / `range`    | DocumentUri / Range | 实现所在文档与区间 |
| `result` 为 `null` | null                | 未找到实现         |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 18,
  "method": "textDocument/implementation",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "position": { "line": 4, "character": 5 }
  }
}
```

**响应示例**（返回多个实现的 `Location[]`）：

```json
{
  "jsonrpc": "2.0",
  "id": 18,
  "result": [
    {
      "uri": "file:///workspace/impl_a.go",
      "range": {
        "start": { "line": 10, "character": 5 },
        "end": { "line": 10, "character": 9 }
      }
    },
    {
      "uri": "file:///workspace/impl_b.go",
      "range": {
        "start": { "line": 8, "character": 5 },
        "end": { "line": 8, "character": 9 }
      }
    }
  ]
}
```

---

### 九、装饰类接口

#### 37. `textDocument/codeLens`（Request）

**功能**：获取文档中的 CodeLens（嵌在代码行上方的可点击信息，如"运行 \| 调试"、"N 处引用"）。

**请求参数（CodeLensParams）**：

| 字段               | 类型        | 说明         |
| ------------------ | ----------- | ------------ |
| `textDocument.uri` | DocumentUri | 目标文档 URI |

**响应参数（CodeLens[] \| null）**：

| 字段               | 类型    | 说明                                                     |
| ------------------ | ------- | -------------------------------------------------------- |
| `[].range`         | Range   | CodeLens 附着的代码区间                                  |
| `[].command`       | Command | 可选，点击执行的命令；缺省则需经 `codeLens/resolve` 补全 |
| `[].data`          | any     | 可选，resolve 时回传的私有数据                           |
| `result` 为 `null` | null    | 无 CodeLens                                              |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 19,
  "method": "textDocument/codeLens",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" }
  }
}
```

**响应示例**（`command` 缺省，待 `codeLens/resolve` 补全）：

```json
{
  "jsonrpc": "2.0",
  "id": 19,
  "result": [
    {
      "range": {
        "start": { "line": 2, "character": 0 },
        "end": { "line": 2, "character": 11 }
      },
      "data": { "uri": "file:///workspace/main.go", "id": 7 }
    }
  ]
}
```

#### 38. `codeLens/resolve`（Request）

**功能**：对单个 CodeLens 做详情懒加载，补全其 `command`，以提升首次返回性能。

**请求参数（CodeLens）**：传入 `codeLens` 返回的某一项。

**响应参数（CodeLens）**：返回补全了 `command` 的同一个 CodeLens。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 20,
  "method": "codeLens/resolve",
  "params": {
    "range": {
      "start": { "line": 2, "character": 0 },
      "end": { "line": 2, "character": 11 }
    },
    "data": { "uri": "file:///workspace/main.go", "id": 7 }
  }
}
```

**响应示例**（补全了 `command`）：

```json
{
  "jsonrpc": "2.0",
  "id": 20,
  "result": {
    "range": {
      "start": { "line": 2, "character": 0 },
      "end": { "line": 2, "character": 11 }
    },
    "command": {
      "title": "3 处引用",
      "command": "editor.action.showReferences",
      "arguments": ["file:///workspace/main.go", { "line": 2, "character": 5 }]
    }
  }
}
```

#### 39. `textDocument/documentLink`（Request）

**功能**：获取文档中可点击的链接（如注释里的 URL、import 路径），供编辑器渲染为超链接。

**请求参数（DocumentLinkParams）**：

| 字段               | 类型        | 说明         |
| ------------------ | ----------- | ------------ |
| `textDocument.uri` | DocumentUri | 目标文档 URI |

**响应参数（DocumentLink[] \| null）**：

| 字段               | 类型   | 说明                                       |
| ------------------ | ------ | ------------------------------------------ |
| `[].range`         | Range  | 链接覆盖的文本区间                         |
| `[].target`        | URI    | 可选，点击跳转的目标 URI；缺省则需 resolve |
| `[].tooltip`       | string | 可选，悬停提示                             |
| `[].data`          | any    | 可选，resolve 时回传的私有数据             |
| `result` 为 `null` | null   | 无链接                                     |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 21,
  "method": "textDocument/documentLink",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" }
  }
}
```

**响应示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 21,
  "result": [
    {
      "range": {
        "start": { "line": 0, "character": 8 },
        "end": { "line": 0, "character": 30 }
      },
      "target": "https://example.com/doc",
      "tooltip": "打开文档"
    }
  ]
}
```

#### 40. `documentLink/resolve`（Request）

**功能**：对单个文档链接做详情懒加载，补全其 `target`。

**请求参数（DocumentLink）**：传入 `documentLink` 返回的某一项。

**响应参数（DocumentLink）**：返回补全了 `target` 的同一个 DocumentLink。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 22,
  "method": "documentLink/resolve",
  "params": {
    "range": {
      "start": { "line": 0, "character": 8 },
      "end": { "line": 0, "character": 30 }
    },
    "data": { "uri": "file:///workspace/main.go", "linkId": 1 }
  }
}
```

**响应示例**（补全了 `target`）：

```json
{
  "jsonrpc": "2.0",
  "id": 22,
  "result": {
    "range": {
      "start": { "line": 0, "character": 8 },
      "end": { "line": 0, "character": 30 }
    },
    "target": "https://example.com/doc"
  }
}
```

#### 41. `textDocument/documentColor`（Request）

**功能**：获取文档中出现的颜色值引用（如 CSS 颜色），供编辑器显示颜色块。

**请求参数（DocumentColorParams）**：

| 字段               | 类型        | 说明         |
| ------------------ | ----------- | ------------ |
| `textDocument.uri` | DocumentUri | 目标文档 URI |

**响应参数（ColorInformation[]）**：

| 字段       | 类型  | 说明                                            |
| ---------- | ----- | ----------------------------------------------- |
| `[].range` | Range | 颜色值所在区间                                  |
| `[].color` | Color | 颜色，`{ red, green, blue, alpha }`，各分量 0~1 |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 23,
  "method": "textDocument/documentColor",
  "params": {
    "textDocument": { "uri": "file:///workspace/theme.css" }
  }
}
```

**响应示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 23,
  "result": [
    {
      "range": {
        "start": { "line": 4, "character": 9 },
        "end": { "line": 4, "character": 16 }
      },
      "color": { "red": 1, "green": 0, "blue": 0, "alpha": 1 }
    }
  ]
}
```

#### 42. `textDocument/colorPresentation`（Request）

**功能**：对一个具体颜色，请求其在该语言中的文本表示形式（如 `#RRGGBB`、`rgb(...)`），用于用户在取色器中改色后写回文档。

**请求参数（ColorPresentationParams）**：

| 字段               | 类型        | 说明                                 |
| ------------------ | ----------- | ------------------------------------ |
| `textDocument.uri` | DocumentUri | 目标文档 URI                         |
| `color`            | Color       | 颜色值 `{ red, green, blue, alpha }` |
| `range`            | Range       | 颜色当前所在区间                     |

**响应参数（ColorPresentation[]）**：

| 字段                     | 类型       | 说明                       |
| ------------------------ | ---------- | -------------------------- |
| `[].label`               | string     | 颜色文本表示，如 `#ff0000` |
| `[].textEdit`            | TextEdit   | 可选，应用该表示时的编辑   |
| `[].additionalTextEdits` | TextEdit[] | 可选，附加编辑             |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 24,
  "method": "textDocument/colorPresentation",
  "params": {
    "textDocument": { "uri": "file:///workspace/theme.css" },
    "color": { "red": 1, "green": 0, "blue": 0, "alpha": 1 },
    "range": {
      "start": { "line": 4, "character": 9 },
      "end": { "line": 4, "character": 16 }
    }
  }
}
```

**响应示例**（返回多种文本表示）：

```json
{
  "jsonrpc": "2.0",
  "id": 24,
  "result": [
    { "label": "#ff0000" },
    { "label": "rgb(255, 0, 0)" }
  ]
}
```

---

### 十、编辑辅助接口

#### 43. `textDocument/foldingRange`（Request）

**功能**：获取文档中可折叠的区间（函数体、注释块、import 区等）。

**请求参数（FoldingRangeParams）**：

| 字段               | 类型        | 说明         |
| ------------------ | ----------- | ------------ |
| `textDocument.uri` | DocumentUri | 目标文档 URI |

**响应参数（FoldingRange[] \| null）**：

| 字段                | 类型   | 说明                                       |
| ------------------- | ------ | ------------------------------------------ |
| `[].startLine`      | number | 折叠起始行，0-based                        |
| `[].endLine`        | number | 折叠结束行，0-based                        |
| `[].startCharacter` | number | 可选，起始字符                             |
| `[].endCharacter`   | number | 可选，结束字符                             |
| `[].kind`           | string | 可选，类型：`comment`、`imports`、`region` |
| `result` 为 `null`  | null   | 无折叠区间                                 |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 25,
  "method": "textDocument/foldingRange",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" }
  }
}
```

**响应示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 25,
  "result": [
    { "startLine": 0, "endLine": 2, "kind": "imports" },
    { "startLine": 4, "endLine": 8, "kind": "region" }
  ]
}
```

#### 44. `textDocument/selectionRange`（Request）

**功能**：获取若干位置的"智能选区"层级，用于扩大/缩小选择（如逐级从标识符→表达式→语句→块）。

**请求参数（SelectionRangeParams）**：

| 字段               | 类型        | 说明                  |
| ------------------ | ----------- | --------------------- |
| `textDocument.uri` | DocumentUri | 目标文档 URI          |
| `positions`        | Position[]  | 多个光标位置，0-based |

**响应参数（SelectionRange[] \| null）**：

| 字段               | 类型           | 说明                           |
| ------------------ | -------------- | ------------------------------ |
| `[].range`         | Range          | 当前层选区                     |
| `[].parent`        | SelectionRange | 可选，父层选区，递归构成层级链 |
| `result` 为 `null` | null           | 无选区信息                     |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 26,
  "method": "textDocument/selectionRange",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "positions": [
      { "line": 3, "character": 6 }
    ]
  }
}
```

**响应示例**（逐级父选区构成层级链）：

```json
{
  "jsonrpc": "2.0",
  "id": 26,
  "result": [
    {
      "range": {
        "start": { "line": 3, "character": 5 },
        "end": { "line": 3, "character": 12 }
      },
      "parent": {
        "range": {
          "start": { "line": 3, "character": 1 },
          "end": { "line": 3, "character": 22 }
        }
      }
    }
  ]
}
```

#### 45. `textDocument/prepareRename`（Request）

**功能**：在执行 `rename` 之前校验光标处是否可重命名，并返回可重命名的范围，避免在非法位置触发重命名。

**请求参数（PrepareRenameParams）**：

| 字段               | 类型        | 说明              |
| ------------------ | ----------- | ----------------- |
| `textDocument.uri` | DocumentUri | 目标文档 URI      |
| `position`         | Position    | 光标位置，0-based |

**响应参数（Range \| { range, placeholder } \| { defaultBehavior } \| null）**：

| 字段               | 类型    | 说明                                      |
| ------------------ | ------- | ----------------------------------------- |
| `range`            | Range   | 可重命名的文本区间                        |
| `placeholder`      | string  | 可选，重命名输入框的默认/占位文本         |
| `defaultBehavior`  | boolean | 可选，`true` 表示用客户端默认行为判定范围 |
| `result` 为 `null` | null    | 该位置不可重命名                          |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 27,
  "method": "textDocument/prepareRename",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "position": { "line": 10, "character": 8 }
  }
}
```

**响应示例**（返回可重命名范围与占位文本）：

```json
{
  "jsonrpc": "2.0",
  "id": 27,
  "result": {
    "range": {
      "start": { "line": 10, "character": 8 },
      "end": { "line": 10, "character": 15 }
    },
    "placeholder": "oldFuncName"
  }
}
```

#### 46. `textDocument/onTypeFormatting`（Request）

**功能**：在用户输入特定触发字符（如 `}`、`;`、换行）时即时格式化局部代码。

**请求参数（DocumentOnTypeFormattingParams）**：

| 字段               | 类型              | 说明                                       |
| ------------------ | ----------------- | ------------------------------------------ |
| `textDocument.uri` | DocumentUri       | 目标文档 URI                               |
| `position`         | Position          | 输入触发字符后的光标位置，0-based          |
| `ch`               | string            | 触发格式化的字符                           |
| `options`          | FormattingOptions | 格式化选项（`tabSize`、`insertSpaces` 等） |

**响应参数（TextEdit[] \| null）**：

| 字段                      | 类型           | 说明             |
| ------------------------- | -------------- | ---------------- |
| `[].range` / `[].newText` | Range / string | 需应用的文本编辑 |
| `result` 为 `null`        | null           | 无需格式化       |

**请求示例**（输入 `}` 触发）：

```json
{
  "jsonrpc": "2.0",
  "id": 28,
  "method": "textDocument/onTypeFormatting",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "position": { "line": 5, "character": 1 },
    "ch": "}",
    "options": {
      "tabSize": 4,
      "insertSpaces": false
    }
  }
}
```

**响应示例**（返回 `TextEdit[]`）：

```json
{
  "jsonrpc": "2.0",
  "id": 28,
  "result": [
    {
      "range": {
        "start": { "line": 5, "character": 0 },
        "end": { "line": 5, "character": 0 }
      },
      "newText": "\t"
    }
  ]
}
```

#### 47. `textDocument/willSaveWaitUntil`（Request）

**功能**：文档保存前，服务器返回一批需在保存前应用的文本编辑（如保存时自动整理 import）。是 `willSave` 通知的请求版本，会阻塞保存直到返回。

**请求参数（WillSaveTextDocumentParams）**：

| 字段               | 类型        | 说明                                               |
| ------------------ | ----------- | -------------------------------------------------- |
| `textDocument.uri` | DocumentUri | 即将保存的文档 URI                                 |
| `reason`           | number      | 保存原因：`1=Manual`、`2=AfterDelay`、`3=FocusOut` |

**响应参数（TextEdit[] \| null）**：

| 字段                      | 类型           | 说明             |
| ------------------------- | -------------- | ---------------- |
| `[].range` / `[].newText` | Range / string | 保存前应用的编辑 |
| `result` 为 `null`        | null           | 无需编辑         |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 29,
  "method": "textDocument/willSaveWaitUntil",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "reason": 1
  }
}
```

**响应示例**（保存前需应用的编辑，如整理 import）：

```json
{
  "jsonrpc": "2.0",
  "id": 29,
  "result": [
    {
      "range": {
        "start": { "line": 1, "character": 0 },
        "end": { "line": 1, "character": 0 }
      },
      "newText": "import \"fmt\"\n"
    }
  ]
}
```

---

### 十一、语义高亮接口

#### 48. `textDocument/semanticTokens/full`（Request）

**功能**：获取整篇文档的语义高亮标记（区分局部变量、参数、类型、命名空间等），用于比正则语法高亮更精确的着色。

**请求参数（SemanticTokensParams）**：

| 字段               | 类型        | 说明         |
| ------------------ | ----------- | ------------ |
| `textDocument.uri` | DocumentUri | 目标文档 URI |

**响应参数（SemanticTokens \| null）**：

| 字段               | 类型     | 说明                                                         |
| ------------------ | -------- | ------------------------------------------------------------ |
| `resultId`         | string   | 可选，本次结果 ID，供后续增量请求使用                        |
| `data`             | number[] | 按 5 个一组编码的整数数组：`[deltaLine, deltaStartChar, length, tokenType, tokenModifiers]`，各 token 相对前一个增量编码 |
| `result` 为 `null` | null     | 无语义标记                                                   |

> token 含义需结合 `initialize` 时服务器在 `semanticTokensProvider.legend` 中声明的 `tokenTypes` / `tokenModifiers` 索引表解码。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 30,
  "method": "textDocument/semanticTokens/full",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" }
  }
}
```

**响应示例**（`data` 按 5 个一组增量编码）：

```json
{
  "jsonrpc": "2.0",
  "id": 30,
  "result": {
    "resultId": "1",
    "data": [
      0, 0, 7, 8, 0,
      2, 5, 4, 12, 0
    ]
  }
}
```

#### 49. `textDocument/inlayHint`（Request）

**功能**：获取指定区间内的内联提示（inlay hints），如在调用处显示参数名、在变量后显示推断类型。

**请求参数（InlayHintParams）**：

| 字段               | 类型        | 说明               |
| ------------------ | ----------- | ------------------ |
| `textDocument.uri` | DocumentUri | 目标文档 URI       |
| `range`            | Range       | 请求提示的可见区间 |

**响应参数（InlayHint[] \| null）**：

| 字段                                 | 类型                           | 说明                          |
| ------------------------------------ | ------------------------------ | ----------------------------- |
| `[].position`                        | Position                       | 提示插入位置，0-based         |
| `[].label`                           | string \| InlayHintLabelPart[] | 提示文本或带交互的标签片段    |
| `[].kind`                            | number                         | 可选，`1=Type`、`2=Parameter` |
| `[].tooltip`                         | string \| MarkupContent        | 可选，悬停说明                |
| `[].paddingLeft` / `[].paddingRight` | boolean                        | 可选，是否两侧留白            |
| `[].textEdits`                       | TextEdit[]                     | 可选，接受该提示时应用的编辑  |
| `[].data`                            | any                            | 可选，`inlayHint/resolve` 时回传的私有数据 |
| `result` 为 `null`                   | null                           | 无提示                        |

`[].label` 字段可为**纯字符串**，或 **`InlayHintLabelPart[]`**（标签片段数组，支持每段独立交互，如点击参数名跳转到定义）。每个 `InlayHintLabelPart` 结构：

| 字段       | 类型                    | 说明                                                  |
| ---------- | ----------------------- | ----------------------------------------------------- |
| `value`    | string                  | 该片段的显示文本                                      |
| `tooltip`  | string \| MarkupContent | 可选，悬停该片段时的说明（MarkupContent 见接口 #11）  |
| `location` | Location                | 可选，点击该片段跳转的位置（`{ uri, range }`）        |
| `command`  | Command                 | 可选，点击该片段执行的命令（Command 见接口 #11）      |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 31,
  "method": "textDocument/inlayHint",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "range": {
      "start": { "line": 0, "character": 0 },
      "end": { "line": 20, "character": 0 }
    }
  }
}
```

**响应示例**（参数名与类型提示）：

```json
{
  "jsonrpc": "2.0",
  "id": 31,
  "result": [
    {
      "position": { "line": 5, "character": 12 },
      "label": "a:",
      "kind": 2,
      "paddingRight": true
    },
    {
      "position": { "line": 7, "character": 8 },
      "label": ": int",
      "kind": 1,
      "paddingLeft": true
    }
  ]
}
```

---

### 十二、调用层级接口

#### 50. `textDocument/prepareCallHierarchy`（Request）

**功能**：在光标处准备调用层级分析，返回作为层级根的层级项，供后续查询调用者/被调用者。

**请求参数（CallHierarchyPrepareParams）**：

| 字段               | 类型        | 说明              |
| ------------------ | ----------- | ----------------- |
| `textDocument.uri` | DocumentUri | 目标文档 URI      |
| `position`         | Position    | 光标位置，0-based |

**响应参数（CallHierarchyItem[] \| null）**：

| 字段                | 类型        | 说明                          |
| ------------------- | ----------- | ----------------------------- |
| `[].name`           | string      | 符号名称                      |
| `[].kind`           | number      | 符号类型枚举（同 SymbolKind） |
| `[].uri`            | DocumentUri | 符号所在文档                  |
| `[].range`          | Range       | 符号完整区间                  |
| `[].selectionRange` | Range       | 符号名称区间                  |
| `result` 为 `null`  | null        | 该位置无调用层级              |

> `[].kind` 为 SymbolKind 枚举（整数 `1~26`），完整取值见接口 #18 `documentSymbol`。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 32,
  "method": "textDocument/prepareCallHierarchy",
  "params": {
    "textDocument": { "uri": "file:///workspace/main.go" },
    "position": { "line": 10, "character": 8 }
  }
}
```

**响应示例**（返回作为层级根的 `CallHierarchyItem[]`）：

```json
{
  "jsonrpc": "2.0",
  "id": 32,
  "result": [
    {
      "name": "add",
      "kind": 12,
      "uri": "file:///workspace/util.go",
      "range": {
        "start": { "line": 3, "character": 0 },
        "end": { "line": 5, "character": 1 }
      },
      "selectionRange": {
        "start": { "line": 3, "character": 5 },
        "end": { "line": 3, "character": 8 }
      }
    }
  ]
}
```

#### 51. `callHierarchy/incomingCalls`（Request）

**功能**：查询某层级项的**调用者**（谁调用了它）。

**请求参数（CallHierarchyIncomingCallsParams）**：

| 字段   | 类型              | 说明                                |
| ------ | ----------------- | ----------------------------------- |
| `item` | CallHierarchyItem | `prepareCallHierarchy` 返回的层级项 |

**响应参数（CallHierarchyIncomingCall[] \| null）**：

| 字段               | 类型              | 说明                         |
| ------------------ | ----------------- | ---------------------------- |
| `[].from`          | CallHierarchyItem | 调用方符号                   |
| `[].fromRanges`    | Range[]           | 调用方内部发生调用的具体区间 |
| `result` 为 `null` | null              | 无调用者                     |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 33,
  "method": "callHierarchy/incomingCalls",
  "params": {
    "item": {
      "name": "main",
      "kind": 12,
      "uri": "file:///workspace/main.go",
      "range": {
        "start": { "line": 2, "character": 0 },
        "end": { "line": 6, "character": 1 }
      },
      "selectionRange": {
        "start": { "line": 2, "character": 5 },
        "end": { "line": 2, "character": 9 }
      }
    }
  }
}
```

**响应示例**（谁调用了它）：

```json
{
  "jsonrpc": "2.0",
  "id": 33,
  "result": [
    {
      "from": {
        "name": "run",
        "kind": 12,
        "uri": "file:///workspace/app.go",
        "range": {
          "start": { "line": 8, "character": 0 },
          "end": { "line": 12, "character": 1 }
        },
        "selectionRange": {
          "start": { "line": 8, "character": 5 },
          "end": { "line": 8, "character": 8 }
        }
      },
      "fromRanges": [
        {
          "start": { "line": 10, "character": 4 },
          "end": { "line": 10, "character": 8 }
        }
      ]
    }
  ]
}
```

#### 52. `callHierarchy/outgoingCalls`（Request）

**功能**：查询某层级项的**被调用者**（它调用了谁）。

**请求参数（CallHierarchyOutgoingCallsParams）**：

| 字段   | 类型              | 说明                                |
| ------ | ----------------- | ----------------------------------- |
| `item` | CallHierarchyItem | `prepareCallHierarchy` 返回的层级项 |

**响应参数（CallHierarchyOutgoingCall[] \| null）**：

| 字段               | 类型              | 说明                       |
| ------------------ | ----------------- | -------------------------- |
| `[].to`            | CallHierarchyItem | 被调用的符号               |
| `[].fromRanges`    | Range[]           | 在当前项内发生该调用的区间 |
| `result` 为 `null` | null              | 无被调用者                 |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 34,
  "method": "callHierarchy/outgoingCalls",
  "params": {
    "item": {
      "name": "main",
      "kind": 12,
      "uri": "file:///workspace/main.go",
      "range": {
        "start": { "line": 2, "character": 0 },
        "end": { "line": 6, "character": 1 }
      },
      "selectionRange": {
        "start": { "line": 2, "character": 5 },
        "end": { "line": 2, "character": 9 }
      }
    }
  }
}
```

**响应示例**（它调用了谁）：

```json
{
  "jsonrpc": "2.0",
  "id": 34,
  "result": [
    {
      "to": {
        "name": "add",
        "kind": 12,
        "uri": "file:///workspace/util.go",
        "range": {
          "start": { "line": 3, "character": 0 },
          "end": { "line": 5, "character": 1 }
        },
        "selectionRange": {
          "start": { "line": 3, "character": 5 },
          "end": { "line": 3, "character": 8 }
        }
      },
      "fromRanges": [
        {
          "start": { "line": 4, "character": 8 },
          "end": { "line": 4, "character": 11 }
        }
      ]
    }
  ]
}
```

---

### 十三、工作区与客户端管理接口

#### 53. `workspace/executeCommand`（Request）

**功能**：请求服务器执行一个其在 `initialize` 时声明的命令（如"整理 import"、"生成代码"）。命令副作用通常通过 `workspace/applyEdit` 反向应用编辑。

**请求参数（ExecuteCommandParams）**：

| 字段        | 类型   | 说明                                                         |
| ----------- | ------ | ------------------------------------------------------------ |
| `command`   | string | 命令标识符，须在服务器 `executeCommandProvider.commands` 中声明 |
| `arguments` | any[]  | 可选，命令参数数组                                           |

**响应参数（any \| null）**：

| 字段     | 类型 | 说明                                              |
| -------- | ---- | ------------------------------------------------- |
| `result` | any  | 命令的返回值，结构由命令自定义；无返回则为 `null` |

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "id": 35,
  "method": "workspace/executeCommand",
  "params": {
    "command": "go.organizeImports",
    "arguments": ["file:///workspace/main.go"]
  }
}
```

**响应示例**（命令无返回值时为 `null`；副作用通常通过 `workspace/applyEdit` 反向应用）：

```json
{
  "jsonrpc": "2.0",
  "id": 35,
  "result": null
}
```

#### 54. `workspace/applyEdit`（Request，S→C）

**功能**：服务器请求客户端将一批编辑应用到工作区（如重命名、代码动作的结果落地）。

**请求参数（ApplyWorkspaceEditParams）**：

| 字段    | 类型          | 说明                                            |
| ------- | ------------- | ----------------------------------------------- |
| `label` | string        | 可选，本次编辑的标签（用于撤销栈显示）          |
| `edit`  | WorkspaceEdit | 要应用的编辑，含 `changes` 或 `documentChanges` |

**响应参数（ApplyWorkspaceEditResult）**：

| 字段            | 类型    | 说明                 |
| --------------- | ------- | -------------------- |
| `applied`       | boolean | 是否成功应用         |
| `failureReason` | string  | 可选，失败原因       |
| `failedChange`  | number  | 可选，失败的变更索引 |

**请求示例**（由服务器发往客户端）：

```json
{
  "jsonrpc": "2.0",
  "id": 103,
  "method": "workspace/applyEdit",
  "params": {
    "label": "整理 import",
    "edit": {
      "changes": {
        "file:///workspace/main.go": [
          {
            "range": {
              "start": { "line": 1, "character": 0 },
              "end": { "line": 1, "character": 0 }
            },
            "newText": "import \"fmt\"\n"
          }
        ]
      }
    }
  }
}
```

**响应示例**（由客户端返回是否应用成功）：

```json
{
  "jsonrpc": "2.0",
  "id": 103,
  "result": { "applied": true }
}
```

#### 55. `workspace/configuration`（Request，S→C）

**功能**：服务器主动向客户端拉取配置项（按作用域和 section 查询）。

**请求参数（ConfigurationParams）**：

| 字段               | 类型                | 说明                                |
| ------------------ | ------------------- | ----------------------------------- |
| `items`            | ConfigurationItem[] | 查询项数组                          |
| `items[].scopeUri` | DocumentUri         | 可选，配置作用域 URI                |
| `items[].section`  | string              | 可选，配置 section 名，如 `go.lint` |

**响应参数（any[]）**：

| 字段     | 类型  | 说明                            |
| -------- | ----- | ------------------------------- |
| `result` | any[] | 与 `items` 一一对应的配置值数组 |

**请求示例**（由服务器发往客户端）：

```json
{
  "jsonrpc": "2.0",
  "id": 104,
  "method": "workspace/configuration",
  "params": {
    "items": [
      { "scopeUri": "file:///workspace/main.go", "section": "go.lint" },
      { "section": "go.format" }
    ]
  }
}
```

**响应示例**（由客户端返回与 `items` 一一对应的配置值）：

```json
{
  "jsonrpc": "2.0",
  "id": 104,
  "result": [
    { "enable": true, "tool": "golangci-lint" },
    "gofmt"
  ]
}
```

#### 56. `workspace/didChangeWatchedFiles`（Notification）

**功能**：客户端通知服务器，工作区内被监视的文件在磁盘上发生了创建/修改/删除（区别于编辑器内编辑）。

**请求参数（DidChangeWatchedFilesParams）**：

| 字段             | 类型        | 说明                                            |
| ---------------- | ----------- | ----------------------------------------------- |
| `changes`        | FileEvent[] | 文件变化数组                                    |
| `changes[].uri`  | DocumentUri | 发生变化的文件 URI                              |
| `changes[].type` | number      | 变化类型：`1=Created`、`2=Changed`、`3=Deleted` |

**响应参数**：**无响应**（通知类消息）。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "method": "workspace/didChangeWatchedFiles",
  "params": {
    "changes": [
      { "uri": "file:///workspace/go.mod", "type": 2 },
      { "uri": "file:///workspace/old.go", "type": 3 }
    ]
  }
}
```

**调用效果**：该通知无返回。服务器据磁盘变化更新内部状态（如 `go.mod` 变化触发依赖重新解析、文件删除清除其符号），随后通常对受影响的已打开文档重新分析并**主动回送** `textDocument/publishDiagnostics`。例如 `go.mod` 变更导致某文档的导入错误被解决：

```json
{
  "jsonrpc": "2.0",
  "method": "textDocument/publishDiagnostics",
  "params": {
    "uri": "file:///workspace/main.go",
    "diagnostics": []
  }
}
```

#### 57. `workspace/didChangeWorkspaceFolders`（Notification）

**功能**：客户端通知服务器，多根工作区的文件夹被添加或移除。

**请求参数（DidChangeWorkspaceFoldersParams）**：

| 字段            | 类型              | 说明                                 |
| --------------- | ----------------- | ------------------------------------ |
| `event.added`   | WorkspaceFolder[] | 新增的文件夹，每项为 `{ uri, name }` |
| `event.removed` | WorkspaceFolder[] | 移除的文件夹                         |

**响应参数**：**无响应**（通知类消息）。

**请求示例**：

```json
{
  "jsonrpc": "2.0",
  "method": "workspace/didChangeWorkspaceFolders",
  "params": {
    "event": {
      "added": [
        { "uri": "file:///workspace/pkg-a", "name": "pkg-a" }
      ],
      "removed": [
        { "uri": "file:///workspace/pkg-b", "name": "pkg-b" }
      ]
    }
  }
}
```

**调用效果**：该通知无返回。服务器将新增文件夹纳入分析范围、移除被删文件夹的索引与诊断。常见后续反应是对新增文件夹启动索引并通过 `$/progress` 上报进度，或对受影响文档刷新 `textDocument/publishDiagnostics`。例如开始索引新加入的文件夹：

```json
{
  "jsonrpc": "2.0",
  "method": "$/progress",
  "params": {
    "token": "indexing-pkg-a",
    "value": { "kind": "begin", "title": "正在索引 pkg-a", "percentage": 0 }
  }
}
```

#### 58. `client/registerCapability`（Request，S→C）

**功能**：服务器在运行期**动态注册**能力（如初始化后才决定支持某语言的格式化），无需在 `initialize` 时静态声明。

**请求参数（RegistrationParams）**：

| 字段                              | 类型           | 说明                                                |
| --------------------------------- | -------------- | --------------------------------------------------- |
| `registrations`                   | Registration[] | 注册项数组                                          |
| `registrations[].id`              | string         | 注册 ID，用于后续注销                               |
| `registrations[].method`          | string         | 要注册的方法名，如 `textDocument/willSaveWaitUntil` |
| `registrations[].registerOptions` | any            | 可选，该能力的注册选项                              |

**响应参数**：

| 字段     | 类型 | 说明                          |
| -------- | ---- | ----------------------------- |
| `result` | null | 成功返回 `null`；失败返回错误 |

**请求示例**（由服务器发往客户端）：

```json
{
  "jsonrpc": "2.0",
  "id": 105,
  "method": "client/registerCapability",
  "params": {
    "registrations": [
      {
        "id": "reg-willSaveWaitUntil-1",
        "method": "textDocument/willSaveWaitUntil",
        "registerOptions": {
          "documentSelector": [{ "language": "go" }]
        }
      }
    ]
  }
}
```

**响应示例**（由客户端确认注册成功）：

```json
{
  "jsonrpc": "2.0",
  "id": 105,
  "result": null
}
```

#### 59. `client/unregisterCapability`（Request，S→C）

**功能**：服务器注销此前动态注册的能力。

**请求参数（UnregistrationParams）**：

| 字段                        | 类型             | 说明                                                  |
| --------------------------- | ---------------- | ----------------------------------------------------- |
| `unregisterations`          | Unregistration[] | 注销项数组（注意规范中此字段名为 `unregisterations`） |
| `unregisterations[].id`     | string           | 之前注册时使用的 ID                                   |
| `unregisterations[].method` | string           | 对应的方法名                                          |

**响应参数**：

| 字段     | 类型 | 说明                          |
| -------- | ---- | ----------------------------- |
| `result` | null | 成功返回 `null`；失败返回错误 |

**请求示例**（由服务器发往客户端）：

```json
{
  "jsonrpc": "2.0",
  "id": 106,
  "method": "client/unregisterCapability",
  "params": {
    "unregisterations": [
      {
        "id": "reg-willSaveWaitUntil-1",
        "method": "textDocument/willSaveWaitUntil"
      }
    ]
  }
}
```

**响应示例**（由客户端确认注销成功）：

```json
{
  "jsonrpc": "2.0",
  "id": 106,
  "result": null
}
```

---

### 扩展接口分类速查

| 分类              | 接口                                                         | 数量   |
| ----------------- | ------------------------------------------------------------ | ------ |
| 窗口消息（S→C）   | `window/showMessage`、`window/showMessageRequest`、`window/logMessage`、`window/showDocument`、`telemetry/event` | 5      |
| 基础协议          | `$/cancelRequest`、`$/progress`、`window/workDoneProgress/create`、`window/workDoneProgress/cancel` | 4      |
| 扩展导航          | `declaration`、`typeDefinition`、`implementation`            | 3      |
| 装饰类            | `codeLens` + `codeLens/resolve`、`documentLink` + `documentLink/resolve`、`documentColor`、`colorPresentation` | 6      |
| 编辑辅助          | `foldingRange`、`selectionRange`、`prepareRename`、`onTypeFormatting`、`willSaveWaitUntil` | 5      |
| 语义高亮          | `semanticTokens/full`、`inlayHint`                           | 2      |
| 调用层级          | `prepareCallHierarchy`、`incomingCalls`、`outgoingCalls`     | 3      |
| 工作区/客户端管理 | `executeCommand`、`applyEdit`、`configuration`、`didChangeWatchedFiles`、`didChangeWorkspaceFolders`、`registerCapability`、`unregisterCapability` | 7      |
| **扩展合计**      |                                                              | **35** |

> 备注：核心 24 + 扩展 35 = 59 个接口，已覆盖 LSP 规范中绝大多数日常使用的方法。规范仍在演进，更多增量/范围变体（如 `semanticTokens/full/delta`、`semanticTokens/range`、`typeHierarchy/*`、`moniker`、`linkedEditingRange`、`inlineValue`、`diagnostic`（pull 模式）等）可按相同模式扩展。`window/*` 与 `client/*`、`workspace/applyEdit`、`workspace/configuration` 是典型的**服务器 → 客户端**消息，实现完整 LSP 客户端时需要在客户端侧处理它们。

---

## 附录

### 附录 A：请求 `id` 的取值约束

LSP 基于 JSON-RPC 2.0，每个 **Request** 必须带 `id`，**Notification** 不带 `id`。`id` 用于把响应对应回它所应答的请求。

**① 类型约束**：`id` 只能是以下两种之一（JSON-RPC 2.0 规定）：

| 类型   | 说明                                                                 |
| ------ | -------------------------------------------------------------------- |
| 整数   | 最常见。规范要求数字**不应包含小数部分**（即应为整数，不要用 `1.5`） |
| 字符串 | 允许，如 `"req-42"`。数字与字符串均可，取决于实现                    |

> 规范虽允许 `null`，但**不推荐**使用 `null` 作为 `id`（易与"解析错误时返回的 `id: null`"混淆）。

**② 数值范围**：协议层面**没有上下限**。但用整数时受 **JSON 双精度浮点**精度约束——可精确表示的最大整数为 **2⁵³ − 1 = 9007199254740991**（即 JavaScript 的 `Number.MAX_SAFE_INTEGER`）。超过此值自增会丢精度，可能导致两个请求拿到相同 `id` 而响应错配。实际 LSP 会话一次发出的请求量远达不到该量级，正常无需担心。

**③ 唯一性约束**：`id` 在**同一发起方、同一连接、当前未完成的请求集合内**必须唯一。已收到响应的 `id` 理论上可复用，但实践中通常一路自增、不复用。

**④ 自增与方向**：

- `id` 由**发起请求的一方**维护一个自增计数器，**响应方原样带回**，不产生新值。
- 客户端发起的请求（C→S）与服务器主动发起的请求（S→C）是**两条独立的请求流**，各自维护计数器，**号段重叠互不影响**——靠"方向 + id"共同定位响应。
- 因此「客户端发 1 → 服务器回 1；客户端发 2 → 服务器回 2」才是正确流程，而非「客户端发 1 → 服务器回 2」。

> 本文示例中的号段划分（客户端核心 1~15、扩展 16~35、服务器主动 100~106）纯属**阅读时区分消息方向的约定**，非协议要求；真实实现里两边各自从 0 或 1 起自增即可。