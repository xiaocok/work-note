



### MCP测试

### 代码

```python
import asyncio
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

# 创建FastAPI应用
fastapi_app = FastAPI(title="MCP Server")

# 配置CORS - 这是关键！
fastapi_app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "*",  # 生产环境不要用这个，应该指定具体域名
        "http://localhost:3000",
        "http://127.0.0.1:3000",
    ],
    allow_credentials=True,
    allow_methods=["*"],  # 允许所有HTTP方法
    allow_headers=["*"],  # 允许所有请求头
)

# 注册你的MCP工具
@fastapi_app.get("/health")
async def health_check():
    return {"status": "healthy"}

@fastapi_app.post("/chat")
async def chat_endpoint(prompt: dict):
    # 这里处理你的MCP逻辑
    return {"response": f"Echo: {prompt.get('message', '')}"}

if __name__ == "__main__":
    # 方式1：使用uvicorn直接运行
    import uvicorn
    uvicorn.run(
        fastapi_app,
        host="0.0.0.0",  # 允许所有网络访问
        port=8000
    )
```

### 依赖

```shell
# 创建虚拟环境
python -m venv venv

# 激活
# Windows
venv\Scripts\activate
# Linux/Mac
source venv/bin/activate

# 安装依赖
pip install fastapi uvicorn[standard]
```

### 运行

```shell
# 方式1：使用python运行
python deepseek_http_restful.py
 
# 方式2：直接使用uvicorn命令行
# deepseek_http_restful:fastapi_app -> 模块文件:应用对象 或 package.module:app
uvicorn deepseek_http_restful:fastapi_app --host 0.0.0.0 --port 8000
```

### 测试

```shell
# 访问方式
# 1. Web浏览器访问
# 假设你的MCP应用运行在 http://localhost:8000
# 在浏览器中访问：
http://localhost:8000/docs   # 查看Swagger/OpenAPI文档
http://localhost:8000/redoc  # 查看ReDoc文档（如果有的话）
 
# 2. Postman访问

# 3. curl命令行
http://127.0.0.1:8000/health

curl -X POST http://localhost:8000/chat \
   -H "Content-Type: application/json" \
   -d '{"message": "Hello, world!"}'
```

