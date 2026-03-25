# 介绍

**Pydantic** 是一个基于 Python 类型注解（type hints）的数据校验和数据模型管理库，其核心理念是“数据校验即数据解析”。它能够帮助开发者高效地处理复杂的数据结构，同时提供直观的错误提示，提升开发效率。

# 常见API

## model_dump_json(indent=2) 

方法签名

```py
model.model_dump_json(
    *,
    indent: int | None = None,     # 缩进空格数（美化输出）
    include: set[str] | None = None,  # 包含的字段
    exclude: set[str] | None = None,  # 排除的字段
    by_alias: bool = False,        # 是否使用别名
    exclude_unset: bool = False,   # 排除未设置的字段
    exclude_defaults: bool = False, # 排除默认值的字段
    exclude_none: bool = False,    # 排除值为 None 的字段
    round_trip: bool = False,      # 往返模式（保持格式）
) -> str                           # 返回 JSON 字符串
```

在你的代码中的用法

```py
parsed.model_dump_json(indent=2)
```

parsed 是 ParsedDocument 类型的 Pydantic 模型实例
indent=2 表示用 2 个空格缩进（美化 JSON 输出）

效果对比
不使用 indent（紧凑格式）：

```py
parsed.model_dump_json()
# 输出：
{"source_path":"paper.pdf","title":"A Paper","page_count":5,"pages":[{"page_num":1,"text":"..."}]}
```

使用 indent=2（美化格式）：

```py
parsed.model_dump_json(indent=2)
# 输出：
{
  "source_path": "paper.pdf",
  "title": "A Paper",
  "page_count": 5,
  "pages": [
    {
      "page_num": 1,
      "text": "..."
    }
  ]
}
```

示例

```py
from pydantic import BaseModel

class Person(BaseModel):
    name: str
    age: int
    email: str | None = None

person = Person(name="Alice", age=25, email="alice@example.com")

# 示例 1: 基础用法（紧凑）
json_str = person.model_dump_json()
# '{"name":"Alice","age":25,"email":"alice@example.com"}'

# 示例 2: 美化输出
json_pretty = person.model_dump_json(indent=2)
# '''
# {
#   "name": "Alice",
#   "age": 25,
#   "email": "alice@example.com"
# }
# '''

# 示例 3: 排除特定字段
json_no_email = person.model_dump_json(exclude={"email"})
# '{"name":"Alice","age":25}'

# 示例 4: 只包含特定字段
json_only_name = person.model_dump_json(include={"name"})
# '{"name":"Alice"}'

# 示例 5: 排除 None 值
person2 = Person(name="Bob", age=30)  # email 为 None
json_no_none = person2.model_dump_json(exclude_none=True)
# '{"name":"Bob","age":30}'  # 不包含 email 字段
```

# 遇到的问题

## Pydantic BaseModel 的工作原理

核心机制：元类（Metaclass）+ 字段声明

当你写：

```py
class ParsedDocument(BaseModel):
    source_path: str
    title: str | None = None
    page_count: int
    pages: list[PageContent] = Field(default_factory=list)
```

Pydantic 会：

1. 拦截类的创建：通过元类 ModelMetaclass 捕获你的类定义
2. 解析类型注解：自动识别 source_path: str 这样的字段声明
3. 生成 __init__ 方法：自动创建初始化方法
4. 添加验证逻辑：在赋值时进行类型检查和数据验证

等价的传统写法

```py
class ParsedDocument:
    def __init__(
        self,
        source_path: str,
        title: str | None = None,
        page_count: int = None,
        pages: list[PageContent] | None = None
    ):
        # 类型验证
        if not isinstance(source_path, str):
            raise TypeError("source_path must be a string")
        
        self.source_path = source_path
        self.title = title
        self.page_count = page_count
        self.pages = pages if pages is not None else []
```

Pydantic 帮你自动生成了这些样板代码

实际例子演示

Pydantic 的自动验证和转换

```py
from pydantic import BaseModel

class User(BaseModel):
    id: int
    name: str
    email: str

# ✅ 自动类型转换
user1 = User(id="123", name="Alice", email="alice@example.com")
print(user1.id)  # 输出：123 (字符串 "123" 被转成整数 123)

# ❌ 验证失败会报错
try:
    user2 = User(id="abc", name="Bob", email="bob@example.com")
except ValueError as e:
    print(e)  # 输出：1 validation error for User\nid\n...input is not valid integer
```

如果用 dataclasses：

```py
from dataclasses import dataclass

@dataclass
class User:
    id: int
    name: str
    email: str

# ❌ 不会自动转换，类型错误
user1 = User(id="123", name="Alice", email="alice@example.com")
print(user1.id)  # 输出："123" (还是字符串，不会转换)

# ✅ 但 mypy 等工具会在静态检查时报错
```

