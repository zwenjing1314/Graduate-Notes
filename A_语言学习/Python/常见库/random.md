# 介绍

提供了各种用于生成随机数的函数

# 常见用法

## random.sample(sequence, k)

- **sequence**：被抽样的序列，可以是列表、元组或字符串等可迭代对象。
- **k**：抽取的样本数量。

```py
import random

students = ["Alice", "Bob", "Charlie", "David", "Eve", "Frank", "Grace", "Harry", "Ivy", "John"]
sample = random.sample(students, min(5, min(5, len(students))))
print(sample)
```

## random.default_rng(42)

*np.random.default_rng(42)* 是 NumPy 中用于创建随机数生成器的推荐方法。它基于现代的 PCG-64 算法，提供更高性能和灵活性，同时确保随机数生成的可重复性。

```py
import numpy as np
# 创建随机数生成器
rng = np.random.default_rng(42)
# 生成 5 个 [0, 1) 区间内的均匀分布随机数
uniform = rng.random(5)
print("均匀分布:", uniform)
# 生成 3 个 [0, 10) 的随机整数
integers = rng.integers(0, 10, size=3)
print("随机整数:", integers)
# 生成 4 个均值为 0，标准差为 1 的正态分布随机数
normal = rng.normal(0, 1, size=4)
print("正态分布:", normal)
# 从列表中随机选择 2 个元素
choices = rng.choice(['a', 'b', 'c'], size=2, replace=True)
print("随机选择:", choices)

"""
均匀分布: [0.77395605 0.43887844 0.85859792 0.69736803 0.09417735]
随机整数: [5 9 7]
正态分布: [-0.31624259 -0.01680116 -0.85304393  0.87939797]
随机选择: ['c' 'a']
"""
```

