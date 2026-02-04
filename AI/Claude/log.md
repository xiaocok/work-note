





### 📁 用户输入日志

> 这是用户输入的日志：包括提问、命令等。不包含claude code的运行日志。

Claude Code 的日志以 **JSONL**（每行一个 JSON 对象）格式存储在本地。路径根据操作系统有所不同。

| 系统              | 全局历史文件                          | 项目会话文件（按项目分类）                      |
| :---------------- | :------------------------------------ | :---------------------------------------------- |
| **Linux / macOS** | `~/.claude/history.jsonl`             | `~/.claude/projects/*/chat_*.jsonl`             |
| **Windows**       | `%USERPROFILE%\.claude\history.jsonl` | `%USERPROFILE%\.claude\projects\*\chat_*.jsonl` |

`~` 和 `%USERPROFILE%` 指你的用户主目录。项目路径会被编码成文件夹名。

#### 🔍 如何查看这些日志

**手动查看日志内容**

`cat /.claude/history.jsonl`

```json
{"display":"ls","pastedContents":{},"timestamp":1765502601101,"project":"/Users/user/python/test","sessionId":"7ec08a6c-c6ba-4940-9f1a-fb4dc402b4bb"}
{"display":"/init ","pastedContents":{},"timestamp":1765502850489,"project":"/Users/user/python/test","sessionId":"18f4de01-7c0d-4216-a29e-18112ce18c8f"}
{"display":"创建一个hello-world的python的程序","pastedContents":{},"timestamp":1766561751296,"project":"/Users/user/python/test","sessionId":"859f38b2-0959-4bd3-960e-3c7a429ddf0b"}
{"display":"<user_input>\nLAUNCH A SPEC DEVELOPMENT WORKFLOW\n\nCreate a requirements document for a new feature\n\nFeature Description: 创建一个冒泡排序的python公共库，支持数组参数传递，输出为已排序好的数组。然后编写一个测试案例。代码放入一个pkg/sort的包中。\n\nWorkspace path: /Users/mac/python/claude-code-router-test\nSpec base path: .claude/specs\n\nYou have full control over the naming and file creation.\n</user_input>","pastedContents":{},"timestamp":1766566590804,"project":"/Users/user/python/test","sessionId":"16ad8990-d6db-4f97-ac3d-032afd0f8428"}
```



你有三种选择，从简单到专业：

1. **用内置命令快速“继续”会话**：这是最直接的方法，无需处理文件。在终端里使用以下命令：

   - `claude -c` 或 `claude --continue`：恢复最近的会话。
   - `claude --resume`：从列表中选择一个会话来恢复。

   > 注：恢复功能依赖项目路径。如果你移动了项目目录，可能会找不到历史记录。

2. **直接查看原始 JSONL 文件**：适合快速确认或技术分析。你可以用系统命令（如 `cat`、`head`）或文本编辑器直接打开上述路径中的 `.jsonl` 文件。但原始 JSON 格式可读性较差。

3. **使用社区工具查看/导出**：为了获得最佳阅读体验，社区开发了专门工具来解析、美化和搜索这些日志。

   - **Claude Conversation Extractor**：可将日志导出为 Markdown、HTML 或 JSON，支持搜索和批量操作。

     ```bash
     # 安装并启动交互式界面
     pipx install claude-conversation-extractor
     claude-start
     ```

   - **claude-code-log**：专注于将日志转换为可读的 HTML 页面，并提供一个交互式界面（TUI）来浏览会话。

#### 💡 日志管理与清理建议

- **日志保留**：有资料指出这些本地日志默认可能保留**30天**，但建议你将其视为参考。
- **主动清理**：由于日志会累积占用空间，你可以定期手动删除 `~/.claude/` 目录下的文件，或使用上述导出工具备份重要会话后清理。

总结一下，要查看日志，最快捷的方式是在项目目录下使用 `claude --resume` 命令；想深度浏览、搜索或备份历史对话，则推荐安装 **Claude Conversation Extractor** 这类工具。

如果你需要了解特定工具的详细用法，或者想查找某一次特定会话的文件，我可以提供进一步的帮助。





### 运行日志

```shell
# 切换日志目录：
cd ~/.claude/debug

# 查看session_id
cat /.claude/history.jsonl

# 查看日志：cat session_id.txt
cat 18f4de01-7c0d-4216-a29e-18112ce18c8f.txt
```





