

https://opencode.ai/

https://opencode.ai/docs

https://github.com/anomalyco/opencode

https://github.com/anomalyco/opencode/blob/dev/README.zh.md



### 安装

```shell
# 直接安装 (YOLO)
curl -fsSL https://opencode.ai/install | bash

# 软件包管理器
npm i -g opencode-ai@latest         # 也可使用 bun/pnpm/yarn
scoop install opencode              # Windows
choco install opencode              # Windows
brew install anomalyco/tap/opencode # macOS 和 Linux（推荐，始终保持最新）
brew install opencode               # macOS 和 Linux（官方 brew formula，更新频率较低）
```

#### 安装目录

安装脚本按照以下优先级决定安装路径：

1. `$OPENCODE_INSTALL_DIR` - 自定义安装目录
2. `$XDG_BIN_DIR` - 符合 XDG 基础目录规范的路径
3. `$HOME/bin` - 如果存在或可创建的用户二进制目录
4. `$HOME/.opencode/bin` - 默认备用路径





### 启动

```shell
# cd 项目目录
opencode
```



#### 免费模型

- Big Pickle
  - 一个集成在开源AI编程助手 OpenCode 中的免费大语言模型，据信是基于智谱 GLM-4.6 微调而来的版本，特别擅长处理Python编程任务。‌‌
  - **基于GLM微调**‌：根据社区信息，Big Pickle 模型被认为是智谱（智谱AI）的 ‌**GLM-4.6 模型经过微调后的版本**‌，主要用于数据收集等任务。‌‌2
- GLM-4.7 Free
  - 智谱模型
- Kimi K2.5 Free
- MiniMax M2.1 Free
- Trinity Large Preview（免费默认模型：用户无需API密钥即可使用）
  - 初创公司 Arcee AI 发布的开源大模型 Trinity 系列中的一个版本



#### 添加收费模型

> 以智谱模型为例：智谱模型收费版性价比也高，对于中文和任务理解较好。

1. 启动opencode
2. 命令/connect 
3. 搜索：Zhipu AI Coding Plan
4. 输入 API key
5. 命令/models
6. 选择Zhipu AI Coding Plan中的：GML-4.7



#### 命令

```shell
/new    	# 新建一个会话
/session 	# 切换不同的会话
/models 	# 切换模型
/connect    # 添加新的模型提供商链接
/exit		# 退出opencode
```



### 使用

#### 全局规则

全局中文规则，否则会一直回答英文

> 项目目录：global-config.json

1. 启动opencode
2. 输入提示：新建一个全局规则文件，我希望始终用中文回复
3. 选择：OpenCode配置文件



#### 切换模式

Tab键切换模式

- Build：执行模式
- Plan：不会修改文件，可以使用plan模式规划，再使用build模式执行



#### 指定文件

@文件名 或者 路径+文件名



#### Skills

每次执行任务都是用skills，可以设置为全局配置，类似于上面的中文回复全局配置。

- 支持使用Claude Code的Skills
- 项目中的skills放在：项目根目录/.opencode/skills/具体的skills文件夹列表



##### Skills市场

https://skills.sh/

https://skillsmp.com/

https://skillsmp.com/zh



##### 推荐Skills

- UI【ui-ux-pro-max-skill】：

  - https://github.com/nextlevelbuilder/ui-ux-pro-max-skill

- web-design-guidelines：

  - 这是一个用于**审查网页 UI/UX 设计质量的 Skill**，可以让你的 AI Agent 按照现代 Web 界面设计规范检查代码和输出结果。

  - ```text
    npx skills add https://github.com/vercel-labs/agent-skills --skill web-design-guidelines
    ```

- vercel-react-best-practices

  - 这是一个**来自 Vercel 的 React 和 Next.js 性能优化最佳实践 Skill**，它把官方多年实战经验结构化成让 AI 代理可以直接调用的规则集。

  - ```text
    npx skills add https://github.com/vercel-labs/agent-skills --skill vercel-react-best-practices
    ```

- agent‑browser

  -  这个 Skill 提供了 **浏览器自动化能力**，让你的 AI Agent 可以**执行网页导航、表单填写、截图、数据抓取等任务**，适合处理需要与真实网页互动的工作场景，比如自动测试、信息收集或表单提交。

  - ```text
    打开 https://example.com 并截取全页截图
    ```

- 脑暴、规划文档、测试(适合coding)【superpowers】：

  - https://github.com/obra/superpowers

- 创建skills的skills【skill-creator】：

  - https://github.com/anthropics/skills

  - npx skills add https://github.com/anthropics/skills --skill skill-creator

- find-skills：

  - npx skills add https://github.com/vercel-labs/skills --skill find-skills
  - 使用时，通过 `npx skills find <关键词>` 命令
  - 收集与 “React 性能优化” 相关的 Skill：npx skills find react performance

- seo‑audit

  - 这是一个用于 **网站 SEO 审计的 Skill**，可以让你的 Agent 对网站做全面的搜索引擎优化检查，包括技术 SEO、内容优化建议等，让网站在搜索结果中更容易被发现。

  - ```text
    npx skills add https://github.com/coreyhaines31/marketingskills --skill seo-audit
    ```

  - 安装后，你可以这样用：请对这个网站做一次 SEO 审计，并给出优化建议



##### 安装skills

方式一：

- 下载之后放入指定位置

方式二：

- 安装具体某一个skills
  - npx skills add https://github.com/anthropics/skills --skill skill-creator
- 指定厂商的skills仓库，需要再次选择安装哪一个
  - npx skills add anthropics/skills
  - bunx skills add anthropics/skills
  - pnpm dlx skills add anthropics/skills



### 参考

https://www.bilibili.com/video/BV1A4zTBxEYx

https://zhuanlan.zhihu.com/p/2001041857476593143





