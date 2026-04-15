# 介绍

可以。**一段话总结**：FastAPI 是一个用 **Python 类型注解**来定义接口的现代 API 框架，主要用来快速开发 Web API、后端服务和微服务；它的核心特点是 **性能高**、**异步支持好**、**请求参数和数据模型自动校验**、**自动生成 OpenAPI/Swagger 文档**，还有比较顺手的 **依赖注入** 机制。官方把它定位成“modern, fast (high-performance)”，底层依赖 Starlette 和 Pydantic，所以在写接口时通常比 Flask 更省样板代码、比 Django/DRF 更轻更偏 API-first，尤其适合做前后端分离后端、数据服务、AI 服务接口这类场景；但如果你要的是很重的“全家桶”能力，比如成熟后台、ORM、模板渲染和传统网站开发，Django 往往还是更完整。

# 使用学习记录

## 启动fastapi服务

```bash
uvicorn main:app --reload

# mian 后面的是FastAPI对象名， --reload 保证在修改代码时，浏览器页面动态生效
```

## 访问FastAPI交互式文档

在访问网址后面加上 docs

如 [http://127.0.0.1:8000/docs](http://127.0.0.1:8000/docs)

## 创建 FastAPI 项目和纯 python 项目的区别

**纯Python项目**：它的目标通常是执行一个具体的、一次性的任务。运行一个纯Python脚本就像是完成一道独立的计算题，任务结束后程序便自动退出。

**FastAPI项目**：它是一个需要持续运行的Web应用程序。运行时，它会启动一个Web服务器（如Uvicorn），并保持持续监听状态，以接收和处理外部的HTTP请求。因此，你需要手动停止它，否则它会一直运行。

1**. 运行纯Python项目**

当你在一个普通Python文件中点击运行按钮时，PyCharm会创建一个临时的“Python”运行配置。其行为是：

- **脚本目标**：直接执行你当前的 `.py` 文件。
- **运行结果**：如果脚本中没有 `print()` 等显式输出语句，运行后你将**看不到任何内容**。此时PyCharm控制台只会显示一条信息：`Process finished with exit code 0`。
- **成功标志**：这里的 `exit code 0` 是程序执行成功并正常退出的**标准信号**，它表示程序没有遇到错误。

**2. 运行FastAPI项目**

当你新建FastAPI项目时，PyCharm专业版会自动为你创建一个预制的“FastAPI”运行配置。它的设计完全不同：

- **应用目标**：它会自动检测项目中的FastAPI应用实例（通常是 `main.py` 里的 `app` 对象），并直接调用 `uvicorn` 服务器来运行它。
- **服务器信息输出**：一旦运行，控制台就会输出清晰的服务器信息：
  - **启动提示**：例如 `INFO: Started server process [12345]`。
  - **监听地址**：会明确显示 `Uvicorn running on http://127.0.0.1:8000`。这意味着你的API服务已经在 `8000` 端口上等待访问了。
  - **持续运行**：程序会一直处于运行状态，直到你手动点击停止按钮。

> **💡 补充说明**：虽然控制台不显示输出，但代码里定义的接口（如 `@app.get("/")`）已生效。你可以直接在浏览器里访问 `http://127.0.0.1:8000` 来验证。



# ORM 学习

## ORM 简介

ORM（Object-RelationalMapping，对象关系映射）是一种编程技术，用于在面向对象编程语言和关系型数据库之间
建立映射。它允许开发者通过操作对象的方式与数据库进行交互，而无需直接编写复杂的SQL语句。

Version:0.9 StartHTML:0000000105 EndHTML:0000001318 StartFragment:0000000141 EndFragment:0000001278

优势：

 减少重复的 SQL 代码

 代码更简洁易读

 自动处理数据库连接和事务

 自动防止 SQL 注入攻击

## ORM使用流程

排名 	ORM 工具			 特点 				适应场景

1 	SQLAlchemy ORM 	功能最强、最灵活、企业级 	各类 API、微服务、数据应用

2 	Django ORM 			封装好、上手快 			Django 项目、管理后台

3	 Tortoise ORM 				全异步 				异步 Web 服务、高并发 API

sqlalchemy (ORM 工具)

aiomysql（异步数据库驱动）

Ubuntu 安装

```bash
pip install sqlalchemy aiomysql
```

Mac 安装

```
pip install "sqlalchemy[asyncio]" aiomysql
```



# Uvicorn 高性能服务器

**Uvicorn** 是一个基于 **uvloop** 和 **httptools** 构建的超高速 **ASGI**（Asynchronous Server Gateway Interface）服务器，专为 Python 异步 Web 应用设计。它支持 **HTTP/1.1**、**WebSocket**，并计划支持 **HTTP/2**，可与 **FastAPI**、**Starlette**、**Django Channels** 等框架无缝配合。

**核心特性**

- **高性能**：*uvloop* 替代默认事件循环，性能提升 2~4 倍。
- **协议支持**：HTTP、WebSocket、Pub/Sub 广播。
- **异步非阻塞**：基于 *asyncio*，适合 IO 密集型任务。
- **易部署**：命令行或 Python 脚本均可启动。

**快速上手示例**

```py
# example.py
async def app(scope, receive, send):
   assert scope['type'] == 'http'
   await send({
       'type': 'http.response.start',
       'status': 200,
       'headers': [(b'content-type', b'text/plain')],
   })
   await send({
       'type': 'http.response.body',
       'body': b'Hello, world!',
   })
```

命令行启动：

```bash
uvicorn example:app --reload --host 0.0.0.0 --port 8000
```

脚本启动：

```bash
uvicorn example:app --reload --host 0.0.0.0 --port 8000
```

**常用参数**

- *--host*：监听地址（默认 127.0.0.1）
- *--port*：端口（默认 8000）
- *--reload*：代码热重载（开发模式）
- *--workers*：进程数（生产环境建议 ≥ CPU 核心数）
- *--ssl-keyfile* / *--ssl-certfile*：启用 HTTPS

**高级用法**

- **与 FastAPI 集成**：

```py
from fastapi import FastAPI
import uvicorn
app = FastAPI()
@app.get("/")
async def root():
   return {"message": "Hello World"}
if __name__ == '__main__':
   uvicorn.run(app, host="0.0.0.0", port=8000)
```

- **WebSocket 支持**：实时通信应用。
- **中间件**：如 *ProxyHeadersMiddleware* 处理代理头。
- **异步任务**：*asyncio.create_task* 在后台运行任务。

**适用场景**

- 高并发 API 服务
- 实时 WebSocket 应用
- 异步微服务网关

Uvicorn 以其**极速性能**和**简洁配置**，已成为 Python 异步 Web 服务的首选生产级服务器。
