



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