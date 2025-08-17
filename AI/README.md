

### 关键词

人工智能、机器学习、深度学习、生成式人工智能



#### 机器学习

监督学习

无监督学习

强化学习



#### 深度学习

神经网络分为：卷积神经网络(CNNs)、循环神经网络(RNNs)、Transformer网络





### 大模型训练

大模型训练分为3个阶段：预训练、SFT(监督微调)、RLHF(基于人类反馈的强化学习)



### 大模型分类

按场景分：

基础模型(大模型)

​	大语音模型(LLM)

​	多模态模型

​		计算机视觉模型-VLLM

​		音频处理模型



#### 大语音模型

专注于自然语言(NLP)，旨在处理语言、文章、对话等自然语言文本。通常基于深度学习架构(如Transformer模型)，经过大模型文本数据集训练而成，能够捕捉语言的复杂性，包括语法、语义、语境以及蕴含的文化和社会知识。

大语音模型典型应用包括：文本生成、问答系统、文本分类、机器编译、对话系统等。

示例：

1、GPT系列(OpenAI)：GPT-3、GPT-3.5、GPT-4等

2、Bard(Google)：用于提供信息丰富的、有创意的文本输出

3、通义千问(阿里云)



#### 多模态模型

多模态模型能够处理和理解来自不同感知通道（文本、图像、音频、视频等）的数据，并在这些模态之间建立关联和交互。

能够整合不同类型的输入信息，进行跨模态推理、生成和理解任务。

多模态大模型和应用涵盖：视觉问答、图像描述生成、跨模态检索、多媒体内容理解等领域。



### 大模型工作原理

#### 分词化(Tokenization)与词表映射

分词化是自然语言(NLP)的重要概念，它是将段落和句子分割成更小的分词(token)的过程。

分词化有不同的粒度分类：

- 词粒度(wold-level Tokenization)：西方语言，英语
- 字符粒度(Character-level)：中文，单个汉字分词化
- 子词粒度(Subword-level)：词根、词缀。处理新词(专有名词、网络用语)

每一个token都会通过预先设置好的词表，映射为一个token id，这是token的身份证，一句话最终会被表示为一个元素为token id的列表，供计算机进行下一步处理。



#### 文本生成过程



### AI Agent

#### Agent认知与原理分析

Agents是什么？

大语音模型可以接受输入，可以推理&分析，可以输出文字、代码、媒体。然而，无法像人类一样，拥有规划思考能力、运用各种工具与物理世界互动，以及拥有人类记忆能力。

AI Agent是基于LLM的能够自主理解、自主规划决策、执行复杂任务的**智能体**。

- LLM：接受输入、思考、输出
- 人类：LLM(接受输入、思考、输出) + 记忆 + 工具 + 规划 ----> Agents



#### Agent的流程

##### 规划(Planning)

智能体会把大型任务**分解为子任务**，并规划执行任务的流程；智能体会对任务的过程进行**思考和反思**，从而决定是继续执行任务，或者判断任务完结并终止执行。



###### 子任务分解

**思维链(Chain of Thoughts, CoT)**

思维链已经是一种比较标准的提示技术，能显著提升LLM完成复杂的任务。当我们对LLM要求[think step by step]，会发现LLM会把问题分解成多个步骤，一步一步思考和解决，能使得输出的结果更加准确。这是一种线性的思维方式。

**思维树(Tree-of-thought, ToT)**

对CoT的进一步扩展，在思维链的每一步，推理出多个分支，拓扑展开成一颗思维树。使用启发式方法评估每个推理分支对问题解决的贡献。选择搜索算法，使用**广度优先搜索(BFS)或深度优先搜索(DFS)**等算法开探索思维树，并进行前瞻和回溯。

![image-20250406005745586](E:\WWW\work-note\AI\README.assets\image-20250406005745586.png)

###### 反思与改进



- 仅推理(Reasoning  Only)
- 仅行动(Acting Only)
- 推理+行动(Reasoning and Acting)



ReAct框架（‌**Reasoning and Acting**‌）

是AI领域用于构建智能体（AI Agent）的通用框架，通过结合逻辑推理与动态行动，使大语言模型（LLM）能够模仿人类思维解决复杂任务。



##### 记忆(Memory)

**生活中的记忆机制：**

- 感觉记忆(Sensory Memory)：
- 短期记忆(或工作记忆)：
- 长期记忆：

**智能体中的记忆机制：**

- 形成记忆：大模型在大量的世界知识的数据集上进行训练。在预训练中，大模型通过调整神经元的权重来学习理解和生成人类语言，这可以被视为”记忆“的形成过程。通过使用深度学习和梯度下降等技术，大模型可以不断提高基于预测或生产文本的能力，进而形成世界记忆或长期记忆。
- 短期记忆：是指在执行任务的过程中的上下文，会在子任务的执行过程产生和缓存，在任务完结后被清空。
- 长期记忆：是长时间保存的信息，一般是指外部知识库，通常用向量数据库来存储和检索。



##### 工具使用(Tools/Toolkits)

为智能体配备工具API，比如：计算器、搜索工具、代码执行器、数据库查询工具等。有了这些工具API，智能体就可以是物理机世界交互，解决实际的问题。

Agent可以通过学习调用外部API来获取模型权重中所缺少的额外信息，这些信息包括当前信息，代码执行能力和访问专有信息源等。这对于预训练后难以修改的模型权重来说是非常重要的。

**构建第一个Agent**

> 使用langchain框架

https://python.langchain.com/docs/introduction/



API提供

https://python.langchain.com/api_reference/



模型提供

https://python.langchain.com/docs/integrations/providers/



**如何使用Tools/Toolkits**

工具包

https://python.langchain.com/docs/integrations/tools/

工具包示例：

Dall-E Image Generator【文本生成图片的模型】

https://python.langchain.com/docs/integrations/tools/dalle_image_generator/

Json工具包

https://python.langchain.com/docs/integrations/tools/json/



**给Agent添加记忆**

```python
from langchain.agents import Tool
from langchain.agents import AgentType
from langchain.agents import initialize_agent
from langchain.chains import LLMMatchChain
from langchain.prompts import MessagesPlaceholder
from langchain.memory import ConversationBufferMemory # 记忆组件
from langchain_community.chat_models import ChatOpenAI
# from langchain_community.agent_toolkits.load_tools import load_tools
from langchain_community.utilities import SerpAPIWrapper
from dotenv import load_dotenv

load_dotenv()

# 定义llm
llm = ChatOpenAI(
	temperature=0,
    model="gpt-4-1106-preview",
)

# 构建一个搜索工具
search = SerpAPIWrapper()

# 创建一个数学计算工具
llm_match_chain = LLMMatchChain(
	llm=llm,
    verbose=True
)

# tools = load_tools(["serpapi", "llm-match"], llm=llm)
tools = [
    Tool(
    	name = "Search",
        func=search.run,
        description=""
    ),
    Tool(
    	name = "Calculator",
        func=llm_match_chain.run,
        description=""
    ),
]
print(tools)

# 记忆组件
memory = ConversationBufferMemory(
	memory_key="caht_histroy",
    return_messages=True
)

agent_chain = initialize_agent(
	tools,
    llm, 
    agent=AgentType.OPENAI_FUNCTIONS,
    verbose=True,
    handle_parsing_errors=True, # 处理解析错误
    agent_kwargs={
    	"extra_prompt_messages": [MessagesPlaceholder(variable_name="chat")]  
    },
    memory=memory # 记忆组件
)

agent_chain.run("我叫什么名字？")
agent_chain.run("刚才我们都聊了些什么？")
```



##### 执行(Action)

根据规划和记忆来实施具体行动，这可能会涉及到与外部世界的互动或通过工具来完成任务。



#### Agent技术框架

##### langchain框架

上面已示例



##### Plan-and-Execute框架

计划与执行(Plan-and-Execute)框架侧重于先规划一系列的行动，然后执行。这个框架可以使大模型能够先综合考虑任务的多个方面，然后按照计划进行行动。应用在比较复杂的项目管理中或者需要多步决策的场景下会比较合适。

![image-20250406230015435](E:\WWW\work-note\AI\README.assets\image-20250406230015435.png)



##### Self-Ask框架

##### Thinking and Self-Refection框架

##### ReAct框架

##### AgentExecutor运行机制

##### 通过llamindex实现ReAct RAG Agent



#### Agent策略分析与Parer解读

#### 单Agent系统与多Agent系统

#### 个性化Agent应用定制全流程详将



### Python

#### pip加速

常用镜像源地址：
清华大学：https://pypi.tuna.tsinghua.edu.cn/simple
阿里云：https://mirrors.aliyun.com/pypi/simple/
豆瓣：https://pypi.douban.com/simple/

1. 临时使用镜像源

   在安装命令后加上 -i 参数指定镜像源，例如：

   ```shell
   pip install package_name -i https://pypi.tuna.tsinghua.edu.cn/simple
   ```

2. 永久配置镜像源

   可以通过修改或创建 pip.conf 文件来永久设置镜像源

   ```shell
   # Windows 系统
   # 在用户目录下创建或修改 %HOMEPATH%\pip\pip.ini 文件，添加以下内容：
   
   # Linux/macOS 系统
   # 在用户目录下创建或修改 ~/.pip/pip.conf 文件，添加以下内容：
   
   [global]
   index-url = https://pypi.tuna.tsinghua.edu.cn/simple
   ```

3. 使用代理

   如果需要通过代理服务器访问网络，可以使用 --proxy 参数：

   ```shell
   pip install package_name --proxy http://user:password@proxy_ip:port
   ```

4. 升级 pip

   确保你的 pip 是最新版本，以获得更好的性能和兼容性：

   ```shell
   pip install --upgrade pip
   ```

   通过以上方法，可以显著提升 pip 的下载速度，加快 Python 包的安装过程。



#### 安装路径

Python安装路径

```shell
用户目录\AppData\Local\Programs\Python\Python<版本>

C:\Users\用户\AppData\Local\Programs\Python\Python<版本>
```

PIP安装路径

```shell
# Windows：
C:\Users\用户\AppData\Local\Programs\Python\Python<版本>\Lib\site-packages

# macOS / Linux：
/usr/local/lib/python<版本>/site-packages/
```

虚拟环境（virtual environment）

```shell
# Windows 虚拟环境：
<你的项目目录>\.venv\Lib\site-packages\

# Unix 或 macOS 虚拟环境：
<你的项目目录>/venv/lib/python<版本>/site-packages/
```

> 虚拟环境需要激活

```shell
# Linux/macOS
source venv/bin/activate
# 激活后提示符会变成这样：
(venv) user@machine:~/project$

# Windows CMD
.venv\Scripts\activate
# 激活后提示符会变成这样：
(venv) C:\your\project\path>
```

**额外建议：避免混淆多个 Python 环境**

可以使用更明确的方式调用虚拟环境中的 pip：

```shell
./venv/bin/pip install gensim nltk   # Linux/macOS
.venv\Scripts\pip install gensim nltk  # Windows
```



**确认环境**

```shell
which python / where python

C:\Users\YourName\AppData\Local\Programs\Python\Python<版本>\python.exe
C:\Users\YourName\AppData\Local\Microsoft\WindowsApps\python.exe

# 安装了Anaconda/Miniconda等工具的效果
C:\Users\YourName\Anaconda3\python.exe
C:\Users\YourName\AppData\Local\Programs\Python\Python<版本>\python.exe
```

- 第一个路径 是你实际安装的 Python 版本。
- 第二个路径（如果有）可能是 Windows 10/11 自带的 Microsoft Store 版本 Python（通常不推荐使用）。
- 如果你安装了 Anaconda、Miniconda 等工具，也可能出现在结果中。





### AI框架

训练：Llama2开源 -> PyTorch开源训练框架 -> CUDA



### AI训练

#### 安装Anaconda

地址：

https://www.anaconda.com/download/

下载：

https://www.anaconda.com/download/success

Python版本

**Python 3.12**

下载文件名

Anaconda3-2024.10-1-Windows-x86_64.exe



### 大模型本地运行

#### Ollama

```shell
ollama run deepseek-r1:1.5b
```



验证

```shell
# 浏览器打开
http://localhost:11434/

# Ollama is running
```



API接口

[Ollama API使用详解！在本地调用API的详细教程](https://zhuanlan.zhihu.com/p/1900147836340926098)

```shell
POST http://localhost:11434/api/generate

Header
Content-Type: application/json

raw Body
{
    "model": "deepseek-r1:1.5b",
    "prompt":"你是谁？",
    "stream":false
}

响应：
{
    "model": "deepseek-r1:1.5b",
    "created_at": "2025-05-18T10:15:40.4866178Z",
    "response": "<think>\n\n</think>\n\n您好！我是由中国的深度求索（DeepSeek）公司开发的智能助手DeepSeek-R1。如您有任何任何问题，我会尽我所能为您提供帮助。",
    "done": true,
    "done_reason": "stop",
    "context": [
        151644,
        56568,
        100165,
        100165,
        11319,
        151645,
        151648,
        271,
        151649,
        271,
        111308,
        6313,
        104198,
        67071,
        105538,
        102217,
        30918,
        50984,
        9909,
        33464,
        39350,
        7552,
        73218,
        100013,
        9370,
        100168,
        110498,
        33464,
        39350,
        10911,
        16,
        1773,
        29524,
        87026,
        110117,
        99885,
        86119,
        3837,
        105351,
        99739,
        35946,
        111079,
        113445,
        100364,
        1773
    ],
    "total_duration": 4420704300,
    "load_duration": 1826048600,
    "prompt_eval_count": 7,
    "prompt_eval_duration": 142000000,
    "eval_count": 40,
    "eval_duration": 2448000000
}
```



```shell
curl --location --request POST 'http://localhost:11434/api/generate' \
  --header 'Content-Type: application/json' \
  --data-raw '{
    "model": "llama3.2",
    "prompt": "Why is the sky blue?",
    "stream": false
}'
```



#### LM Studio

[Ollama平替！LM Studio本地大模型调用实战](https://zhuanlan.zhihu.com/p/1899576951523509763)

[LM Studio 教程：通过暴露本地端口实现局域网内客户端调用（LM Studio 作为本地 LLM API 服务器）](https://blog.csdn.net/a772304419/article/details/145800588)

[LMstudio 部署模型后如何联网使用API](https://blog.csdn.net/yexiaomodemo/article/details/146222893)



官方文档：https://lmstudio.ai/docs

模型：https://lmstudio.ai/models

```shell
POST http://localhost:1234/v1/chat/completions

Header
Content-Type: application/json

raw Body
{
    "model": "deepseek-r1:1.5b",
    "messages": [
        {
            "role": "user", 
            "content": "你的问题"
        }
    ]
}

响应：
{
    "id": "chatcmpl-vp35s423iqhgavdi6kmuc",
    "object": "chat.completion",
    "created": 1747571569,
    "model": "deepseek-r1-distill-qwen-1.5b",
    "choices": [
        {
            "index": 0,
            "logprobs": null,
            "finish_reason": "stop", # 标识结束，后续不需要再交互
            "message": {
                "role": "assistant", # 机器人的回答
                "content": "<think>\n\n</think>\n\n您好！我是由中国的深度求索（DeepSeek）公司开发的智能助手DeepSeek-R1。有关模型和产品的详细内容请参考官方文档。"
            }
        }
    ],
    "usage": {
        "prompt_tokens": 5,
        "completion_tokens": 37,
        "total_tokens": 42
    },
    "stats": {},
    "system_fingerprint": "deepseek-r1-distill-qwen-1.5b"
}
```



```shell
curl -X POST http://localhost:1234/v1/chat/completions \
-H "Authorization: Bearer $DASHSCOPE_API_KEY" \
-H "Content-Type: application/json" \
-d '{
    "model": "deepseek-r1",
    "messages": [
        {
            "role": "user", 
            "content": "你好"
        },
        {
        	# 机器人AI的回复内容
            "role": "assistant",
            "content": "你好！很高兴见到你，有什么我可以帮忙的吗？"
        },
        {
            "role": "user",
            "content": "你是谁？"
        }
    ]
}'
```



##### mcp的支持

[LM Studio 官方文档 Tool支持](https://lmstudio.ai/docs/app/api/tools)

[优雅地开发 MCP 服务器（一）MCP 中的 Resources，Prompts，Tools 和基本调试方法](https://zhuanlan.zhihu.com/p/32593727614)

[使用 MCP C# SDK 实现 MCP Tool ](https://it.sohu.com/a/879650344_121124363)

第一次：问问题，并告诉可用函数

```shell
curl -X POST http://localhost:1234/v1/chat/completions \
-H "Content-Type: application/json" \
-d '{
    "model": "deepseek-r1:1.5b",
    "messages": [
        {
            "role": "user", 
            "content": "成都的天气怎么样？"
        }
    ],
    "tools": [
        {
            "type": "function",
            "function": {
                "name": "get_current_weather",
                "description": "获取给定地点的天气",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "location": {
                            "type": "string",
                            "description": "城市，比如杭州，北京，上海"
                        }
                    },
                    "required": ["location"]
                }
            }
        }
    ],
    "tool_choice": "auto",  # 可选：控制是否强制调用某个工具
    "stream": "false"
}'

响应：
{
    "id": "chatcmpl-60wekyw63tsidef8ek15yl",
    "object": "chat.completion",
    "created": 1747573318,
    "model": "deepseek-r1-distill-qwen-1.5b",
    "choices": [
        {
            "index": 0,
            "logprobs": null,
            "finish_reason": "tool_calls",  // stop：结束，tool_calls：调用函数
            "message": {
                "role": "assistant",		// user：用户的文本，assistant AI机器人的回答
                "tool_calls": [
                    {
                        "id": "173859425",
                        "type": "function",
                        "function": {
                            "name": "get_current_weather",
                            "arguments": "{\"location\":\"成都\"}"
                        }
                    }
                ]
            }
        }
    ],
    "usage": {
        "prompt_tokens": 349,
        "completion_tokens": 275,
        "total_tokens": 624
    },
    "stats": {},
    "system_fingerprint": "deepseek-r1-distill-qwen-1.5b"
}
```

调用第三方函数获取天气

第二次：将天气返回给机器人

```shell
curl -X POST http://localhost:1234/v1/chat/completions \
-H "Content-Type: application/json" \
-d '{
    "model": "deepseek-r1:1.5b",
    "messages": [
        {
            "role": "user", 				// 用户的问题
            "content": "成都的天气怎么样？"
        }，
        {
            "role": "assistant", 			// 机器人的回答，响应中message的完整内容
            "tool_calls": [
               {
                    "id": "173859425",
                    "type": "function",
                    "function": {
                        "name": "get_current_weather",
                        "arguments": "{\"location\":\"成都\"}"
                    }
                }
            ]
        },
        {
        	"role": "tool",					// 调用tool的返回
        	"content": "37度"
        }
    ],
    "tool_choice": "auto",  # 可选：控制是否强制调用某个工具
    "stream": "false"
}'
```



### MCP

[MCP](https://github.com/modelcontextprotocol)

[MCP Servers](https://github.com/modelcontextprotocol/servers)

[MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)



#### MCP Inspector（MCP调试）

[Python-MCPInspector调试](https://blog.csdn.net/2301_77717148/article/details/147881127)



Python-MCPInspector调试

> 使用FastMCP开发MCPServer，熟悉【McpServer编码过程】+【MCPInspector调试方法】-> 可以这样理解：只编写一个McpServer，然后使用MCPInspector作为McpClient进行McpServer的调试



![6f411de4d70d401d921fde10792ab32a](E:\WWW\work-note\AI\README.assets\6f411de4d70d401d921fde10792ab32a.gif)



**整理流程**

1. 编写传统的Service业务代码
2. 在Service业务代码头上添加@tool装饰器，即可实现FastMCP的Tool功能
3. 在Service业务代码头上添加@mcp.tool()装饰器，即可实现FastMCP的McpServer功能
4. 主程序指定运行方法-stdio进程启动（但是不要自己去启动）
5. 使用MCPInspector调试McpServer（2个步骤）
   【mcp dev city_02_mcp_server.py】是启动mcpInspector并指定mcpServer的路径，
   然后在Inspector中启动city_02_mcp_server.py->【uv run --with mcp mcp run city_02_mcp_server.py】



**MCP Inspector调试**

> 运行机制：先运行【MCPInspector】再运行【uv run --with mcp mcp run city_02_mcp_server.py】

1. 安装MCP Inspector

   ```shell
   # 1-安装MCP Inspector
   pip install mcp[cli]
   ```

2. 运行MCP Inspector服务

   ```shell
   # 2-运行MCP Inspector
   # 运行MCP Inspector的Web客户端
   mcp dev city_02_mcp_server.py
   ```

3. 访问MCP Inspector网页

   > 新开命令行终端，运行调试的程序

   再运行【uv run --with mcp mcp run city_02_mcp_server.py】

   访问：http://127.0.0.1:6274



### 参考

[【全748集】目前B站最全最细的AI大模型零基础全套教程，2025最新版，包含所有干货！七天就能从小白到大神！少走99%的弯路！存下吧！很难找全的！](https://www.bilibili.com/video/BV1uNk1YxEJQ/?spm_id_from=333.1387.favlist.content.click&vd_source=0eafa421f09152cc7c6c879ff42465e3)
