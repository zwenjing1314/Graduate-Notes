# 介绍

NumPy 是 Python 中一个强大的科学计算库，主要用于数组和矩阵运算。它提供了多维数组对象（ndarray）以及丰富的数学函数库，广泛应用于数据处理、科学研究、机器学习等领域。

# 常用API

## concatenate()

```py
import numpy as np

a = [np.array([1, 2, 3]), np.array([4, 5, 6]), np.array([7, 8, 9])]
b = np.concatenate(a, axis=0)
print(b)

# [1 2 3 4 5 6 7 8 9]
```

axis参数

```py
import numpy as np
a = np.array([[[1, 2], [3, 4]], [[5, 6], [7, 8]]])
b = np.array([[[9, 10], [11, 12]], [[13, 14], [15, 16]]])
result = np.concatenate((a, b), axis=2)
result1 = np.concatenate((a, b), axis=1)
result2 = np.concatenate((a, b), axis=0)
print(result)
print(result1)
print(result2)

"""
[[[ 1  2  9 10]
  [ 3  4 11 12]]

 [[ 5  6 13 14]
  [ 7  8 15 16]]]
[[[ 1  2]
  [ 3  4]
  [ 9 10]
  [11 12]]

 [[ 5  6]
  [ 7  8]
  [13 14]
  [15 16]]]
[[[ 1  2]
  [ 3  4]]

 [[ 5  6]
  [ 7  8]]

 [[ 9 10]
  [11 12]]

 [[13 14]
  [15 16]]]
"""
```

注意事项

1. **维度一致性**：在使用 *np.concatenate* 时，除了指定的轴外，其他轴的维度必须一致。例如，在上述示例中，除了第三个维度外，其他两个维度的大小都必须相同。
2. **性能考虑**：对于大数组，拼接操作可能会消耗大量内存和计算资源，因此在处理大数据时需要谨慎。

## argsort()

函数返回的是数组值从小到大的索引值

```py
>>> x = np.array([3, 1, 2])
>>> np.argsort(x)
array([1, 2, 0])
```

``` py
>>> x = np.array([[0, 3], [2, 2]])
>>> x
array([[0, 3],
[2, 2]])

>>> np.argsort(x, axis=0) #按列排序
array([[0, 1],
[1, 0]])

>>> np.argsort(x, axis=1) #按行排序
array([[0, 1],
[0, 1]])
```

## tolist()

```py
import numpy as np
# 创建一个二维 NumPy 数组
array = np.array([[1, 2, 3], [4, 5, 6]])
# 使用 tolist 方法将数组转换为 Python 列表
list_from_array = array.tolist()
print(list_from_array) # 输出: [[1, 2, 3], [4, 5, 6]]
```

## transpose()

`np.transpose` 是 NumPy 中用于**交换数组的轴（维度）**的函数。它的基本作用是重新排列数组的维度顺序，相当于矩阵转置在高维空间的推广。

```
numpy.transpose(a, axes=None)
```

- **a**：输入数组。
- **axes**：可选参数，指定新轴的顺序。如果不提供，则默认反转轴的顺序（即对二维数组就是通常的转置）。

基本用法

```
import numpy as np

arr = np.array([[1, 2, 3],
                [4, 5, 6]])
print(arr.shape)      # (2, 3)

transposed = np.transpose(arr)
print(transposed)
# 输出：
# [[1 4]
#  [2 5]
#  [3 6]]
print(transposed.shape)  # (3, 2)
```

也可以使用数组的 `.T` 属性完成相同操作：

```
arr.T  # 结果与 np.transpose(arr) 相同
```



# 踩过的坑+解决办法