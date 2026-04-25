# 介绍

*threading* 是 Python 提供的一个高层次线程管理模块，用于在单个进程中并发运行多个线程。线程适用于 I/O 密集型任务（如文件操作、网络请求等），但对于 CPU 密集型任务，由于全局解释器锁（GIL）的限制，性能提升有限。



# 常见API

## Thread

threading.Thread 是 Python 标准库中用于创建和管理线程的类。线程是操作系统能够进行运算调度的最小单位，它被包含在进程之中，是进程中的实际运作单位。
在你的代码场景中，使用线程的目的是：让 Uvicorn 服务器在后台运行，不阻塞 Jupyter Notebook 的主线程，这样你可以继续执行后续单元格中的代码。

代码中的数据流向和执行过程

```py
# 第1步：创建线程对象（此时线程还未启动）
thread = threading.Thread(target=server.run, daemon=True)
#                       ↓              ↓
#                  要执行的函数    设置为守护线程

# 第2步：启动线程（真正开始执行）
thread.start()
#     ↓
# 操作系统创建一个新线程
# 在新线程中调用 server.run()
# 主线程（Jupyter）继续执行下一行代码

# 第3步：主线程等待1秒，让服务器完成初始化
time.sleep(1)
#     ↓
# 此时两个线程并行运行：
# - 主线程：继续执行 Jupyter 的代码
# - 子线程：运行 Uvicorn 服务器，监听 HTTP 请求

# 第4步：打印消息（主线程执行）
print("Server started at http://127.0.0.1:8000")
```

### Thread() 参数详解

1. target 参数（必需/常用）
- 作用：指定线程要执行的函数（可调用对象）
- 类型：函数、方法或任何可调用对象
- 你的代码：target=server.run
  - 这里传递的是 server.run 方法本身（注意没有括号 ()）
  - 线程启动时会调用这个方法，相当于在新线程中执行 server.run()

重要理解：

```py
# ❌ 错误写法：立即执行函数，将返回值传给 target
thread = threading.Thread(target=server.run())  # run() 会立即在主线程执行

# ✅ 正确写法：传递函数引用，线程启动时才执行
thread = threading.Thread(target=server.run)    # 只传递函数对象
```

2. daemon 参数（可选）
  - 作用：设置线程是否为"守护线程"
  - 类型：布尔值（True 或 False）
  - 默认值：False
  - 你的代码：daemon=True

守护线程 vs 非守护线程的区别：

特性			守护线程 (daemon=True)	 		非守护线程 (daemon=False)

程序退出行为	主线程结束时，守护线程自动终止	主线程会等待所有非守护线程结束

适用场景			后台服务、监控任务				要完成的重要任务

你的场景		✅ 适合：服务器在后台运行即可		❌ 不适合：会导致 Jupyter 无法退出

实际效果演示：

```py
# 情况1：daemon=True（你的代码）
thread = threading.Thread(target=server.run, daemon=True)
thread.start()
# 当 Jupyter 停止或程序退出时，服务器线程会自动被杀死
# 不会阻止程序退出

# 情况2：daemon=False（默认）
thread = threading.Thread(target=server.run, daemon=False)
thread.start()
# 即使你尝试停止 Jupyter，Python 也会等待服务器线程结束
# 可能导致程序无法正常退出
```

3. args 参数（可选）

  - 作用：传递给 target 函数的位置参数（元组形式）
  - 类型：元组
  - 示例：

  ```py
  def greet(name, greeting):
      print(f"{greeting}, {name}!")
  
  # 传递参数给 target 函数
  thread = threading.Thread(target=greet, args=("World", "Hello"))
  thread.start()
  # 输出：Hello, World!
  ```

  

4. kwargs 参数（可选）

  - 作用：传递给 target 函数的关键字参数（字典形式）
  - 类型：字典
  - 示例：

  ```py
  def greet(name, greeting="Hello"):
      print(f"{greeting}, {name}!")
  
  thread = threading.Thread(target=greet, kwargs={"name": "World", "greeting": "Hi"})
  thread.start()
  # 输出：Hi, World!
  ```

  

5. name 参数（可选）

  - 作用：给线程起一个名字，便于调试和识别
  - 类型：字符串
  - 默认值：自动生成（如 Thread-1, Thread-2）
  - 示例：

  ```py
  thread = threading.Thread(target=server.run, name="UvicornServer")
  print(thread.name)  # 输出：UvicornServer
  ```

  

### Thread 对象的常用方法

创建线程对象后，可以使用以下方法控制线程：

1. start() - 启动线程

```py
thread.start()
# 触发操作系统创建新线程
# 新线程开始执行 target 指定的函数
# 每个线程只能调用一次 start()
```

2. join(timeout=None) - 等待线程结束

```py
# 在你的代码第121行使用：
thread.join(timeout=3)
#       ↓
# 主线程等待子线程结束，最多等3秒
# 如果超时，主线程继续执行（不管子线程是否结束）

# 其他用法：
thread.join()        # 无限期等待，直到线程结束
thread.join(5)       # 最多等5秒
```

数据流向：

```py
主线程执行到 thread.join(timeout=3)
         ↓
    暂停主线程，等待子线程
         ↓
    ┌────条件判断────┐
    ↓                ↓
子线程在3秒内结束    超过3秒子线程还在运行
    ↓                ↓
主线程继续执行      主线程也继续执行（不再等待）
```

3. is_alive() - 检查线程是否还在运行

```py
if thread.is_alive():
    print("服务器还在运行")
else:
    print("服务器已停止")
```

### 为什么在你的代码中需要使用线程？

让我们对比两种情况：
❌ 不用线程（直接调用）

```py
config = uvicorn.Config(app, host="127.0.0.1", port=8000, log_level="info")
server = uvicorn.Server(config)
server.run()  # ← 这会阻塞！

# 上面的代码永远不会执行到这里
print("Server started")  # ❌ 这行代码不会执行
```

问题：server.run() 是一个阻塞调用，它会一直运行直到服务器停止。在 Jupyter 中，这会导致：

- 单元格永远显示 [*]（正在执行）
- 无法执行后续的测试代码
- 无法发送 HTTP 请求测试服务器

✅ 使用线程（你的代码）

```py
thread = threading.Thread(target=server.run, daemon=True)
thread.start()  # ← 在后台启动，立即返回
time.sleep(1)   # ← 给服务器1秒时间初始化
print("Server started")  # ✅ 这行代码会执行
```

优势：

- 服务器在后台线程运行
- 主线程可以继续执行后续代码
- 可以发送 HTTP 请求测试服务器
- 可以随时通过 server.should_exit = True 停止服务器

### 完整的执行流程图

```py
时间线 →

主线程 (Jupyter)                    子线程 (后台)
─────────────────                   ────────────────
创建 Thread 对象                    
    ↓                               
调用 thread.start()                 
    ├──────────────────→           操作系统创建新线程
    │                                   ↓
    │                              执行 server.run()
    │                                   ↓
time.sleep(1)                     绑定到 127.0.0.1:8000
    │                                   ↓
    │                              开始监听 HTTP 请求
    │                                   ↓
print("Server started")                 │
    │                                   │
执行测试代码                      处理收到的请求
urlopen("http://...")          ←── 接收请求
    │                               ↓
    │                           返回响应
    │                          ──→ 发送响应
    ↓                                   │
                                        │
server.should_exit = True               │
thread.join(timeout=3)                  ↓
    ├──────────────────→           清理资源，退出
    │                                   ↓
    │                              线程结束
主线程继续执行
```

## Lock

Lock 是 Python threading 模块提供的互斥锁（Mutex），用于在多线程环境中保护共享资源，防止多个线程同时访问和修改同一数据而导致的数据不一致问题。

### 应用中的使用场景

1. 锁的初始化

```py
_request_state_lock = Lock()
```

