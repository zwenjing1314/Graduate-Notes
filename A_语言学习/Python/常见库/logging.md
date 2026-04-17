# 介绍

**是非常常用的记录日志库**，通过 logging 模块存储各种格式的日志，主要用于输出运行日志，可以设置输出日志的等级、日志保存路径、日志文件回滚等

# 常见API

## `getLogger`()

`logging.getLogger(name)` 遵循 **单例模式**。这意味着：

- **全局唯一性**：在程序的任何地方调用相同名称的 `getLogger`，返回的都是同一个对象。
- **层级结构**：名称中的点号 `.` 表示层级。例如 `a.b` 是 `a` 的子级。

常用实践

通常在开发中，我们会看到这种写法：

```py
logger = logging.getLogger(__name__)
```

这里的 `__name__` 是 Python 的内置变量，会自动获取当前模块的路径（如 `package.module`）。这样做的好处是**日志会自动分类**，方便你在排查问题时一眼看出哪行代码报错。

`logger = logging.getLogger("mmclip.cli")` 的具体作用是：

- **创建/获取记录器**：声明一个名为 `mmclip.cli` 的日志对象。
- **命名空间定位**：通过字符串 `"mmclip.cli"`，它告诉程序这条日志来源于 `mmclip` 项目下的 `cli` 模块。
- **继承配置**：它会自动继承父级（如 `mmclip` 或根 logger）的设置（如日志级别、输出格式等）。

核心用法总结

我们可以把 `getLogger` 的逻辑简化为以下三点：

特性		说明
按名索引	传入相同字符串，永远返回同一个 logger 对象。
树状管理	mmclip.cli 会继承 mmclip 的所有配置，无需重复设置。
解耦控制	你可以单独把 mmclip.cli 的级别设为 DEBUG，而让其他模块保持 INFO。

**关键提醒**：如果不传参数 `logging.getLogger()`，它会返回 **Root Logger**（根记录器），这是所有日志的终点。

# 系统记一下

## 1. 导入

```py
import loggging
```

## 2. 日志级别

日志级别用于控制日志的详细程度。`logging` 模块提供了以下几种日志级别：

- **DEBUG**：详细的调试信息，通常用于开发阶段。
- **INFO**：程序正常运行时的信息。
- **WARNING**：表示潜在的问题，但程序仍能正常运行。
- **ERROR**：表示程序中的错误，导致某些功能无法正常工作。
- **CRITICAL**：表示严重的错误，可能导致程序崩溃。

默认日志级别是warning

可以通过以下代码设置日志级别：
```py
logging.basicConfig(level=logging.DEBUG)
```

## 3. 记录日志

设置好日志级别后，可以使用以下方法记录日志：
```py
import logging

# 默认的日志输出级别是warning
# 使用 logging.basicConfig() 配置日志输出级别
print("This is print log")  # 这个输出与logging输出顺序不一定，但是logging确实按顺序输出的
logging.basicConfig(level=logging.INFO)
logging.debug("This is debug log")
logging.info("This is info log")
logging.warning("This is warning log")
logging.error("This is error log")
logging.critical("This is critical log")

"""
INFO:root:This is info log
WARNING:root:This is warning log
ERROR:root:This is error log
CRITICAL:root:This is critical log
This is print log
"""


#####################
import logging


logging.basicConfig(level=logging.DEBUG)  # 默认日志是追加在文件尾部的
name = "张三"
age = 18
logging.debug("姓名 %s, 年龄 %d", name, age)
logging.debug("姓名  %s, 年龄 %d" %(name, age))
logging.debug("姓名  {}, 年龄 {}".format(name, age))
logging.debug(f"姓名  {name}, 年龄 {age}")

'''
DEBUG:root:姓名  张三, 年龄 18
DEBUG:root:姓名  张三, 年龄 18
DEBUG:root:姓名  张三, 年龄 18
DEBUG:root:姓名  张三, 年龄 18
'''
```

## 4. 日志输出格式

你可以通过 `basicConfig` 方法自定义日志的输出格式。例如：

```py
import logging

# 输出格式和添加一些公共信息
logging.basicConfig(format="%(asctime)s %(levelname)s %(filename)s %(lineno)s %(message)s",
                    datefmt="%Y-%m_%d %H:%M:%S", level=logging.DEBUG)  # 默认日志是追加在文件尾部的
name = "张三"
age = 18
logging.debug("姓名 %s, 年龄 %d", name, age)
logging.error("姓名 %s, 年龄 %d", name, age)
"""
2026-03_07 18:12:23 DEBUG main.py 8 姓名 张三, 年龄 18
2026-03_07 18:12:23 ERROR main.py 9 姓名 张三, 年龄 18
"""
```

## 5. 将日志输出到文件

默认情况下，日志会输出到控制台。如果你希望将日志保存到文件中，可以这样配置：

```py
import logging

logging.basicConfig(filename="demo.log",level=logging.INFO)  # 默认日志是追加在文件尾部的
# logging.basicConfig(filename="demo.log", filemode="w", level=logging.INFO)  # 将filemode="w"，则每次运行都会覆盖掉之前的日志
logging.debug("This is debug log")
logging.info("This is info log")
logging.warning("This is warning log")
logging.error("This is error log")
logging.critical("This is critical log")
```

## logging的高级应用

logging模块采用了模块化设计，主要包含四种组件：

**`logging.Logger`**：

 记录器，用于发出日志消息（通过 `logging.getLogger(name)` 获取） logger = logging.getLogger("my_logger")

**`logging.Handler`**：

处理器，决定日志输出位置（如文件、控制台等） handler = logging.FileHandler("app.log")

**`logging.Formatter`**：

格式化器，控制日志输出的格式  formatter = logging.Formatter('%(asctime)s - %(levelname)s - %(message)s')

**`logging.Filter`**：

 过滤器，用于更精细地控制日志记录 filter = logging.Filter("module.name") 

### 处理流程

​		一、 创建一个logger并设置默认等级

二、 创建屏幕 StreamHandler 	二、创建文件 FileHandleer

三、设置日志等级			   三、设置日志等级

​			 四、创建 formatter

​	      五、用 formatter 渲染所有的 Handler 

​	      六、将所有的 Handler 加入 logger 内

​			七、程序调用logger