# 介绍

`collections` 模块是标准库中提供高性能专用容器数据类型的模块，用于补充内置类型（如 `dict`、`list`、`tuple`、`set`）的功能

# 常见类

## Counter

Counter 是一个计数器字典。它的主要作用是统计元素出现的次数。

- 本质：它是一个特殊的 dict，键（Key）是元素，值（Value）是该元素出现的次数。
- 默认值：如果你访问一个不存在的键，它不会报错，而是返回 0。这在做统计时非常方便。

示例：

```py
from collections import Counter

# 统计单词频率
c = Counter(["apple", "banana", "apple", "orange"])
print(c) 
# 输出: Counter({'apple': 2, 'banana': 1, 'orange': 1})

# 访问不存在的词
print(c["grape"]) 
# 输出: 0 (而不是报错 KeyError)
```

### update()

update 方法会把集合里的每个词在 Counter 里的计数加 1。
执行演示： 假设我们要处理两个文档：

1. 处理第一个文档 ["万科", "营收"]：
   - set(doc) → {"万科", "营收"}
   - self.df.update(...)
   - 此时 self.df 变为：Counter({"万科": 1, "营收": 1})
2. 处理第二个文档 ["万科", "比亚迪"]：
   - set(doc) → {"万科", "比亚迪"}
   - self.df.update(...)
   - 此时 self.df 变为：Counter({"万科": 2, "营收": 1, "比亚迪": 1})