# 介绍

FastAPI 是一个现代化、高性能的 Python Web 框架，专注于快速开发和生产就绪的 API。

高性能与标准支持

FastAPI 基于 **Starlette** 和 **Pydantic**，提供了接近 NodeJS 和 Go 的性能。它完全兼容 **OpenAPI** 和 **JSON Schema** 标准，支持自动生成交互式 API 文档（如 Swagger UI 和 ReDoc），方便开发者快速测试和集成。

类型提示与数据验证

FastAPI 使用 Python 的类型提示（Type Hints）来定义请求和响应数据模型。通过 **Pydantic**，它能够自动验证复杂的嵌套数据结构，包括字符串、数字、URL、Email 等类型，并在数据无效时提供清晰的错误提示。

```py
from fastapi import FastAPI
from pydantic import BaseModel
app = FastAPI()
class Item(BaseModel):
   name: str
   price: float
   is_offer: bool = False
@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
   return {"item_id": item_id, "item_name": item.name}py
```

依赖注入系统

FastAPI 提供了强大的依赖注入机制，通过 *Depends* 可以轻松实现模块化和可复用的代码。它支持复杂的依赖关系，例如数据库连接、身份验证等。

```py
from fastapi import Depends, FastAPI
app = FastAPI()
def common_parameters(q: str = None, skip: int = 0, limit: int = 10):
   return {"q": q, "skip": skip, "limit": limit}
@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
   return commons
```

安全性与认证

FastAPI 内置支持多种认证方式，包括 **OAuth2**、JWT、API 密钥等。它还集成了 **Starlette** 的安全特性，如会话管理和跨域资源共享（CORS）。

```py
from fastapi import Depends, FastAPI, HTTPException
from fastapi.security import OAuth2PasswordBearer
app = FastAPI()
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")
@app.get("/users/me")
async def read_users_me(token: str = Depends(oauth2_scheme)):
   if token != "valid-token":
       raise HTTPException(status_code=401, detail="Invalid token")
   return {"username": "user"}
```

异步编程与性能优化

FastAPI 完全支持 Python 的异步特性，能够高效处理并发请求。通过 *asyncio*，可以轻松实现异步任务和性能优化。

```py
import asyncio
from fastapi import FastAPI
app = FastAPI()
async def task1():
   await asyncio.sleep(1)
   return "Task 1 completed"
async def task2():
   await asyncio.sleep(2)
   return "Task 2 completed"
@app.get("/tasks/")
async def run_tasks():
   results = await asyncio.gather(task1(), task2())
   return results
```

数据库集成与操作

FastAPI 可以轻松集成主流数据库（如 MySQL、PostgreSQL）。通过 SQLAlchemy 或其他 ORM 工具，可以快速实现数据库操作。

```py
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Base = declarative_base()
class User(Base):
   __tablename__ = "users"
   id = Column(Integer, primary_key=True, index=True)
   name = Column(String, index=True)
Base.metadata.create_all(bind=engine)
```

自动化文档与调试

FastAPI 自动生成交互式 API 文档，开发者可以通过 */docs* 和 */redoc* 路径访问。它还支持使用 **pytest** 进行单元测试，确保代码的稳定性。

部署与生产就绪

FastAPI 支持使用 **Uvicorn** 或 **Gunicorn** 部署，结合 Docker 和 CI/CD 工具（如 GitHub Actions），可以快速实现生产环境的部署。

```
FROM python:3.10-slim
WORKDIR /app
COPY requirements.txt
RUN pip install --no-cache-dir -r requirements.txt
COPY .
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

总结

FastAPI 是一个功能强大、易于使用的框架，适合构建高性能、可维护的 API 服务。通过其丰富的特性和生态支持，开发者可以快速实现从开发到生产的全流程。

# 装饰器

## @app.get()

用于**获取资源**，通常执行查询操作，不应对服务器数据产生副作用。

```py
@app.get("/")
def index() -> FileResponse:
    """返回上传页面。"""
    index_file = WEB_DIR / "index.html"
    if not index_file.exists():
        raise HTTPException(status_code=500, detail="缺少 web/index.html 文件。")
    return FileResponse(index_file)
```

1. 路由注册机制

```py
@app.get("/")          # 将 index 函数注册为 GET / 的处理器
```

- @app.get("/") 本质上是一个装饰器工厂，返回一个装饰器函数
- 当 Python 解释器执行到被装饰的函数时，装饰器会：
  - 记录函数的元信息（参数、类型注解、文档字符串）
  - 将 URL 路径 / 与函数 index 建立映射关系
  - 将 HTTP 方法 GET 与该路由绑定
  - 将路由注册到 FastAPI 内部的路由表中

等价的手动注册方式：

```py
def index() -> FileResponse:
    ...

# 不使用装饰器的等价写法
app.add_api_route("/", index, methods=["GET"])
```

## @app.post()

用于**提交数据**，常用于新增、修改或删除资源（实际开发中删除也常用 POST 实现）

装饰器不仅注册路由，还触发了 FastAPI 的依赖注入机制：

```py
@app.post("/ocr")
def upload_pdf(
    file: UploadFile = File(...),           # ← 依赖声明
    ocr_lang: str = Form("eng"),            # ← 依赖声明
):
```

设计机制：

* introspecton（内省）：装饰器通过 inspect 模块读取函数的签名
* 类型提取：从类型注解（如 UploadFile, str）识别参数类型
* 默认值解析：从默认值（如 File(...), Form(...)）识别参数来源
* 依赖图构建：在应用启动时构建依赖关系图
* 运行时注入：请求到达时，按依赖图顺序解析并注入参数

装饰器模式 + 注册表模式

```py
# 简化的内部实现逻辑
class FastAPI:
    def __init__(self):
        self.routes = {}  # 路由注册表
    
    def get(self, path: str):
        """装饰器工厂"""
        def decorator(func):
            # 注册路由
            self.routes[(path, "GET")] = {
                "endpoint": func,                    # 处理函数
                "signature": inspect.signature(func), # 函数签名
                "annotations": func.__annotations__,  # 类型注解
            }
            return func  # 返回原函数，不改变其行为
        return decorator
```

请求分发流程

```
客户端请求 GET /
       ↓
FastAPI 接收请求
       ↓
查询路由表 routes[("/", "GET")]
       ↓
找到 index 函数
       ↓
检查依赖项（此例无依赖）
       ↓
调用 index()
       ↓
获取返回值 FileResponse
       ↓
序列化为 HTTP 响应
       ↓
返回给客户端
```

声明式编程

```py
# 声明式：清晰表达意图
@app.post("/ocr")
def upload_pdf(file: UploadFile = File(...)):
    ...

# 对比命令式：需要手动配置
def upload_pdf(file):
    ...
app.add_route("/ocr", upload_pdf, methods=["POST"])
validate_file_parameter(upload_pdf)
```



# 常见API

## FastAPI()

作用：Web 框架主类，用于构建 RESTful API
功能：提供路由装饰器（如 @app.post）、自动请求验证、自动生成文档等

```py
from fastapi import FastAPI

app = FastAPI(
    title="OCR Inspector",
    description="上传 PDF，输出 ocr.json、叠框图和纯文本。",
    version="1.0.0",
)
```

### 属性和函数

#### add_middleware()

从源码层面深入分析 add_middleware 和 CORSMiddleware 的工作机制。

##### 源码层级分析

1. 方法调用链

```py
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

这段代码的执行流程涉及多个层次：
第一层：FastAPI.add_middleware()
FastAPI 继承自 Starlette，这个方法定义在 Starlette 的 applications.py 中：

```py
# Starlette 源码简化版
class Starlette:
    def __init__(self):
        self.user_middleware = []  # 中间件配置列表
        self.middleware_stack = None  # 最终构建的中间件链
    
    def add_middleware(
        self,
        middleware_class: type,  # CORSMiddleware 类
        **options: typing.Any    # allow_origins, allow_credentials 等参数
    ) -> None:
        # 1. 将中间件配置保存到列表
        self.user_middleware.insert(0, Middleware(
            middleware_class, 
            **options
        ))
        
        # 2. 重新构建中间件链
        self.middleware_stack = self.build_middleware_stack()
```

关键设计点：

- insert(0, ...) - 新中间件插入到列表开头，保证后添加的先执行（LIFO）
- Middleware 是一个数据类，存储类和参数的映射关系
- 每次添加都会重建整个中间件链

第二层：Middleware 数据类

```py
# Starlette 源码简化版
class Middleware:
    def __init__(self, cls: type, **options: typing.Any):
        self.cls = cls          # CORSMiddleware
        self.options = options  # {allow_origins: ["*"], ...}
    
    def __call__(self, app: ASGIApp) -> ASGIApp:
        # 实例化中间件，传入下一个 app
        return self.cls(app, **self.options)
```

2. 中间件链构建机制

```py
def build_middleware_stack(self) -> ASGIApp:
    """构建中间件栈"""
    # 从内到外：app -> 中间件N -> ... -> 中间件1
    handler = self.router  # 最内层是路由处理器
    
    for middleware in self.user_middleware:
        # 包装：每个中间件包裹当前的 handler
        handler = middleware(handler)
    
    return handler
```

实际构建过程（以你的代码为例）：

```py
初始状态：
handler = router (路由处理器)

添加 CORSMiddleware 后：
handler = CORSMiddleware(router, allow_origins=["*"], ...)

如果再加一个中间件 AuthMiddleware：
handler = AuthMiddleware(
    CORSMiddleware(router, ...),
    ...
)
```

数据结构示意

```
请求进入 → AuthMiddleware
              ↓ (验证通过)
         CORSMiddleware
              ↓ (检查跨域)
           Router (路由分发)
              ↓
         业务逻辑处理
```

3. CORSMiddleware 内部实现

```py
# Starlette 源码简化版
class CORSMiddleware:
    def __init__(
        self,
        app: ASGIApp,           # 下一个 ASGI 应用（可能是另一个中间件或 router）
        allow_origins: list[str],
        allow_credentials: bool,
        allow_methods: list[str],
        allow_headers: list[str],
    ):
        self.app = app
        self.allow_origins = allow_origins
        self.allow_credentials = allow_credentials
        self.allow_methods = allow_methods
        self.allow_headers = allow_headers
    
    async def __call__(self, scope, receive, send):
        """ASGI 协议入口"""
        if scope["type"] != "http":
            await self.app(scope, receive, send)
            return
        
        method = scope.get("method", "").upper()
        
        # === 预检请求处理 (OPTIONS) ===
        if method == "OPTIONS":
            response = self._preflight_request(scope)
            await response(scope, receive, send)
            return
        
        # === 实际请求处理 ===
        # 1. 调用下一个中间件/应用
        simple_headers = {}
        if self.allow_origins == ["*"]:
            simple_headers["Access-Control-Allow-Origin"] = "*"
        
        # 创建包装的 send 函数，在响应中添加 CORS 头
        async def send_wrapper(message):
            if message["type"] == "http.response.start":
                headers = dict(message["headers"])
                # 添加 CORS 响应头
                for key, value in simple_headers.items():
                    headers[key.encode()] = value.encode()
                message["headers"] = list(headers.items())
            await send(message)
        
        # 2. 继续处理链
        await self.app(scope, receive, send_wrapper)
    
    def _preflight_request(self, scope):
        """处理 OPTIONS 预检请求"""
        headers = {
            "Access-Control-Allow-Origin": self._get_allow_origin(scope),
            "Access-Control-Allow-Methods": ", ".join(self.allow_methods),
            "Access-Control-Allow-Headers": ", ".join(self.allow_headers),
        }
        
        if self.allow_credentials:
            headers["Access-Control-Allow-Credentials"] = "true"
        
        return Response(
            status_code=200,
            headers=headers,
        )
```

4. ASGI 协议层面的工作原理

```py
# ASGI 应用签名
async def app(scope: dict, receive: callable, send: callable):
    """
    scope: 请求上下文（包含 method, path, headers 等）
    receive: 接收消息的协程函数
    send: 发送消息的协程函数
    """
```

请求流转过程：

```py
浏览器发起跨域请求
       ↓
[ASGI Server (Uvicorn)] 接收原始 TCP 连接
       ↓
解析为 ASGI scope 格式
       ↓
┌──────────────────────────────┐
│ CORSMiddleware.__call__      │ ← 第45行添加的中间件
│                              │
│ 1. 检查是否为 OPTIONS 预检   │
│ 2. 如果是预检：               │
│    - 直接返回 200 + CORS 头  │
│ 3. 如果是实际请求：           │
│    - 包装 send 函数          │
│    - 调用 self.app(...)      │ → 转发给下一层
└──────────────────────────────┘
       ↓
┌──────────────────────────────┐
│ FastAPI Router               │
│                              │
│ 1. 匹配路由 /ocr             │
│ 2. 解析参数                   │
│ 3. 调用 upload_pdf 函数      │
└──────────────────────────────┘
       ↓
生成响应
       ↓
沿原路返回，经过 CORSMiddleware
       ↓
CORSMiddleware 在响应头中添加：
  Access-Control-Allow-Origin: *
  Access-Control-Allow-Methods: *
  Access-Control-Allow-Headers: *
       ↓
返回给浏览器
```

5. 关键设计模式

责任链模式 (Chain of Responsibility)

```py
# 每个中间件只关心自己的逻辑，然后传递给下一个
class Middleware:
    async def __call__(self, scope, receive, send):
        # 前置处理
        print("Before request")
        
        # 传递给下一个
        await self.next_app(scope, receive, send)
        
        # 后置处理
        print("After response")
```

装饰器模式的变体

```py
# 中间件本质上是在运行时动态包装 ASGI 应用
original_app = router
wrapped_app = CORSMiddleware(original_app, ...)
# wrapped_app 具有和 original_app 相同的接口，但增加了功能
```

6. 你的配置的具体含义

```py
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],        # 允许所有域名访问
    allow_credentials=True,     # 允许携带 Cookie/认证信息
    allow_methods=["*"],        # 允许所有 HTTP 方法 (GET, POST, PUT, DELETE...)
    allow_headers=["*"],        # 允许所有自定义请求头
)
```

生成的响应头示例：

```py
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: GET, POST, PUT, DELETE, OPTIONS
Access-Control-Allow-Headers: *
Access-Control-Allow-Credentials: true
```

#### mount() 

FastAPI 继承自 Starlette，mount 方法定义在 Starlette 的 applications.py 中：

```py
# Starlette 源码简化版
class Starlette:
    def __init__(self):
        self.routes = []  # 路由列表
    
    def mount(
        self, 
        path: str,              # "/outputs"
        app: ASGIApp,           # StaticFiles 实例
        name: str | None = None # "outputs"
    ) -> None:
        """挂载子应用"""
        route = Mount(path, app=app, name=name)
        self.routes.append(route)
```

关键点：

- mount 不是注册普通路由，而是创建 Mount 对象
- Mount 是一种特殊的路由类型，可以包含自己的子路由
- 挂载的应用（StaticFiles）拥有独立的路由空间

#####  Mount 类的内部实现

```py
# Starlette 源码简化版
class Mount(BaseRoute):
    def __init__(
        self, 
        path: str,      # "/outputs"
        app: ASGIApp,   # StaticFiles 实例
        name: str | None = None
    ):
        self.path = path.rstrip("/")  # "/outputs"
        self.app = app                # StaticFiles(...)
        self.name = name              # "outputs"
    
    async def __call__(self, scope, receive, send):
        """当请求匹配到此 Mount 路由时调用"""
        # 1. 检查路径是否匹配
        if not scope["path"].startswith(self.path):
            await self._not_found(scope, receive, send)
            return
        
        # 2. 路径重写：移除挂载前缀
        # 例如：/outputs/5ad243864b2e/ocr.json 
        # 变成：/5ad243864b2e/ocr.json
        root_path = scope.get("root_path", "")
        scope["root_path"] = root_path + self.path
        
        remaining_path = scope["path"][len(self.path):]
        scope["path"] = remaining_path or "/"
        
        # 3. 调用子应用（StaticFiles）
        await self.app(scope, receive, send)
    
    def matches(self, scope):
        """检查请求是否匹配此挂载点"""
        if scope["type"] == "http":
            if scope["path"].startswith(self.path):
                return Match.FULL
        return Match.NONE
```

核心机制 - 路径重写：

```py
原始请求: GET /outputs/5ad243864b2e/ocr.json
          ↓
Mount 拦截，检查路径前缀是否为 /outputs
          ↓
路径重写:
  scope["path"] = "/5ad243864b2e/ocr.json"  (去掉 /outputs)
  scope["root_path"] = "/outputs"            (记录前缀)
          ↓
转发给 StaticFiles 处理
```

##### StaticFiles 类的完整实现

```py
# Starlette 源码简化版
class StaticFiles:
    def __init__(
        self,
        directory: str | Path | None = None,  # outputs 目录路径
        packages: list[str | tuple[str, str]] | None = None,
        html: bool = False,
        check_dir: bool = True,
        follow_symlink: bool = False,
    ):
        # 1. 验证目录存在
        if check_dir and directory is not None:
            directory = Path(directory).resolve()
            if not directory.is_dir():
                raise RuntimeError(f"Directory '{directory}' does not exist")
        
        self.directory = Path(directory)
        self.html = html
        self.follow_symlink = follow_symlink
        
        # 2. 构建文件索引（可选优化）
        self.all_directories = [self.directory]
    
    async def __call__(self, scope, receive, send):
        """ASGI 应用入口"""
        if scope["type"] != "http":
            await self._not_found(scope, receive, send)
            return
        
        # 1. 从路径中提取文件名
        path = scope["path"]
        if path.startswith("/"):
            path = path[1:]  # 去掉开头的 /
        
        # 2. 安全检查：防止目录遍历攻击
        if not self._is_safe_path(path):
            await self._forbidden(scope, receive, send)
            return
        
        # 3. 构建完整文件路径
        full_path = self.directory / path
        
        # 4. 检查文件是否存在
        if not full_path.is_file():
            await self._not_found(scope, receive, send)
            return
        
        # 5. 获取文件信息
        stat_result = await anyio.to_thread.run_sync(os.stat, full_path)
        
        # 6. 设置响应头
        headers = [
            (b"content-type", self._get_content_type(full_path)),
            (b"content-length", str(stat_result.st_size).encode()),
            (b"last-modified", http_date(stat_result.st_mtime)),
            (b"etag", self._generate_etag(stat_result)),
        ]
        
        # 7. 处理 If-None-Match (缓存协商)
        if self._check_cache(scope, headers):
            await self._not_modified(scope, receive, send)
            return
        
        # 8. 发送文件
        await self._send_file(scope, receive, send, full_path, headers)
    
    def _is_safe_path(self, path: str) -> bool:
        """防止目录遍历攻击"""
        # 拒绝包含 .. 的路径
        if ".." in path.split("/"):
            return False
        
        # 解析后的路径必须在允许目录内
        resolved = (self.directory / path).resolve()
        return str(resolved).startswith(str(self.directory))
    
    def _get_content_type(self, filepath: Path) -> str:
        """根据文件扩展名推断 MIME 类型"""
        import mimetypes
        content_type, _ = mimetypes.guess_type(str(filepath))
        return content_type or "application/octet-stream"
    
    async def _send_file(
        self, scope, receive, send, 
        filepath: Path, headers: list
    ):
        """高效发送文件（支持大文件）"""
        # 发送响应头
        await send({
            "type": "http.response.start",
            "status": 200,
            "headers": headers,
        })
        
        # 分块读取文件，避免一次性加载到内存
        async with await anyio.open_file(filepath, "rb") as f:
            while True:
                chunk = await f.read(64 * 1024)  # 64KB 分块
                if not chunk:
                    break
                await send({
                    "type": "http.response.body",
                    "body": chunk,
                    "more_body": True,
                })
        
        # 发送结束标记
        await send({
            "type": "http.response.body",
            "body": b"",
            "more_body": False,
        })
```

##### 完整的请求处理流程

以访问 /outputs/5ad243864b2e/ocr.json 为例：

```py
┌─────────────────────────────────────────┐
│ 1. 浏览器发起请求                        │
│    GET /outputs/5ad243864b2e/ocr.json   │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 2. Uvicorn (ASGI Server)                │
│    - 接收 TCP 连接                       │
│    - 解析 HTTP 请求                      │
│    - 构建 ASGI scope                     │
│      {                                   │
│        "type": "http",                   │
│        "method": "GET",                  │
│        "path": "/outputs/5ad243864b2e/ocr.json",
│        "headers": [...],                 │
│      }                                   │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 3. FastAPI Router                       │
│    - 遍历所有路由，寻找匹配项             │
│    - 发现 Mount("/outputs", ...) 匹配   │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 4. Mount.__call__()                     │
│    - 检查路径前缀: /outputs ✓           │
│    - 路径重写:                           │
│      scope["path"] = "/5ad243864b2e/ocr.json"
│      scope["root_path"] = "/outputs"    │
│    - 调用 self.app (StaticFiles)        │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 5. StaticFiles.__call__()               │
│    - 提取路径: "5ad243864b2e/ocr.json"  │
│    - 安全检查: 无 ".." ✓                │
│    - 构建完整路径:                       │
│      /home/.../outputs/5ad243864b2e/ocr.json
│    - 检查文件存在性 ✓                    │
│    - 获取文件状态 (stat)                 │
│    - 推断 MIME 类型: application/json   │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 6. 发送响应                              │
│    http.response.start:                 │
│      status: 200                         │
│      headers:                            │
│        content-type: application/json    │
│        content-length: 12345             │
│        last-modified: Wed, 08 Apr ...    │
│        etag: "abc123"                    │
│                                         │
│    http.response.body (分块发送):        │
│      chunk1: 64KB                        │
│      chunk2: 64KB                        │
│      ...                                 │
│      final: "" (结束标记)                │
└─────────────────────────────────────────┘
              ↓
┌─────────────────────────────────────────┐
│ 7. 浏览器接收并解析 JSON                 │
└─────────────────────────────────────────┘
```

##### 与普通路由的区别

```py
# 方式1: mount 挂载（你的代码）
app.mount("/outputs", StaticFiles(directory=OUTPUTS_DIR))

# 方式2: 手动编写路由（等价但繁琐）
@app.get("/outputs/{filepath:path}")
def serve_file(filepath: str):
    file_path = OUTPUTS_DIR / filepath
    if not file_path.is_file():
        raise HTTPException(404)
    return FileResponse(file_path)
```



## File()

作用：声明参数来自上传的文件
原理：告诉 FastAPI 从 HTTP 请求的 multipart/form-data 中读取文件数据
数据流：客户端上传文件 → FastAPI 接收二进制流 → 转换为 UploadFile 对象

```py
from fastapi import File

file: UploadFile = File(...)
```

## Form()

作用：声明参数来自表单字段
原理：从 HTTP 请求的 form data 中提取普通字段值
数据流：客户端发送表单数据 → FastAPI 解析键值对 → 自动类型转换并赋值

```py
from fastapi import Form

ocr_lang: str = Form(os.getenv("OCR_LANG", "eng"))
```

## HTTPException()

作用：抛出 HTTP 错误响应
使用场景：当请求无效时，返回特定状态码和错误信息给客户端

```py
from fastapi import HTTPException

raise HTTPException(status_code=500, detail="缺少 web/index.html 文件。")
```

## UploadFile()

作用：表示上传的文件对象
包含属性：
filename: 原始文件名
file: 实际的文件对象（支持异步读取）
content_type: MIME 类型

```py

from fastapi import UploadFile

file: UploadFile = File(...)
```

