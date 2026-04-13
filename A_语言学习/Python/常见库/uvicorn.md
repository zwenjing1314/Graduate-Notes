# 介绍

 Uvicorn 是一个快速的 [ASGI](https://zhida.zhihu.com/search?content_id=239758071&content_type=Article&match_order=1&q=ASGI&zhida_source=entity)（Asynchronous Server Gateway Interface）服务器，用于构建异步 Web 服务。它基于 asyncio 库，支持高性能的异步请求处理，适用于各种类型的 Web 应用程序。

# 常见API

## Config

uvicorn.Config 是 Uvicorn ASGI 服务器的配置类，用于封装服务器的所有运行参数。它的工作原理是：

1. 配置阶段：创建 Config 对象时，将所有服务器参数（如主机、端口、日志级别等）封装到一个配置对象中
2. 实例化阶段：将 Config 对象传递给 uvicorn.Server，Server 读取这些配置来初始化自己
3. 运行阶段：调用 server.run() 时，按照配置启动 HTTP 服务器

这种设计模式称为配置与执行分离，好处是可以先配置好参数，再决定何时启动服务器。

代码中的数据流向

```py
# 第1步：创建 FastAPI 应用
app = FastAPI()

# 第2步：创建配置对象，将 app 和服务器参数打包
config = uvicorn.Config(app, host="127.0.0.1", port=8000, log_level="info")
#                    ↓
#            Config 内部存储了：
#            - self.app = FastAPI 实例
#            - self.host = "127.0.0.1"
#            - self.port = 8000
#            - self.log_level = "info"

# 第3步：创建 Server 实例，读取配置
server = uvicorn.Server(config)
#                   ↓
#        Server 从 config 对象中提取所有参数
#        并初始化内部的 socket、logger 等组件

# 第4步：在后台线程启动服务器
thread = threading.Thread(target=server.run, daemon=True)
thread.start()
#              ↓
#    server.run() 根据 config 中的配置：
#    - 绑定到 127.0.0.1:8000
#    - 设置日志级别为 info
#    - 开始监听 HTTP 请求
```

### Config 常用参数详解

1. 必需参数
    app (ASGI application):

  	作用：要运行的 ASGI 应用实例（FastAPI、Starlette 等）
  	类型：ASGI 应用对象
  	示例：app = FastAPI()

2. 网络配置参数
host (str):
         作用：服务器绑定的 IP 地址
         默认值："127.0.0.1"（仅本地访问）
         常用值：
                  "127.0.0.1" - 仅本机可访问（开发环境安全） 
                  "0.0.0.0" - 所有网卡都可访问（允许外部连接）
port (int):
         作用：服务器监听的端口号
         默认值：8000
         范围：0-65535（建议使用 1024 以上的端口）

3. 日志配置参数
log_level (str):
         作用：控制日志输出的详细程度
         可选值（从低到高）：
                  "critical" - 仅严重错误
                  "error" - 错误信息
                  "warning" - 警告信息
                  "info" - 一般信息（默认，显示请求日志）
                  "debug" - 调试信息（最详细）
你的代码中使用 "info"，会输出类似：

```py
    INFO:     Started server process [13224]
    INFO:     Uvicorn running on http://127.0.0.1:8000
    INFO:     127.0.0.1:52748 - "GET /hello HTTP/1.1" 200 OK
```

4. 其他常用参数（你的代码中未使用，但实际开发常用）
	workers (int):
	作用：工作进程数量
	默认值：1
	说明：多进程可以提高并发性能，但在 Jupyter 中通常保持为 1
	reload (bool):
	作用：开启代码热重载（开发模式）
	默认值：False
	说明：代码修改后自动重启服务器，适合开发环境
	root_path (str):
	作用：应用的前缀路径
	示例："/api/v1" - 所有路由会自动加上此前缀
	timeout_keep_alive (int):
	作用：HTTP Keep-Alive 超时时间（秒）
	默认值：5
	limit_concurrency (int):
	作用：最大并发连接数
	默认值：无限制

### 为什么需要 Config 这个中间层？

对比两种写法：
方式1：直接使用 uvicorn.run()（简单但不灵活）

```py
uvicorn.run(app, host="127.0.0.1", port=8000)
# 这会阻塞当前线程，无法在 Jupyter 中后续执行代码
```

方式2：使用 Config + Server（灵活可控）

```py
config = uvicorn.Config(app, host="127.0.0.1", port=8000, log_level="info")
server = uvicorn.Server(config)

# 可以在后台线程启动
thread = threading.Thread(target=server.run, daemon=True)
thread.start()

# 可以随时停止
server.should_exit = True
thread.join()
```

优势：
✅ 可以在 Jupyter Notebook 中非阻塞运行
✅ 可以精确控制服务器的启动和停止时机
✅ 可以先配置多个不同的 Config，根据需要选择启动哪个
✅ 便于单元测试和程序化控制

# Uvicorn 的工作原理详解

## Uvicorn 在系统中的完整角色

1. 架构层次图

```py
┌─────────────────────────────────────────────────┐
│              你的 FastAPI 应用                   │
│         (业务逻辑、路由、数据处理)                │
└──────────────────┬──────────────────────────────┘
                   │ ASGI 接口
┌──────────────────▼──────────────────────────────┐
│              Uvicorn Server                      │
│         (HTTP 服务器 / Web 服务器)               │
│                                                  │
│  - 监听网络端口                                  │
│  - 接收 HTTP 请求                                │
│  - 解析 HTTP 协议                                │
│  - 调用 FastAPI 应用                             │
│  - 发送 HTTP 响应                                │
└──────────────────┬──────────────────────────────┘
                   │ TCP/IP Socket
┌──────────────────▼──────────────────────────────┐
│           操作系统网络栈                         │
│      (TCP/IP 协议栈、Socket API)                 │
└──────────────────┬──────────────────────────────┘
                   │ 网络接口
┌──────────────────▼──────────────────────────────┐
│           物理/虚拟网络层                        │
│      (网卡、回环接口 lo、TCP/IP 协议)            │
└─────────────────────────────────────────────────┘
```

## Uvicorn 的工作原理详解

从客户端发起请求到收到响应的完整流程
让我用你的代码作为例子，追踪一个完整的请求-响应周期：

```py
# 你的代码
config = uvicorn.Config(app, host="127.0.0.1", port=8000, log_level="info")
server = uvicorn.Server(config)
thread = threading.Thread(target=server.run, daemon=True)
thread.start()
```

阶段1：服务器启动（监听阶段）

```py
执行 server.run() 后的内部过程：

┌──────────────────────────────────────────────────┐
│          Uvicorn Server 启动流程                  │
└──────────────────────────────────────────────────┘

第1步：创建 Socket
    ↓
    import socket
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # AF_INET: 使用 IPv4
    # SOCK_STREAM: 使用 TCP 协议
    
第2步：绑定地址和端口
    ↓
    sock.bind(("127.0.0.1", 8000))
    # 告诉操作系统：
    # "我要监听 127.0.0.1 这个 IP 的 8000 端口"
    
第3步：开始监听
    ↓
    sock.listen(1024)
    # 1024: 最大等待队列长度
    # 此时 Socket 进入 LISTEN 状态
    # 操作系统内核开始接收发往 127.0.0.1:8000 的连接
    
第4步：进入事件循环
    ↓
    while True:
        # 异步等待新连接
        connection, client_address = await accept_connection(sock)
        # 当有客户端连接时，创建一个新的连接 Socket
        
        # 在新任务中处理这个连接
        asyncio.create_task(handle_connection(connection))
```

阶段1：服务器启动（监听阶段）

```
执行 server.run() 后的内部过程：

┌──────────────────────────────────────────────────┐
│          Uvicorn Server 启动流程                  │
└──────────────────────────────────────────────────┘

第1步：创建 Socket
    ↓
    import socket
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    # AF_INET: 使用 IPv4
    # SOCK_STREAM: 使用 TCP 协议
    
第2步：绑定地址和端口
    ↓
    sock.bind(("127.0.0.1", 8000))
    # 告诉操作系统：
    # "我要监听 127.0.0.1 这个 IP 的 8000 端口"
    
第3步：开始监听
    ↓
    sock.listen(1024)
    # 1024: 最大等待队列长度
    # 此时 Socket 进入 LISTEN 状态
    # 操作系统内核开始接收发往 127.0.0.1:8000 的连接
    
第4步：进入事件循环
    ↓
    while True:
        # 异步等待新连接
        connection, client_address = await accept_connection(sock)
        # 当有客户端连接时，创建一个新的连接 Socket
        
        # 在新任务中处理这个连接
        asyncio.create_task(handle_connection(connection))
```

此时的网络状态：

```py
# 在终端运行 netstat 可以看到：
$ netstat -tlnp | grep 8000
tcp   0   0 127.0.0.1:8000   0.0.0.0:*   LISTEN   12345/python

# 解释：
# - 协议：TCP
# - 本地地址：127.0.0.1:8000
# - 状态：LISTEN（正在监听）
# - 进程：Python (Uvicorn)
```

阶段2：接收请求（连接建立）
假设浏览器访问 http://127.0.0.1:8000/news/1

```py
客户端（浏览器）                    Uvicorn 服务器
─────────────                    ───────────────
                                    
1. DNS 解析
   127.0.0.1 → 本地回环地址
       ↓
2. 创建 Socket
   client_sock = socket()
       ↓
3. 发起 TCP 三次握手
   SYN ──────────────────────────→
       ←────────────────────────── SYN-ACK
   ACK ──────────────────────────→
       ↓
4. TCP 连接建立成功
   状态：ESTABLISHED
       
5. 发送 HTTP 请求报文
   GET /news/1 HTTP/1.1\r\n
   Host: 127.0.0.1:8000\r\n
   User-Agent: Mozilla/5.0\r\n
   Accept: */*\r\n
   \r\n
       ↓
   （数据通过 TCP 分段传输）
       ↓
6. Uvicorn 接收数据
   data = await connection.recv(4096)
       ↓
7. 解析 HTTP 请求
   parse_http_request(data)
   得到：
   {
     'method': 'GET',
     'path': '/news/1',
     'headers': {...},
     'version': 'HTTP/1.1'
   }
```

TCP 三次握手的详细过程：

```
浏览器（客户端）                    Uvicorn（服务器）
─────────────                    ───────────────
                                    
CLOSED                           LISTEN
   │                                  │
   │  SYN (seq=x)                    │
   │────────────────────────────────>│
   │                              SYN-RECEIVED
   │                                  │
   │  SYN-ACK (seq=y, ack=x+1)       │
   │<────────────────────────────────│
   │                              ESTABLISHED
   │                                  │
   │  ACK (ack=y+1)                  │
   │────────────────────────────────>│
   │                                  │
ESTABLISHED                       
   │                                  │
   │  可以开始传输数据                │
   │<══════════════════════════════>│
```

阶段3：处理请求（应用层）

```py
Uvicorn 接收到 HTTP 请求后：

第1步：构建 ASGI scope（作用域对象）
    ↓
    scope = {
        'type': 'http',
        'asgi': {'version': '3.0'},
        'http_version': '1.1',
        'method': 'GET',
        'path': '/news/1',
        'query_string': b'',
        'headers': [
            (b'host', b'127.0.0.1:8000'),
            (b'user-agent', b'Mozilla/5.0'),
            ...
        ],
        'server': ('127.0.0.1', 8000),
        'client': ('127.0.0.1', 54321),  # 客户端端口
    }
    
第2步：创建 ASGI receive 和 send 函数
    ↓
    async def receive():
        """从客户端接收更多数据"""
        return await connection.recv()
    
    async def send(message):
        """向客户端发送数据"""
        await connection.send(message)
    
第3步：调用 FastAPI 应用（ASGI 应用）
    ↓
    await app(scope, receive, send)
    # FastAPI 开始处理：
    # 1. 路由匹配：找到 @app.get("/news/{id}")
    # 2. 参数提取：id = 1
    # 3. 执行函数：result = await get_news(id=1)
    # 4. 构建响应：response = {"id": 1, "title": "..."}
    
第4步：FastAPI 通过 send() 发送响应
    ↓
    # FastAPI 内部调用：
    await send({
        'type': 'http.response.start',
        'status': 200,
        'headers': [
            (b'content-type', b'application/json'),
            (b'content-length', b'50'),
        ],
    })
    
    await send({
        'type': 'http.response.body',
        'body': b'{"id":1,"title":"this is 1","content":"hello world"}',
    })
```

阶段4：发送响应（返回数据）

```py
Uvicorn 收到 FastAPI 的响应后：

第1步：构建 HTTP 响应报文
    ↓
    response_bytes = (
        b"HTTP/1.1 200 OK\r\n"
        b"content-type: application/json\r\n"
        b"content-length: 50\r\n"
        b"\r\n"
        b'{"id":1,"title":"this is 1","content":"hello world"}'
    )
    
第2步：通过 Socket 发送
    ↓
    await connection.sendall(response_bytes)
    # 操作系统将数据分成 TCP 段
    # 通过网络栈发送
    
第3步：客户端接收
    ↓
    浏览器收到响应报文
    解析 HTTP 头
    解析 JSON  body
    显示结果
    
第4步：关闭连接（或保持长连接）
    ↓
    # HTTP/1.1 默认使用 Keep-Alive
    # 连接可以复用于下一个请求
```

