> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# null

# Claude Code 文档导航图

这是所有 Claude Code 文档页面的综合导航图，包含其标题，旨在方便 LLM 轻松导航。

> **注意：** 此文件由 GitHub Actions 自动生成。请勿手动编辑。
> 最后更新时间：2026-02-05 18:34:06 UTC

## 文档结构

此导航图使用分层结构：

* **##** 标记文档组（例如"入门指南"）
* **###** 标记单个文档页面
* **嵌套项目符号** 显示每个页面内的标题结构
* 每个页面标题都链接到完整文档

## 入门指南

### [overview](https://code.claude.com/docs/zh-CN/overview.md)

* 30 秒快速开始
* Claude Code 为您做什么
* 开发者为什么喜欢 Claude Code
* 在任何地方使用 Claude Code
* 后续步骤
* 其他资源

### [quickstart](https://code.claude.com/docs/zh-CN/quickstart.md)

* 开始之前
* 步骤 1：安装 Claude Code
* 步骤 2：登录您的账户
* 步骤 3：开始您的第一个会话
* 步骤 4：提出您的第一个问题
* 步骤 5：进行您的第一次代码更改
* 步骤 6：在 Claude Code 中使用 Git
* 步骤 7：修复错误或添加功能
* 步骤 8：尝试其他常见工作流
* 基本命令
* 初学者专业提示
* 接下来是什么？
* 获取帮助

### [changelog](https://code.claude.com/docs/zh-CN/changelog.md)

* （未找到标题）

## 核心概念

### [how-claude-code-works](https://code.claude.com/docs/zh-CN/how-claude-code-works.md)

* 代理循环
  * 模型
  * 工具
* Claude 可以访问什么
* 使用会话
  * 跨分支工作
  * 恢复或分叉会话
  * context window
    * 当 context 填满时
    * 使用 skills 和 subagents 管理 context
* 使用 checkpoints 和权限保持安全
  * 使用 checkpoints 撤销更改
  * 控制 Claude 可以做什么
* 有效地使用 Claude Code
  * 向 Claude Code 寻求帮助
  * 这是一次对话
    * 中断和引导
  * 提前具体说明
  * 给 Claude 一些东西来验证
  * 在实施前探索
  * 委派，不要指示
* 接下来是什么

### [features-overview](https://code.claude.com/docs/zh-CN/features-overview.md)

* 概述
* 将功能与您的目标相匹配
  * 比较类似功能
  * 了解功能如何分层
  * 组合功能
* 了解 context 成本
  * 按功能的 context 成本
  * 了解功能如何加载
* 了解更多

### [common-workflows](https://code.claude.com/docs/zh-CN/common-workflows.md)

* 理解新的代码库
  * 获取快速代码库概览
  * 查找相关代码
* 高效修复错误
* 重构代码
* 使用专门的 subagents
* 使用 Plan Mode 进行安全代码分析
  * 何时使用 Plan Mode
  * 如何使用 Plan Mode
  * 示例：规划复杂重构
  * 将 Plan Mode 配置为默认值
* 使用测试
* 创建拉取请求
* 处理文档
* 使用图像
* 引用文件和目录
* 使用扩展思考（Thinking Mode）
  * 配置 Thinking Mode
  * 扩展思考如何工作
* 恢复以前的对话
  * 命名您的会话
  * 使用会话选择器
* 使用 Git worktrees 运行并行 Claude Code 会话
* 将 Claude 用作 unix 风格的实用程序
  * 将 Claude 添加到您的验证流程
  * 管道输入，管道输出
  * 控制输出格式
* 询问 Claude 其功能
  * 示例问题
* 后续步骤

### [best-practices](https://code.claude.com/docs/zh-CN/best-practices.md)

* 给 Claude 一种验证其工作的方式
* 先探索，然后规划，然后编码
* 在提示中提供具体的 context
  * 提供丰富的内容
* 配置您的环境
  * 编写有效的 CLAUDE.md
  * 配置权限
  * 使用 CLI 工具
  * 连接 MCP servers
  * 设置 hooks
  * 创建 skills
  * 创建自定义 subagents
  * 安装 plugins
* 有效沟通
  * 提出代码库问题
  * 让 Claude 采访您
* 管理您的会话
  * 尽早且经常纠正方向
  * 积极管理 context
  * 使用 subagents 进行调查
  * 使用 checkpoints 回退
  * 恢复对话
* 自动化和扩展
  * 运行 headless 模式
  * 运行多个 Claude 会话
  * 跨文件分散
  * 安全自主模式
* 避免常见失败模式
* 培养您的直觉
* 相关资源

## 终端外

### [claude-code-on-the-web](https://code.claude.com/docs/zh-CN/claude-code-on-the-web.md)

* 什么是网络上的 Claude Code？
* 谁可以使用网络上的 Claude Code？
* 入门
* 它如何工作
* 使用 diff 视图查看更改
* 在网络和终端之间移动任务
  * 从终端到网络
    * 后台任务提示
  * 从网络到终端
    * 传送的要求
  * 共享会话
    * 从企业或团队账户共享
    * 从 Max 或 Pro 账户共享
* 云环境
  * 默认镜像
    * 检查可用工具
    * 特定于语言的设置
    * 数据库
  * 环境配置
  * 依赖管理
    * 本地与远程执行
    * 持久化环境变量
* 网络访问和安全
  * 网络策略
    * GitHub 代理
    * 安全代理
  * 访问级别
  * 默认允许的域
    * Anthropic 服务
    * 版本控制
    * 容器注册表
    * 云平台
    * 包管理器 - JavaScript/Node
    * 包管理器 - Python
    * 包管理器 - Ruby
    * 包管理器 - Rust
    * 包管理器 - Go
    * 包管理器 - JVM
    * 包管理器 - 其他语言
    * Linux 发行版
    * 开发工具和平台
    * 云服务和监控
    * 内容交付和镜像
    * 架构和配置
    * Model Context Protocol
  * 自定义网络访问的安全最佳实践
* 安全和隔离
* 定价和速率限制
* 限制
* 最佳实践
* 相关资源

### [desktop](https://code.claude.com/docs/zh-CN/desktop.md)

* 安装和设置
* 入门
* 您可以做什么
  * 选择权限模式
  * 与会话并行工作
  * 远程运行长时间运行的任务
  * 使用 diff 视图查看更改
* 扩展 Claude Code
  * 连接外部工具
  * 创建自定义 skills
  * 使用 hooks 自动化工作流
* 环境配置
* Desktop 与 CLI 的关系
  * CLI 标志等效项
  * 共享配置
  * 有什么不同
* 故障排除
  * 检查您的版本
  * 在 CLI 中打开时"分支尚不存在"
  * "无法加载会话"错误
  * 应用无法退出
  * Windows 安装问题
  * 会话找不到已安装的工具
  * MCP servers 不工作（Windows）
  * Git LFS 错误
* 企业配置
* 相关资源

### [chrome](https://code.claude.com/docs/zh-CN/chrome.md)

* 功能
* 先决条件
* 在 CLI 中开始
  * 默认启用 Chrome
  * 管理网站权限
* 示例工作流
  * 测试本地网络应用
  * 使用控制台日志调试
  * 自动化表单填充
  * 在 Google Docs 中起草内容
  * 从网页提取数据
  * 运行多站点工作流
  * 录制演示 GIF
* 故障排除
  * 未检测到扩展
  * 浏览器无响应
  * 长会话期间连接断开
  * Windows 特定问题
  * 常见错误消息
* 另请参阅

### [vs-code](https://code.claude.com/docs/zh-CN/vs-code.md)

* 先决条件
* 安装扩展
* 入门
* 使用提示框
  * 引用文件和文件夹
  * 恢复过去的对话
  * 从 Claude.ai 恢复远程会话
* 自定义您的工作流
  * 选择 Claude 的位置
  * 运行多个对话
  * 切换到终端模式
* 管理 plugins
  * 安装 plugins
  * 管理市场
* 使用 Chrome 自动化浏览器任务
* VS Code 命令和快捷键
* 配置设置
  * 扩展设置
* VS Code 扩展与 Claude Code CLI
  * 使用 checkpoints 回退
  * 在 VS Code 中运行 CLI
  * 在扩展和 CLI 之间切换
  * 在提示中包含终端输出
  * 监控后台进程
  * 使用 MCP 连接到外部工具
* 使用 git
  * 创建提交和拉取请求
  * 使用 git worktrees 进行并行任务
* 使用第三方提供商
* 安全和隐私
* 修复常见问题
  * 扩展无法安装
  * Spark 图标不可见
  * Claude Code 从不响应
* 卸载扩展
* 后续步骤

### [jetbrains](https://code.claude.com/docs/zh-CN/jetbrains.md)

* 支持的 IDE
* 功能
* 安装
  * 市场安装
* 用法
  * 从您的 IDE
  * 从外部终端
* 配置
  * Claude Code 设置
  * Plugin 设置
    * 常规设置
    * ESC 键配置
* 特殊配置
  * 远程开发
  * WSL 配置
* 故障排除
  * Plugin 不工作
  * IDE 未检测到
  * 命令未找到
* 安全考虑

### [github-actions](https://code.claude.com/docs/zh-CN/github-actions.md)

* 为什么使用 Claude Code GitHub Actions？
* Claude 可以做什么？
  * Claude Code Action
* 设置
* 快速设置
* 手动设置
* 从 Beta 升级
  * 基本更改
  * 破坏性更改参考
  * 前后示例
* 示例用例
  * 基本工作流
  * 使用 skills
  * 使用提示的自定义自动化
  * 常见用例
* 最佳实践
  * CLAUDE.md 配置
  * 安全考虑
  * 优化性能
  * CI 成本
* 配置示例
* 与 AWS Bedrock 和 Google Vertex AI 一起使用
  * 先决条件
    * 对于 Google Cloud Vertex AI：
    * 对于 AWS Bedrock：
* 故障排除
  * Claude 不响应 @claude 命令
  * CI 不在 Claude 的提交上运行
  * 身份验证错误
* 高级配置
  * Action 参数
    * 传递 CLI 参数
  * 替代集成方法
  * 自定义 Claude 的行为

### [gitlab-ci-cd](https://code.claude.com/docs/zh-CN/gitlab-ci-cd.md)

* 为什么在 GitLab 中使用 Claude Code？
* 它如何工作
* Claude 可以做什么？
* 设置
  * 快速设置
  * 手动设置（推荐用于生产）
* 示例用例
  * 将问题转换为 MR
  * 获取实施帮助
  * 快速修复错误
* 与 AWS Bedrock 和 Google Vertex AI 一起使用
* 配置示例
  * 基本 .gitlab-ci.yml（Claude API）
  * AWS Bedrock 作业示例（OIDC）
  * Google Vertex AI 作业示例（工作负载身份联合）
* 最佳实践
  * CLAUDE.md 配置
  * 安全考虑
  * 优化性能
  * CI 成本
* 安全和治理
* 故障排除
  * Claude 不响应 @claude 命令
  * 作业无法写入评论或打开 MR
  * 身份验证错误
* 高级配置
  * 常见参数和变量
  * 自定义 Claude 的行为

### [slack](https://code.claude.com/docs/zh-CN/slack.md)

* 用例
* 先决条件
* 在 Slack 中设置 Claude Code
* 它如何工作
  * 自动检测
  * Context 收集
  * 会话流
* 用户界面元素
  * 应用主页
  * 消息操作
  * 存储库选择
* 访问和权限
  * 用户级访问
  * 工作区级访问
  * 基于频道的访问控制
* 什么可以在哪里访问
* 最佳实践
  * 编写有效的请求
  * 何时使用 Slack 与网络
* 故障排除
  * 会话未启动
  * 存储库未显示
  * 选择了错误的存储库
  * 身份验证错误
  * 会话过期
* 当前限制
* 相关资源

## 使用 Claude Code 构建

### [sub-agents](https://code.claude.com/docs/zh-CN/sub-agents.md)

* 内置 subagents
* 快速入门：创建您的第一个 subagent
* 配置 subagents
  * 使用 /agents 命令
  * 选择 subagent 范围
  * 编写 subagent 文件
    * 支持的 frontmatter 字段
  * 选择模型
  * 控制 subagent 功能
    * 可用工具
    * 权限模式
    * 将 skills 预加载到 subagents
    * 启用持久内存
      * 持久内存提示
    * 使用 hooks 的条件规则
    * 禁用特定 subagents
  * 为 subagents 定义 hooks
    * subagent frontmatter 中的 hooks
    * subagent 事件的项目级 hooks
* 使用 subagents
  * 了解自动委派
  * 在前台或后台运行 subagents
  * 常见模式
    * 隔离高容量操作
    * 运行并行研究
    * 链接 subagents
  * 在 subagents 和主对话之间选择
  * 管理 subagent context
    * 恢复 subagents
    * 自动压缩
* 示例 subagents
  * 代码审查员
  * 调试器
  * 数据科学家
  * 数据库查询验证器
* 后续步骤

### [agent-teams](https://code.claude.com/docs/zh-CN/agent-teams.md)

* 何时使用 agent teams
  * 与 subagents 比较
* 启用 agent teams
* 开始您的第一个 agent team
* 控制您的 agent team
  * 选择显示模式
  * 指定队友和模型
  * 要求队友批准计划
  * 使用委派模式
  * 直接与队友交谈
  * 分配和认领任务
  * 关闭队友
  * 清理团队
* agent teams 如何工作
  * Claude 如何启动 agent teams
  * 架构
  * 权限
  * Context 和通信
  * Token 使用
* 用例示例
  * 运行并行代码审查
  * 使用竞争假设进行调查
* 最佳实践
  * 给队友足够的 context
  * 适当调整任务大小
  * 等待队友完成
  * 从研究和审查开始
  * 避免文件冲突
  * 监控和引导
* 故障排除
  * 队友未出现
  * 过多权限提示
  * 队友在错误时停止
  * Lead 在工作完成前关闭
  * 孤立的 tmux 会话
* 限制
* 后续步骤

### [plugins](https://code.claude.com/docs/zh-CN/plugins.md)

* 何时使用 plugins 与独立配置
* 快速入门
  * 先决条件
  * 创建您的第一个 plugin
* Hello Command
  * Plugin 结构概述
  * 开发更复杂的 plugins
    * 将 Skills 添加到您的 plugin
    * 将 LSP servers 添加到您的 plugin
    * 组织复杂的 plugins
    * 在本地测试您的 plugins
    * 调试 plugin 问题
    * 共享您的 plugins
  * 将现有配置转换为 plugins
    * 迁移步骤
    * 迁移时的变化
  * 后续步骤
    * 对于 plugin 用户
    * 对于 plugin 开发者

### [discover-plugins](https://code.claude.com/docs/zh-CN/discover-plugins.md)

* 市场如何工作
* 官方 Anthropic 市场
  * 代码智能
    * Claude 从代码智能 plugins 获得什么
  * 外部集成
  * 开发工作流
  * 输出样式
* 尝试：添加演示市场
* 添加市场
  * 从 GitHub 添加
  * 从其他 Git 主机添加
  * 从本地路径添加
  * 从远程 URL 添加
* 安装 plugins
* 管理已安装的 plugins
* 管理市场
  * 使用交互式界面
  * 使用 CLI 命令
  * 配置自动更新
* 配置团队市场
* 故障排除
  * /plugin 命令未识别
  * 常见问题
  * 代码智能问题
* 后续步骤

### [skills](https://code.claude.com/docs/zh-CN/skills.md)

* 入门
  * 创建您的第一个 skill
  * skills 的位置
    * 从嵌套目录自动发现
* 配置 skills
  * skill 内容的类型
  * Frontmatter 参考
    * 可用的字符串替换
  * 添加支持文件
  * 控制谁调用 skill
  * 限制工具访问
  * 将参数传递给 skills
* 高级模式
  * 注入动态 context
  * 在 subagent 中运行 skills
    * 示例：使用 Explore agent 的研究 skill
  * 限制 Claude 的 skill 访问
* 共享 skills
  * 生成视觉输出
* 故障排除
  * Skill 未触发
  * Skill 触发过于频繁
  * Claude 看不到我的所有 skills
* 相关资源

### [output-styles](https://code.claude.com/docs/zh-CN/output-styles.md)

* 内置输出样式
* 输出样式如何工作
* 更改您的输出样式
* 创建自定义输出样式
  * Frontmatter
* 与相关功能的比较
  * 输出样式与 CLAUDE.md 与 --append-system-prompt
  * 输出样式与 [Agents](/zh-CN/sub-agents)
  * 输出样式与 [Skills](/zh-CN/skills)

### [hooks-guide](https://code.claude.com/docs/zh-CN/hooks-guide.md)

* 设置您的第一个 hook
* 您可以自动化什么
  * 在 Claude 需要输入时获得通知
  * 编辑后自动格式化代码
  * 阻止对受保护文件的编辑
  * 压缩后重新注入 context
* hooks 如何工作
  * 读取输入并返回输出
    * Hook 输入
    * Hook 输出
    * 结构化 JSON 输出
  * 使用匹配器过滤 hooks
  * 配置 hook 位置
* 基于提示的 hooks
* 基于代理的 hooks
* 限制和故障排除
  * 限制
  * Hook 未触发
  * Hook 输出中的错误
  * `/hooks` 显示未配置 hooks
  * Stop hook 永远运行
  * JSON 验证失败
  * 调试技术
* 了解更多

### [headless](https://code.claude.com/docs/zh-CN/headless.md)

* 基本用法
* 示例
  * 获取结构化输出
  * 流式响应
  * 自动批准工具
  * 创建提交
  * 自定义系统提示
  * 继续对话
* 后续步骤

### [mcp](https://code.claude.com/docs/zh-CN/mcp.md)

* 您可以使用 MCP 做什么
* 流行的 MCP servers
* 安装 MCP servers
  * 选项 1：添加远程 HTTP server
  * 选项 2：添加远程 SSE server
  * 选项 3：添加本地 stdio server
  * 管理您的 servers
  * 动态工具更新
  * Plugin 提供的 MCP servers
* MCP 安装范围
  * 本地范围
  * 项目范围
  * 用户范围
  * 选择正确的范围
  * 范围层次和优先级
  * `.mcp.json` 中的环境变量扩展
* 实际示例
  * 示例：使用 Sentry 监控错误
  * 示例：连接到 GitHub 进行代码审查
  * 示例：查询您的 PostgreSQL 数据库
* 使用远程 MCP servers 进行身份验证
  * 使用预配置的 OAuth 凭证
* 从 JSON 配置添加 MCP servers
* 从 Claude Desktop 导入 MCP servers
* 将 Claude Code 用作 MCP server
* MCP 输出限制和警告
* 使用 MCP 资源
  * 引用 MCP 资源
* 使用 MCP Tool Search 扩展
  * 它如何工作
  * 对于 MCP server 作者
  * 配置工具搜索
* 使用 MCP prompts 作为命令
  * 执行 MCP prompts
* 托管 MCP 配置
  * 选项 1：使用 managed-mcp.json 进行独占控制
  * 选项 2：使用允许列表和拒绝列表进行基于策略的控制
    * 限制选项
    * 示例配置
    * 基于命令的限制如何工作
    * 基于 URL 的限制如何工作
    * 允许列表行为（`allowedMcpServers`）
    * 拒绝列表行为（`deniedMcpServers`）
    * 重要说明

### [troubleshooting](https://code.claude.com/docs/zh-CN/troubleshooting.md)

* 常见安装问题
  * Windows 安装问题：WSL 中的错误
  * WSL2 sandbox 设置
  * Linux 和 Mac 安装问题：权限或命令未找到错误
    * 推荐解决方案：本地 Claude Code 安装
  * Windows："Claude Code on Windows requires git-bash"
  * Windows："installMethod is native, but claude command not found"
* 权限和身份验证
  * 重复的权限提示
  * 身份验证问题
* 配置文件位置
  * 重置配置
* 性能和稳定性
  * 高 CPU 或内存使用
  * 命令挂起或冻结
  * 搜索和发现问题
  * WSL 上的搜索结果缓慢或不完整
* IDE 集成问题
  * WSL2 上未检测到 JetBrains IDE
    * WSL2 网络模式
  * 报告 Windows IDE 集成问题（本地和 WSL）
  * JetBrains（IntelliJ、PyCharm 等）终端中的 Escape 键不工作
* Markdown 格式问题
  * 代码块中缺少语言标签
  * 不一致的间距和格式
  * markdown 生成的最佳实践
* 获取更多帮助

## 部署

### [third-party-integrations](https://code.claude.com/docs/zh-CN/third-party-integrations.md)

* 比较部署选项
* 配置代理和网关
  * Amazon Bedrock
  * Microsoft Foundry
  * Google Vertex AI
* 组织的最佳实践
  * 投资文档和内存
  * 简化部署
  * 从引导使用开始
  * 配置安全策略
  * 利用 MCP 进行集成
* 后续步骤

### [amazon-bedrock](https://code.claude.com/docs/zh-CN/amazon-bedrock.md)

* 先决条件
* 设置
  * 1. 提交用例详情
  * 2. 配置 AWS 凭证
    * 高级凭证配置
      * 示例配置
      * 配置设置说明
  * 3. 配置 Claude Code
  * 4. 模型配置
  * 5. 输出 token 配置
* IAM 配置
* AWS Guardrails
* 故障排除
* 其他资源

### [google-vertex-ai](https://code.claude.com/docs/zh-CN/google-vertex-ai.md)

* 先决条件
* 区域配置
* 设置
  * 1. 启用 Vertex AI API
  * 2. 请求模型访问
  * 3. 配置 GCP 凭证
  * 4. 配置 Claude Code
  * 5. 模型配置
* IAM 配置
* 1M token context window
* 故障排除
* 其他资源

### [microsoft-foundry](https://code.claude.com/docs/zh-CN/microsoft-foundry.md)

* 先决条件
* 设置
  * 1. 配置 Microsoft Foundry 资源
  * 2. 配置 Azure 凭证
  * 3. 配置 Claude Code
* Azure RBAC 配置
* 故障排除
* 其他资源

### [network-config](https://code.claude.com/docs/zh-CN/network-config.md)

* 代理配置
  * 环境变量
  * 基本身份验证
* 自定义 CA 证书
* mTLS 身份验证
* 网络访问要求
* 其他资源

### [llm-gateway](https://code.claude.com/docs/zh-CN/llm-gateway.md)

* 网关要求
* 配置
  * 模型选择
* LiteLLM 配置
  * 先决条件
  * 基本 LiteLLM 设置
    * 身份验证方法
      * 静态 API 密钥
      * 使用助手的动态 API 密钥
    * 统一端点（推荐）
    * 特定于提供商的传递端点（替代方案）
      * 通过 LiteLLM 的 Claude API
      * 通过 LiteLLM 的 Amazon Bedrock
      * 通过 LiteLLM 的 Google Vertex AI
* 其他资源

### [devcontainer](https://code.claude.com/docs/zh-CN/devcontainer.md)

* 主要功能
* 4 步快速入门
* 配置分解
* 安全功能
* 自定义选项
* 示例用例
  * 安全客户端工作
  * 团队入职
  * 一致的 CI/CD 环境
* 相关资源

## 管理

### [setup](https://code.claude.com/docs/zh-CN/setup.md)

* 系统要求
  * 其他依赖项
* 安装
  * 特定于平台的设置
  * 身份验证
    * 对于个人
    * 对于团队和组织
  * 安装特定版本
  * 二进制完整性和代码签名
* NPM 安装（已弃用）
* Windows 设置
* 更新 Claude Code
  * 自动更新
  * 配置发布频道
  * 禁用自动更新
  * 手动更新
* 卸载 Claude Code
  * 本地安装
  * Homebrew 安装
  * WinGet 安装
  * NPM 安装
  * 清理配置文件（可选）

### [authentication](https://code.claude.com/docs/zh-CN/authentication.md)

* 身份验证方法
  * Claude for Teams 或 Enterprise
  * Claude Console 身份验证
  * 云提供商身份验证
* 凭证管理
* 另请参阅

### [security](https://code.claude.com/docs/zh-CN/security.md)

* 我们如何处理安全
  * 安全基础
  * 基于权限的架构
  * 内置保护
  * 用户责任
* 防止提示注入
  * 核心保护
  * 隐私保障
  * 其他保障
* MCP 安全
* IDE 安全
* 云执行安全
* 安全最佳实践
  * 使用敏感代码
  * 团队安全
  * 报告安全问题
* 相关资源

### [data-usage](https://code.claude.com/docs/zh-CN/data-usage.md)

* 数据策略
  * 数据培训策略
  * 开发合作伙伴计划
  * 使用 `/bug` 命令的反馈
  * 会话质量调查
  * 数据保留
* 数据访问
* 本地 Claude Code：数据流和依赖项
  * 云执行：数据流和依赖项
* 遥测服务
* 按 API 提供商的默认行为

### [monitoring-usage](https://code.claude.com/docs/zh-CN/monitoring-usage.md)

* 快速开始
* 管理员配置
* 配置详情
  * 常见配置变量
  * 指标基数控制
  * 动态标头
    * 设置配置
    * 脚本要求
    * 刷新行为
  * 多团队组织支持
  * 示例配置
* 可用指标和事件
  * 标准属性
  * 指标
  * 指标详情
    * 会话计数器
    * 代码行计数器
    * 拉取请求计数器
    * 提交计数器
    * 成本计数器
    * Token 计数器
    * 代码编辑工具决策计数器
    * 活跃时间计数器
  * 事件
    * 用户提示事件
    * 工具结果事件
    * API 请求事件
    * API 错误事件
    * 工具决策事件
* 解释指标和事件数据
  * 使用监控
  * 成本监控
  * 警报和分段
  * 事件分析
* 后端考虑
  * 对于指标
  * 对于事件/日志
* 服务信息
* ROI 测量资源
* 安全/隐私考虑
* 在 Amazon Bedrock 上监控 Claude Code

### [costs](https://code.claude.com/docs/zh-CN/costs.md)

* 跟踪您的成本
  * 使用 `/cost` 命令
* 为团队管理成本
  * 速率限制建议
  * Agent team token 成本
* 减少 token 使用
  * 主动管理 context
  * 选择正确的模型
  * 减少 MCP server 开销
  * 为类型化语言安装代码智能 plugins
  * 将处理卸载到 hooks 和 skills
  * 将说明从 CLAUDE.md 移到 skills
  * 调整扩展思考
  * 将冗长操作委派给 subagents
  * 管理 agent team 成本
  * 编写具体的提示
  * 在复杂任务上高效工作
* 后台 token 使用
* 了解 Claude Code 行为的变化

### [analytics](https://code.claude.com/docs/zh-CN/analytics.md)

* 访问 Teams 和 Enterprise 的分析
  * 启用贡献指标
  * 查看摘要指标
  * 探索图表
    * 跟踪采用
    * 衡量每个用户的 PR
    * 查看拉取请求分解
    * 查找顶级贡献者
  * PR 归属
    * 标记标准
    * 归属流程
    * 时间窗口
    * 排除的文件
    * 归属说明
  * 从分析中获得最大收益
    * 监控采用
    * 衡量 ROI
    * 识别超级用户
    * 以编程方式访问数据
* 访问 API 客户的分析
  * 查看团队见解
* 相关资源

### [plugin-marketplaces](https://code.claude.com/docs/zh-CN/plugin-marketplaces.md)

* 概述
* 演练：创建本地市场
* 创建市场文件
* 市场架构
  * 必需字段
  * 所有者字段
  * 可选元数据
* Plugin 条目
  * 必需字段
  * 可选 plugin 字段
* Plugin 源
  * 相对路径
  * GitHub 存储库
  * Git 存储库
  * 高级 plugin 条目
* 托管和分发市场
  * 在 GitHub 上托管（推荐）
  * 在其他 git 服务上托管
  * 私有存储库
  * 在分发前在本地测试
  * 为您的团队要求市场
  * 托管市场限制
    * 常见配置
    * 限制如何工作
* 验证和测试
* 故障排除
  * 市场未加载
  * 市场验证错误
  * Plugin 安装失败
  * 私有存储库身份验证失败
  * 相对路径的 Plugins 在基于 URL 的市场中失败
  * 安装后找不到文件
* 另请参阅

## 配置

### [settings](https://code.claude.com/docs/zh-CN/settings.md)

* 配置范围
  * 可用范围
  * 何时使用每个范围
  * 范围如何相互作用
  * 什么使用范围
* 设置文件
  * 可用设置
  * 权限设置
  * 权限规则语法
  * Sandbox 设置
  * 归属设置
  * 文件建议设置
  * Hook 配置
  * 设置优先级
  * 关于配置系统的关键点
  * 系统提示
  * 排除敏感文件
* Subagent 配置
* Plugin 配置
  * Plugin 设置
    * `enabledPlugins`
    * `extraKnownMarketplaces`
    * `strictKnownMarketplaces`
  * 管理 plugins
* 环境变量
* 可用于 Claude 的工具
  * Bash 工具行为
  * 使用 hooks 扩展工具
* 另请参阅

### [permissions](https://code.claude.com/docs/zh-CN/permissions.md)

* 权限系统
* 管理权限
* 权限模式
* 权限规则语法
  * 匹配工具的所有使用
  * 使用说明符进行细粒度控制
  * 通配符模式
* 特定于工具的权限规则
  * Bash
  * Read 和 Edit
  * WebFetch
  * MCP
  * Task（subagents）
* 使用 hooks 扩展权限
* 工作目录
* 权限如何与 sandboxing 相互作用
* 托管设置
  * 仅托管设置
* 设置优先级
* 示例配置
* 另请参阅

### [sandboxing](https://code.claude.com/docs/zh-CN/sandboxing.md)

* 概述
* 为什么 sandboxing 很重要
* 它如何工作
  * 文件系统隔离
  * 网络隔离
  * 操作系统级强制
* 入门
  * 先决条件
  * 启用 sandboxing
  * Sandbox 模式
  * 配置 sandboxing
* 安全优势
  * 防止提示注入
  * 减少攻击面
  * 透明操作
* 安全限制
* sandboxing 与权限的关系
* 高级用法
  * 自定义代理配置
  * 与现有安全工具的集成
* 最佳实践
* 开源
* 限制
* 另请参阅

### [terminal-config](https://code.claude.com/docs/zh-CN/terminal-config.md)

* 主题和外观
* 换行符
* 通知设置
  * iTerm 2 系统通知
  * 自定义通知 hooks
* 处理大型输入
* Vim 模式

### [model-config](https://code.claude.com/docs/zh-CN/model-config.md)

* 可用模型
  * 模型别名
  * 设置您的模型
* 特殊模型行为
  * `default` 模型设置
  * `opusplan` 模型设置
  * 调整努力级别
  * 使用 \[1m] 扩展 context
* 检查您的当前模型
* 环境变量
  * Prompt caching 配置

### [memory](https://code.claude.com/docs/zh-CN/memory.md)

* 确定内存类型
* CLAUDE.md 导入
* Claude 如何查找内存
  * 从其他目录加载内存
* 使用 `/memory` 直接编辑内存
* 设置项目内存
* 使用 `.claude/rules/` 的模块化规则
  * 基本结构
  * 特定于路径的规则
  * Glob 模式
  * 子目录
  * 符号链接
  * 用户级规则
* 组织级内存管理
* 内存最佳实践

### [statusline](https://code.claude.com/docs/zh-CN/statusline.md)

* 创建自定义状态行
* 它如何工作
* JSON 输入结构
* 示例脚本
  * 简单状态行
  * Git 感知状态行
  * Python 示例
  * Node.js 示例
  * 助手函数方法
  * Context Window 使用
* 提示
* 故障排除

### [keybindings](https://code.claude.com/docs/zh-CN/keybindings.md)

* 配置文件
* 上下文
* 可用操作
  * 应用操作
  * 历史操作
  * 聊天操作
  * 自动完成操作
  * 确认操作
  * 权限操作
  * 记录操作
  * 历史搜索操作
  * 任务操作
  * 主题操作
  * 帮助操作
  * 标签操作
  * 附件操作
  * 页脚操作
  * 消息选择器操作
  * Diff 操作
  * 模型选择器操作
  * 选择操作
  * Plugin 操作
  * 设置操作
* 按键语法
  * 修饰符
  * 大写字母
  * 和弦
  * 特殊键
* 取消绑定默认快捷键
* 保留快捷键
* 终端冲突
* Vim 模式交互
* 验证

## 参考

### [cli-reference](https://code.claude.com/docs/zh-CN/cli-reference.md)

* CLI 命令
* CLI 标志
  * Agents 标志格式
  * 系统提示标志
* 另请参阅

### [interactive-mode](https://code.claude.com/docs/zh-CN/interactive-mode.md)

* 键盘快捷键
  * 常规控制
  * 文本编辑
  * 主题和显示
  * 多行输入
  * 快速命令
* 内置命令
  * MCP prompts
* Vim 编辑器模式
  * 模式切换
  * 导航（NORMAL 模式）
  * 编辑（NORMAL 模式）
  * 文本对象（NORMAL 模式）
* 命令历史
  * 使用 Ctrl+R 反向搜索
* 后台 bash 命令
  * 后台工作原理
  * 使用 `!` 前缀的 Bash 模式
* 提示建议
* 任务列表
* PR 审查状态
* 另请参阅

### [checkpointing](https://code.claude.com/docs/zh-CN/checkpointing.md)

* checkpoints 如何工作
  * 自动跟踪
  * 回退更改
* 常见用例
* 限制
  * Bash 命令更改未跟踪
  * 外部更改未跟踪
  * 不是版本控制的替代品
* 另请参阅

### [hooks](https://code.claude.com/docs/zh-CN/hooks.md)

* Hook 生命周期
  * Hook 如何解析
* 配置
  * Hook 位置
  * 匹配器模式
    * 匹配 MCP 工具
  * Hook 处理程序字段
    * 常见字段
    * 命令 hook 字段
    * 提示和代理 hook 字段
  * 按路径引用脚本
  * skills 和 agents 中的 hooks
  * `/hooks` 菜单
  * 禁用或删除 hooks
* Hook 输入和输出
  * 常见输入字段
  * 退出代码输出
    * 每个事件的退出代码 2 行为
  * JSON 输出
    * 决策控制
* Hook 事件
  * SessionStart
    * SessionStart 输入
    * SessionStart 决策控制
    * 持久化环境变量
  * UserPromptSubmit
    * UserPromptSubmit 输入
    * UserPromptSubmit 决策控制
  * PreToolUse
    * PreToolUse 输入
      * Bash
      * Write
      * Edit
      * Read
      * Glob
      * Grep
      * WebFetch
      * WebSearch
      * Task
    * PreToolUse 决策控制
  * PermissionRequest
    * PermissionRequest 输入
    * PermissionRequest 决策控制
  * PostToolUse
    * PostToolUse 输入
    * PostToolUse 决策控制
  * PostToolUseFailure
    * PostToolUseFailure 输入
    * PostToolUseFailure 决策控制
  * Notification
    * Notification 输入
  * SubagentStart
    * SubagentStart 输入
  * SubagentStop
    * SubagentStop 输入
  * Stop
    * Stop 输入
    * Stop 决策控制
  * PreCompact
    * PreCompact 输入
  * SessionEnd
    * SessionEnd 输入
* 基于提示的 hooks
  * 基于提示的 hooks 如何工作
  * 提示 hook 配置
  * 响应架构
  * 示例：多条件 Stop hook
* 基于代理的 hooks
  * 代理 hooks 如何工作
  * 代理 hook 配置
* 在后台运行 hooks
  * 配置异步 hook
  * 异步 hooks 如何执行
  * 示例：文件更改后运行测试
  * 限制
* 安全考虑
  * 免责声明
  * 安全最佳实践
* 调试 hooks

### [plugins-reference](https://code.claude.com/docs/zh-CN/plugins-reference.md)

* Plugin 组件参考
  * Skills
  * Agents
  * Hooks
  * MCP servers
  * LSP servers
* Plugin 安装范围
* Plugin 清单架构
  * 完整架构
  * 必需字段
  * 元数据字段
  * 组件路径字段
  * 路径行为规则
  * 环境变量
* Plugin 缓存和文件解析
  * Plugin 缓存如何工作
  * 路径遍历限制
  * 使用外部依赖项
* Plugin 目录结构
  * 标准 plugin 布局
  * 文件位置参考
* CLI 命令参考
  * plugin install
  * plugin uninstall
  * plugin enable
  * plugin disable
  * plugin update
* 调试和开发工具
  * 调试命令
  * 常见问题
  * 示例错误消息
  * Hook 故障排除
  * MCP server 故障排除
  * 目录结构错误
* 分发和版本控制参考
  * 版本管理
* 另请参阅

## 资源

### [legal-and-compliance](https://code.claude.com/docs/zh-CN/legal-and-compliance.md)

* 法律协议
  * 许可证
  * 商业协议
* 合规性
  * 医疗保健合规性（BAA）
* 安全和信任
  * 信任和安全
  * 安全漏洞报告
