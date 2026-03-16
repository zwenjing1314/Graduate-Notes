# 介绍

是 **Python 的一个装饰器**，用于自动生成类的一些常用方法，如 **init** 、 **repr** 、 **eq** 等。 不需要显式地定义这些方法，dataclass 会根据类的字段自动生成。

# 常见API

## dataclass

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

@dataclass(slots=True)中的slots参数

作用使用\__slots__ 机制来优化数据类的内存占用和访问速度。

```py
@dataclass(slots=True)
class OCRWord:
    """Single OCR word/token."""

    text: str
    bbox: BBox
    confidence: float | None = None
    word_id: str | None = None
    index: int | None = None

    def __post_init__(self) -> None:
        self.text = self.text.strip()

    @property
    def is_empty(self) -> bool:
        return not self.text
```

具体好处
节省内存：不使用 __dict__ 存储实例属性，而是为所有实例共享一个固定的内存结构。对于创建大量实例的场景（如你的 BBox、OCRWord、OCRLine 等），可以显著减少内存消耗。
更快的属性访问：由于属性存储在预分配的槽中而不是动态的 __dict__ 中，访问速度更快。
防止随意添加属性：只能访问在类定义中声明的属性，不能在运行时动态添加新属性，这有助于捕获错误并保持数据结构的严格性。

注意事项
使用 slots=True 后：
不能动态添加属性（如 obj.new_attr = value 会报错）
默认不可 pickle（除非显式声明 __slots__ 包含 __dict__）
不支持多重继承（如果多个父类都有 __slots__）
总的来说，这是一个性能优化的选择，特别适合你的这种数据处理场景。

## field

field() 用于配置 dataclass 中属性的行为，提供比直接赋值更灵活的控制。

常见用法
1. default_factory - 可变默认值（代码中使用的）

```py
meta: dict[str, Any] = field(default_factory=dict)
answers: list[str] = field(default_factory=list)
words: list[OCRWord] = field(default_factory=list)
```

为什么需要？
❌ 错误写法：meta: dict = {} （所有实例共享同一个字典！）
✅ 正确写法：meta: dict = field(default_factory=dict) （每个实例获得独立的新字典）
原理：每次创建新实例时，调用 default_factory 指定的可调用对象生成新的默认值。

2. default - 固定默认值

```py
@dataclass
class User:
    name: str
    age: int = field(default=18)
    status: str = field(default="active")
# 等价于
name: str           # 必须提供
age: int = 18       # 可选，默认 18
status: str = "active"  # 可选，默认 "active"
```

3. init - 控制是否加入 __init_

```py
@dataclass
class Product:
    name: str
    price: float
    created_at: datetime = field(default_factory=datetime.now, init=False)
```

init=False：创建对象时不需要传这个参数
通常用于内部计算或自动生成的字段

4. repr - 控制是否在 __repr__ 中显示

```py
@dataclass
class User:
    username: str
    password: str = field(repr=False)  # repr 时不显示敏感信息
# 输出实例
User(username="alice")  # password 不会出现在 repr 中
```

5. hash - 控制哈希行为

```py
@dataclass(frozen=True)
class Point:
    x: int
    y: int
    metadata: dict = field(default_factory=dict, hash=False)  # 不参与 hash 计算
```

6. compare - 控制比较操作

```py
@dataclass(order=True)
class Student:
    grade: int
    name: str = field(compare=False)  # 排序时忽略 name
```

我的代码中的应用场景

```py
@dataclass(slots=True)
class DocumentSample:
    answers: list[str] = field(default_factory=list)      # 每个样本有独立的答案列表
    meta: dict[str, Any] = field(default_factory=dict)    # 每个样本有独立的元数据
    
@dataclass(slots=True)
class OCRPage:
    lines: list[OCRLine] = field(default_factory=list)    # 每页有独立的行列表
    words: list[OCRWord] = field(default_factory=list)    # 每页有独立的词列表
```



# 踩过的坑+解决方法

