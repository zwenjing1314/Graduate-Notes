# 介绍

`torch.Tensor` 是 **PyTorch** 中最核心的数据结构，用来表示多维数组（张量），类似于 NumPy 的 `ndarray`，但它额外支持 **GPU 加速** 和 **自动梯度计算**，因此在深度学习中被广泛使用。

# 常用API

## permute()

`permute()` 是 `torch.Tensor` 的一个方法，用于 **重新排列张量的维度顺序**，不会改变数据本身，只是改变维度的排列方式（返回的是原张量的视图，数据共享内存）。

```
Tensor.permute(*dims)
```

**参数**：`dims` 是一组整数，表示新的维度顺序。

**要求**：`dims` 必须是原张量所有维度的一个排列（即包含所有维度索引且不重复）。

**返回值**：一个新的张量视图（view），维度顺序已改变。

示例

```py
import torch

# 创建一个形状为 (2, 3, 4) 的张量
x = torch.randn(2, 3, 4)
print("原形状:", x.shape)  # torch.Size([2, 3, 4])

# 交换维度顺序
y = x.permute(1, 0, 2)  # 新顺序: 原第1维 -> 新第0维, 原第0维 -> 新第1维, 原第2维保持
print("新形状:", y.shape)  # torch.Size([3, 2, 4])
```

## transpose()

`transpose()` 是一个用于交换张量（Tensor）两个维度的函数，常用于调整数据的维度顺序以适配不同的网络层输入要求。

```
Tensor.transpose(dim0, dim1)
```

**dim0**：要交换的第一个维度（int）

**dim1**：要交换的第二个维度（int）

返回一个 **新的视图（view）**，不会复制数据（除非后续调用 `.contiguous()`）

示例

```py
import torch

# 创建一个形状为 (2, 3, 4) 的张量
x = torch.arange(24).reshape(2, 3, 4)
print("原始形状:", x.shape)

# 交换第 0 维和第 1 维
y = x.transpose(0, 1)
print("交换后形状:", y.shape)

# 交换第 1 维和第 2 维
z = x.transpose(1, 2)
print("交换后形状:", z.shape)
"""
原始形状: torch.Size([2, 3, 4])
交换后形状: torch.Size([3, 2, 4])
交换后形状: torch.Size([2, 4, 3])
"""
```

