



### 推荐的插件

https://zhuanlan.zhihu.com/p/2001617367025877608



#### planning-with-files

是一个 Claude Code 插件，借鉴了 **[Manus](https://zhida.zhihu.com/search?content_id=269871871&content_type=Article&match_order=1&q=Manus&zd_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJ6aGlkYV9zZXJ2ZXIiLCJleHAiOjE3NzAyOTg1NjgsInEiOiJNYW51cyIsInpoaWRhX3NvdXJjZSI6ImVudGl0eSIsImNvbnRlbnRfaWQiOjI2OTg3MTg3MSwiY29udGVudF90eXBlIjoiQXJ0aWNsZSIsIm1hdGNoX29yZGVyIjoxLCJ6ZF90b2tlbiI6bnVsbH0.Td2v9YjDVR41JJHTJUPuQTNHv6QiKRbzPZTLHnZ9E4E&zhida_source=entity)** 的设计思路，用持久化的 **Markdown** 文件来做规划、进度跟踪和知识存储。它关注的不是模型能力，而是把文件系统当作稳定的**工作记忆**来使用。

**planning-with-files** 的核心做法是为每个复杂任务维护**三个文件**：

```text
task_plan.md      → 拆分任务、规划阶段、追踪进度
findings.md       → 记录调研结果、关键发现与设计决策
progress.md       → 保存会话日志、实验记录和测试结果
```

##### 安装

**步骤一：把插件加入市场**

在 Claude Code 的终端中执行：

```text
/plugin marketplace add OthmanAdi/planning-with-files
```

**步骤二：安装插件**

```text
/plugin install planning-with-files@planning-with-files
```

安装完成后，重启 Claude Code，就可以使用了。



##### 运行

运行 `/planning-with-files:plan` + 任务描述

**恢复机制**

再次运行 `/planning-with-files` 时，技能会自动处理之前的会话数据



### 插件

#### 安装目录

```shell
/home/{user}/.claude/plugins
```

#### 安装

```shell
claude # 启动claude
/plugin marketplace add <market-place-path> 或者 <git-url> # 添加市场 本地路径 互联网地址
/plugin install <market-name>@<plugin-name>               # 安装插件
/plugin marketplace update <market-name>                  # 更新插件
/plugin marketplace remove <market-name>                  # 删除插件
/plugin list                                              # 查看插件列表
/<market-name>:<plugin-name> "message"                    # 使用指定插件
/<plugin-name> "message"                                  # 使用指定插件：当插件名称不冲突时
```

#### 调整本地开发的插件

```shell
claude --plugin-dir <本地开发的插件目录> --debug
```





### 插件开发

```
plugins
┗━━plugin-name  # 开发的插件名称
   ┣━━.calude-plugin
   ┃   ┣━━marketplace.json # claude code可以识别，可以使用/plugin marketplace add 方式安装插件
   ┃   ┗━━plugin.json      # 插件的描述信息
   ┣━━agents
   ┣━━hooks
   ┣━━skills
   ┗━━.mcp.json
```

marketplace.json

> 如果复制到claude的插件目录来实现插件的管理和维护，不通过市场来安装和维护，则不需要该文件

```json
{
  "name": "市场名称",
  "owner": {
    "name": "市场拥有者"
  },
  "plugins": [
    {
      "name": "插件名称",
      "source": "./skills/xxxx", // 该插件的目标：可以值skill,也可以是sub agent
      "description": "",         // 该插件的描述
    }
  ]
}
```

plugin.json

```json
{
  "name": "插件的名称",
  "description": "该插件的描述",
  "version": "1.0.0",
  "author": {
    "name": ""
  }
}
```

mcp.json

```json
{
  "mcpServers": {
    "xxxx": {  // mcp服务的名称标识
      "type": "http",
      "url": "http://www.xxxx.com/mcp" // mcp服务的域名，mcp服务的地址
    }
  }
}
```

