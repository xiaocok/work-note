> ## Documentation Index
> Fetch the complete documentation index at: https://code.claude.com/docs/llms.txt
> Use this file to discover all available pages before exploring further.

# Claude Code 钩子入门

> 学习如何通过注册 shell 命令来自定义和扩展 Claude Code 的行为

Claude Code 钩子是用户定义的 shell 命令，在 Claude Code 生命周期的各个点执行。钩子提供对 Claude Code 行为的确定性控制，确保某些操作总是发生，而不是依赖 LLM 选择运行它们。

<Tip>
  有关钩子的参考文档，请参阅 [钩子参考](/zh-CN/hooks)。
</Tip>

钩子的示例用例包括：

* **通知**：自定义当 Claude Code 等待您的输入或权限运行某些操作时如何获得通知。
* **自动格式化**：在每次文件编辑后对 .ts 文件运行 `prettier`，对 .go 文件运行 `gofmt` 等。
* **日志记录**：跟踪和计数所有执行的命令以进行合规性或调试。
* **反馈**：当 Claude Code 生成不遵循您的代码库约定的代码时提供自动反馈。
* **自定义权限**：阻止对生产文件或敏感目录的修改。

通过将这些规则编码为钩子而不是提示指令，您将建议转变为应用级代码，每次在预期运行时执行。

<Warning>
  在添加钩子时，您必须考虑钩子的安全隐患，因为钩子在代理循环期间使用您当前环境的凭证自动运行。例如，恶意钩子代码可能会泄露您的数据。在注册钩子之前，始终审查您的钩子实现。

  有关完整的安全最佳实践，请参阅钩子参考文档中的 [安全考虑](/zh-CN/hooks#security-considerations)。
</Warning>

## 钩子事件概述

Claude Code 提供了在工作流的不同点运行的多个钩子事件：

* **PreToolUse**：在工具调用之前运行（可以阻止它们）
* **PermissionRequest**：在显示权限对话框时运行（可以允许或拒绝）
* **PostToolUse**：在工具调用完成后运行
* **UserPromptSubmit**：当用户提交提示时运行，在 Claude 处理之前
* **Notification**：当 Claude Code 发送通知时运行
* **Stop**：当 Claude Code 完成响应时运行
* **SubagentStop**：当子代理任务完成时运行
* **PreCompact**：在 Claude Code 即将运行压缩操作之前运行
* **SessionStart**：当 Claude Code 启动新会话或恢复现有会话时运行
* **SessionEnd**：当 Claude Code 会话结束时运行

每个事件接收不同的数据，并可以以不同的方式控制 Claude 的行为。

## 快速入门

在本快速入门中，您将添加一个钩子来记录 Claude Code 运行的 shell 命令。

### 前置条件

安装 `jq` 用于命令行中的 JSON 处理。

### 步骤 1：打开钩子配置

运行 `/hooks` [斜杠命令](/zh-CN/slash-commands) 并选择 `PreToolUse` 钩子事件。

`PreToolUse` 钩子在工具调用之前运行，可以阻止它们，同时向 Claude 提供关于如何不同处理的反馈。

### 步骤 2：添加匹配器

选择 `+ Add new matcher…` 仅在 Bash 工具调用上运行您的钩子。

为匹配器输入 `Bash`。

<Note>您可以使用 `*` 来匹配所有工具。</Note>

### 步骤 3：添加钩子

选择 `+ Add new hook…` 并输入此命令：

```bash  theme={null}
jq -r '"\(.tool_input.command) - \(.tool_input.description // "No description")"' >> ~/.claude/bash-command-log.txt
```

### 步骤 4：保存您的配置

对于存储位置，选择 `User settings`，因为您正在记录到您的主目录。然后此钩子将应用于所有项目，而不仅仅是您的当前项目。

然后按 `Esc` 直到您返回 REPL。您的钩子现在已注册。

### 步骤 5：验证您的钩子

再次运行 `/hooks` 或检查 `~/.claude/settings.json` 以查看您的配置：

```json  theme={null}
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '\"\\(.tool_input.command) - \\(.tool_input.description // \"No description\")\"' >> ~/.claude/bash-command-log.txt"
          }
        ]
      }
    ]
  }
}
```

### 步骤 6：测试您的钩子

要求 Claude 运行一个简单的命令，如 `ls`，并检查您的日志文件：

```bash  theme={null}
cat ~/.claude/bash-command-log.txt
```

您应该看到类似的条目：

```
ls - Lists files and directories
```

## 更多示例

<Note>
  有关完整的示例实现，请参阅我们公共代码库中的 [bash 命令验证器示例](https://github.com/anthropics/claude-code/blob/main/examples/hooks/bash_command_validator_example.py)。
</Note>

### 代码格式化钩子

在编辑后自动格式化 TypeScript 文件：

```json  theme={null}
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "jq -r '.tool_input.file_path' | { read file_path; if echo \"$file_path\" | grep -q '\\.ts$'; then npx prettier --write \"$file_path\"; fi; }"
          }
        ]
      }
    ]
  }
}
```

### Markdown 格式化钩子

自动修复 markdown 文件中缺失的语言标签和格式化问题：

```json  theme={null}
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/markdown_formatter.py"
          }
        ]
      }
    ]
  }
}
```

使用以下内容创建 `.claude/hooks/markdown_formatter.py`：

````python  theme={null}
#!/usr/bin/env python3
"""
Markdown formatter for Claude Code output.
Fixes missing language tags and spacing issues while preserving code content.
"""
import json
import sys
import re
import os

def detect_language(code):
    """Best-effort language detection from code content."""
    s = code.strip()
    
    # JSON detection
    if re.search(r'^\s*[{\[]', s):
        try:
            json.loads(s)
            return 'json'
        except:
            pass
    
    # Python detection
    if re.search(r'^\s*def\s+\w+\s*\(', s, re.M) or \
       re.search(r'^\s*(import|from)\s+\w+', s, re.M):
        return 'python'
    
    # JavaScript detection  
    if re.search(r'\b(function\s+\w+\s*\(|const\s+\w+\s*=)', s) or \
       re.search(r'=>|console\.(log|error)', s):
        return 'javascript'
    
    # Bash detection
    if re.search(r'^#!.*\b(bash|sh)\b', s, re.M) or \
       re.search(r'\b(if|then|fi|for|in|do|done)\b', s):
        return 'bash'
    
    # SQL detection
    if re.search(r'\b(SELECT|INSERT|UPDATE|DELETE|CREATE)\s+', s, re.I):
        return 'sql'
        
    return 'text'

def format_markdown(content):
    """Format markdown content with language detection."""
    # Fix unlabeled code fences
    def add_lang_to_fence(match):
        indent, info, body, closing = match.groups()
        if not info.strip():
            lang = detect_language(body)
            return f"{indent}```{lang}\n{body}{closing}\n"
        return match.group(0)
    
    fence_pattern = r'(?ms)^([ \t]{0,3})```([^\n]*)\n(.*?)(\n\1```)\s*$'
    content = re.sub(fence_pattern, add_lang_to_fence, content)
    
    # Fix excessive blank lines (only outside code fences)
    content = re.sub(r'\n{3,}', '\n\n', content)
    
    return content.rstrip() + '\n'

# Main execution
try:
    input_data = json.load(sys.stdin)
    file_path = input_data.get('tool_input', {}).get('file_path', '')
    
    if not file_path.endswith(('.md', '.mdx')):
        sys.exit(0)  # Not a markdown file
    
    if os.path.exists(file_path):
        with open(file_path, 'r', encoding='utf-8') as f:
            content = f.read()
        
        formatted = format_markdown(content)
        
        if formatted != content:
            with open(file_path, 'w', encoding='utf-8') as f:
                f.write(formatted)
            print(f"✓ Fixed markdown formatting in {file_path}")
    
except Exception as e:
    print(f"Error formatting markdown: {e}", file=sys.stderr)
    sys.exit(1)
````

使脚本可执行：

```bash  theme={null}
chmod +x .claude/hooks/markdown_formatter.py
```

此钩子自动：

* 检测未标记代码块中的编程语言
* 为语法突出显示添加适当的语言标签
* 修复过多的空行，同时保留代码内容
* 仅处理 markdown 文件（`.md`、`.mdx`）

### 自定义通知钩子

当 Claude 需要输入时获得桌面通知：

```json  theme={null}
{
  "hooks": {
    "Notification": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "notify-send 'Claude Code' 'Awaiting your input'"
          }
        ]
      }
    ]
  }
}
```

### 文件保护钩子

阻止对敏感文件的编辑：

```json  theme={null}
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "python3 -c \"import json, sys; data=json.load(sys.stdin); path=data.get('tool_input',{}).get('file_path',''); sys.exit(2 if any(p in path for p in ['.env', 'package-lock.json', '.git/']) else 0)\""
          }
        ]
      }
    ]
  }
}
```

## 了解更多

* 有关钩子的参考文档，请参阅 [钩子参考](/zh-CN/hooks)。
* 有关全面的安全最佳实践和安全指南，请参阅钩子参考文档中的 [安全考虑](/zh-CN/hooks#security-considerations)。
* 有关故障排除步骤和调试技术，请参阅钩子参考文档中的 [调试](/zh-CN/hooks#debugging)。
