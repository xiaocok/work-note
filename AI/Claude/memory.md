## Memory (记忆) 的存储和总结机制
根据代码分析，Claude Code 有两套独立的记忆系统：

### 1. Auto Memory (长期记忆) - 文件系统持久化
位置 : ~/.claude/projects/<slug>/memory/

存储结构 ( src/memdir/memdir.ts ):

```
memoryDir/
├── MEMORY.md          # 索引文件 (入口点)
├── user_role.md       # 用户类型记忆
├── feedback_*.md      # 反馈记忆
├── project_*.md       # 项目记忆
└── reference_*.md     # 参考记忆
```
记忆类型 ( src/memdir/memoryTypes.ts ):

类型 说明 user 用户角色、目标、偏好 feedback 用户指导 (避免/继续做的事) project 项目上下文、目标、截止日期 reference 外部系统指针 (Linear, Slack等)

加载流程 ( src/constants/prompts.ts ):

```
systemPromptSection('memory', () => loadMemoryPrompt()),
```
MEMORY.md 加载 ( src/memdir/memdir.ts ):

- 通过 loadMemoryPrompt() 读取
- 如果 MEMORY.md 超过 200 行或 25KB，会被截断并警告
- 模型在每次对话开始时读取
### 2. Session Memory (会话记忆) - 当前会话总结
位置 : ~/.claude/sessions/<session_id>/memory.md

用途 : 自动维护当前会话的笔记，在 compaction 时使用

模板结构 ( src/services/SessionMemory/prompts.ts ):

```
# Session Title
# Current State          # 当前正在做什么
# Task specification     # 用户要求什么
# Files and Functions    # 重要文件
# Workflow              # 常用命令顺序
# Errors & Corrections  # 错误和纠正
# Codebase and System   # 系统文档
# Learnings             # 经验教训
# Key results           # 关键结果
# Worklog               # 工作日志
```
触发条件 ( src/services/SessionMemory/sessionMemory.ts ):

```
shouldExtractMemory() 检查:
1. 初始化阈值: token数量超过限制
2. 更新阈值: 
   - token增长超过 minimumTokensBetweenUpdate
   - tool calls 超过 toolCallsBetweenUpdates (默认50)
3. 最后一次 assistant turn 没有 tool calls (确保在自然断点提取)
```
提取流程 ( src/services/SessionMemory/sessionMemory.ts ):

1. 注册 postSamplingHook 在采样后触发
2. 创建子 agent (forked) 执行提取
3. 只允许 Edit 操作到 memory 文件
4. 使用 runForkedAgent 利用 prompt caching
大小限制 ( src/services/SessionMemory/prompts.ts ):

- 每 section 最大 ~2000 tokens
- 总计最大 ~12000 tokens
- 超过时会压缩
### 3. Compact 时的处理
当发生自动压缩 (auto-compact) 时:

1. Session Memory 会被截断 ( src/services/SessionMemory/prompts.ts ) 并放入压缩消息
2. Auto Memory (MEMORY.md) 保持在 system prompt 中不变
### 总结流程图
```
┌─────────────────────────────────────────────────────────────┐
│                    Auto Memory (长期)                        │
├─────────────────────────────────────────────────────────────┤
│  触发: 用户说 "remember X" 或 模型判断需要保存               │
│  存储: ~/.claude/projects/<slug>/memory/MEMORY.md          │
│  加载: 每次对话开始时通过 systemPromptSection('memory')      │
│  格式: frontmatter (name/description/type) + 内容          │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│                  Session Memory (会话)                      │
├─────────────────────────────────────────────────────────────┤
│  触发: postSamplingHook + token/tool_call 阈值               │
│  存储: ~/.claude/sessions/<id>/memory.md                   │
│  方式: forked subagent 调用 Edit tool 更新                   │
│  用途: compaction 时提供会话上下文                          │
│  限制: 每section ~2000 tokens, 总计 ~12000 tokens           │
└─────────────────────────────────────────────────────────────┘
```