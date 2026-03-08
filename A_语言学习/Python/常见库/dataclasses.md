# 介绍

是 **Python 的一个装饰器**，用于自动生成类的一些常用方法，如 **init** 、 **repr** 、 **eq** 等。 不需要显式地定义这些方法，dataclass 会根据类的字段自动生成。

# 常见API

导入

```py
from dataclasses import dataclass
```

@dataclass装饰器

在代码中的作用:

```py
@dataclass
class ClipEncoder:
    model_name: str = "openai/clip-vit-base-patch32"
    device: str = "cuda"
    use_amp: bool = True
```

这个装饰器自动为 ClipEncoder 类生成：
构造函数：根据声明的字段自动生成 __init__ 方法
字符串表示：生成 __repr__ 方法，方便打印调试
相等性比较：生成 __eq__ 方法
等价于手动编写：

```py
class ClipEncoder:
    def __init__(self, model_name: str = "openai/clip-vit-base-patch32", 
                 device: str = "cuda", 
                 use_amp: bool = True):
        self.model_name = model_name
        self.device = device
        self.use_amp = use_amp
```

 @dataclass(frozen=True) 中的 frozen 参数

作用：让对象不可修改（只读）
一旦创建了 IndexPaths 对象，就不能再修改它的属性了。

```py
from dataclasses import dataclass
from pathlib import Path

# ❌ 没有 frozen=True（默认情况）
@dataclass
class MutablePaths:
    out_dir: Path

paths1 = MutablePaths(Path("./output"))
paths1.out_dir = Path("./new_output")  # ✅ 可以修改（但可能导致错误）

# ✅ 有 frozen=True
@dataclass(frozen=True)
class ImmutablePaths:
    out_dir: Path

paths2 = ImmutablePaths(Path("./output"))
paths2.out_dir = Path("./new_output")  # ❌ 会报错！AttributeError: cannot assign to field 'out_dir'
```

为什么要用 frozen=True？
安全性保障：
IndexPaths 存储的是文件路径，这些路径在程序运行期间应该保持不变
防止意外修改导致文件读写错乱
让代码更可靠、更易维护

# 踩过的坑+解决方法
