# 介绍

`torch` 是核心库，几乎所有的张量运算、数学计算、自动求导等功能都来自它。可以把它理解为 **NumPy 的 GPU 加强版**，但同时支持深度学习所需的自动微分和神经网络构建。

# 常见API

## arange()

返回一个大小为 ⌈end−startstep⌉ / step 的一维张量，其值取自区间 `[start, end)`，以 start 为起点，公差为 `step`。

```py
torch.arange(start=0, end, step=1, *, out=None, dtype=None, layout=torch.strided, device=None, requires_grad=False) → Tensor
```

**start** (*Number**,* *可选*) – 点集合的起始值。默认值：`0`。

**end** (*Number*) – 点集合的结束值。

**step** (*Number**,* *可选*) – 相邻点对之间的间距。默认值：`1`。

示例

```py
torch.arange(5)
torch.arange(1, 4)
torch.arange(1, 2.5, 0.5)
```



## cat()

基本语法

```py
torch.cat(tensors, dim=0, out=None)
```

- tensors: 要拼接的张量序列（元组或列表）
- dim: 沿哪个维度进行拼接（整数）
- out: 可选，输出到的目标张量

torch.cat() 会将多个张量沿指定维度首尾相连，形成一个更大的张量。

示例

```py
import torch

# 示例 1：沿维度 0 拼接（垂直堆叠）
a = torch.tensor([[1, 2], [3, 4]])  # shape: (2, 2)
b = torch.tensor([[5, 6]])          # shape: (1, 2)
result = torch.cat((a, b), dim=0)
# result: [[1, 2],
#          [3, 4],
#          [5, 6]]  shape: (3, 2)

# 示例 2：沿维度 1 拼接（水平拼接）
a = torch.tensor([[1, 2], [3, 4]])  # shape: (2, 2)
b = torch.tensor([[5], [6]])        # shape: (2, 1)
result = torch.cat((a, b), dim=1)
# result: [[1, 2, 5],
#          [3, 4, 6]]  shape: (2, 3)
```

## unsqueeze()

*unsqueeze* 函数用于在指定的维度上插入一个大小为 1 的维度。这个函数对于改变张量的形状非常有用，特别是在需要对张量的形状进行匹配以便进行后续操作时。

基本用法

*torch.unsqueeze(input, dim)* 接受两个参数：

- *input*：输入张量。
- *dim*：要插入的维度索引。索引范围是 *[-input.dim()-1, input.dim()]*，负索引将从末尾开始计算。

示例

```py
import torch
# 创建一个2D张量
x = torch.tensor([[1, 2, 3], [4, 5, 6]])
print("原始张量：", x)
print("原始张量形状：", x.shape)
# 在0维度上插入一个新的维度
x_unsqueezed_0 = torch.unsqueeze(x, 0)
print("在0维度上插入新维度后的张量：", x_unsqueezed_0)
print("在0维度上插入新维度后的张量形状：", x_unsqueezed_0.shape)
# 在1维度上插入一个新的维度
x_unsqueezed_1 = torch.unsqueeze(x, 1)
print("在1维度上插入新维度后的张量：", x_unsqueezed_1)
print("在1维度上插入新维度后的张量形状：", x_unsqueezed_1.shape)
# 在2维度上插入一个新的维度
x_unsqueezed_2 = torch.unsqueeze(x, 2)
print("在2维度上插入新维度后的张量：", x_unsqueezed_2)
print("在2维度上插入新维度后的张量形状：", x_unsqueezed_2.shape)

”“”
原始张量： tensor([[1, 2, 3], [4, 5, 6]])
原始张量形状： torch.Size([2, 3])
在0维度上插入新维度后的张量： tensor([[[1, 2, 3], [4, 5, 6]]])
在0维度上插入新维度后的张量形状： torch.Size([1, 2, 3])
在1维度上插入新维度后的张量： tensor([[[1, 2, 3]], [[4, 5, 6]]])
在1维度上插入新维度后的张量形状： torch.Size([2, 1, 3])
在2维度上插入新维度后的张量： tensor([[[1], [2], [3]], [[4], [5], [6]]])
在2维度上插入新维度后的张量形状： torch.Size([2, 3, 1])
“”“
```

## save()

保存对象

```py
torch.save(obj, f, pickle_module=pickle, pickle_protocol=2, _use_new_zipfile_serialization=True)
```

参数说明：

- obj: 要保存的 Python 对象（模型、张量、字典等）
- f: 文件路径或文件对象
- pickle_module: Pickle 序列化模块（默认用 pickle）
- pickle_protocol: Pickle 协议版本
- _use_new_zipfile_serialization: 是否使用新的 zipfile 格式（PyTorch 1.6+）

底层原理流程：

```
┌─────────────────────────────────────────────────────┐
│ torch.save(model.state_dict(), 'model.pt')          │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│ 1. 检查对象类型                                      │
│    - Tensor → 直接序列化                            │
│    - dict → 递归序列化每个键值对                     │
│    - nn.Module → 调用 state_dict() 获取参数字典     │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│ 2. 使用 pickle 序列化                                │
│    obj → pickle.dumps(obj) → bytes                  │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│ 3. 写入文件                                          │
│    - 旧版本：直接写入 pickle bytes                   │
│    - 新版本：打包成 zipfile 格式                     │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│ 4. 生成 .pt 文件                                     │
└─────────────────────────────────────────────────────┘
```

示例

```py
import torch
import torch.nn as nn

# ========== 示例 1: 保存模型参数（推荐）==========
model = nn.Linear(10, 1)
torch.save(model.state_dict(), 'model_params.pt')
# 文件内容：{'weight': tensor(...), 'bias': tensor(...)}

# ========== 示例 2: 保存整个模型 ==========
torch.save(model, 'complete_model.pt')
# 保存了模型结构 + 参数

# ========== 示例 3: 保存检查点 ==========
torch.save({
    'epoch': 50,
    'model_state_dict': model.state_dict(),
    'optimizer_state_dict': optimizer.state_dict(),
    'loss': 0.023,
}, 'checkpoint.tar')

# ========== 示例 4: 保存多个模型 ==========
torch.save({
    'encoder': encoder.state_dict(),
    'decoder': decoder.state_dict(),
}, 'seq2seq_model.pt')
```

## load()

 加载对象

```
torch.load(f, map_location=None, pickle_module=pickle, weights_only=False, **pickle_load_args)
```

参数说明：

- f: 文件路径或文件对象
- map_location: 设备映射（CPU/GPU）
- pickle_module: Pickle 反序列化模块
- weights_only: 是否只加载权重数据（安全模式，PyTorch 1.13+）

```
┌─────────────────────────────────────────────────────┐
│ torch.load('model.pt')                              │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│ 1. 读取文件                                          │
│    - 检测是否为 zipfile 格式                         │
│    - 解压并读取 pickle bytes                         │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│ 2. 使用 pickle 反序列化                              │
│    bytes → pickle.loads(bytes) → obj                │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│ 3. 设备映射（如果指定了 map_location）              │
│    - CUDA tensor → CPU: tensor.cuda().cpu()         │
│    - CPU tensor → CUDA: tensor.cpu().cuda()         │
└─────────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────────┐
│ 4. 返回还原的对象                                    │
└─────────────────────────────────────────────────────┘
```

示例

```py
# ========== 示例 1: 加载参数 ==========
state_dict = torch.load('model_params.pt')
# state_dict = {'weight': tensor(...), 'bias': tensor(...)}

# ========== 示例 2: 加载到 CPU（即使是在 GPU 上训练的）==========
state_dict = torch.load('model.pt', map_location=torch.device('cpu'))

# ========== 示例 3: 加载检查点 ==========
checkpoint = torch.load('checkpoint.tar')
model.load_state_dict(checkpoint['model_state_dict'])
optimizer.load_state_dict(checkpoint['optimizer_state_dict'])
epoch = checkpoint['epoch']

# ========== 示例 4: 安全加载（PyTorch 1.13+）==========
# weights_only=True 防止执行恶意代码
state_dict = torch.load('model.pt', weights_only=True)
```

## from_numpy()

`torch.from_numpy()` 是 **PyTorch** 中用于将 **NumPy 数组** 转换为 **PyTorch 张量** 的函数。
 它的特点是 **共享内存** —— 修改其中一个，另一个也会同步变化（除非显式复制）。

```py
import numpy as np
import torch

# 创建 NumPy 数组
np_array = np.array([[1, 2, 3], [4, 5, 6]], dtype=np.float32)

# 转换为 PyTorch 张量（共享内存）
tensor_from_np = torch.from_numpy(np_array)

print("NumPy 数组:\n", np_array)
print("PyTorch 张量:\n", tensor_from_np)

# 修改 NumPy 数组，张量同步变化
np_array[0, 0] = 99
print("\n修改 NumPy 后的张量:\n", tensor_from_np)

# 修改张量，NumPy 同步变化
tensor_from_np[1, 1] = -5
print("\n修改张量后的 NumPy:\n", np_array)

"""
NumPy 数组:
 [[1. 2. 3.]
 [4. 5. 6.]]
PyTorch 张量:
 tensor([[1., 2., 3.],
        [4., 5., 6.]])

修改 NumPy 后的张量:
 tensor([[99.,  2.,  3.],
        [ 4.,  5.,  6.]])

修改张量后的 NumPy:
 [[99.  2.  3.]
 [ 4. -5.  6.]]
"""
```

## unique()

类似于数学中的集合，就是挑出 tensor 中的独立不重复元素。

```
torch.unique(input, sorted=True, return_inverse=False, return_counts=False, dim=None)
```

input: 待处理的tensor

sorted：是否对返回的无重复张量按照数值进行排列，默认是生序排列的

return_inverse: 是否返回原始tensor中的每个元素在这个无重复张量中的索引

return_counts: 统计原始张量中每个独立元素的个数

dim: 值沿着哪个维度进行unique的处理，这个我试验后没有搞懂怎样的机理。如果处理的张量都是一维的，那么这个不需要理会。


示例

```py
import torch
 
x = torch.tensor([[1，3][1,2],[1,3]])#生成一个tensor,作为实验输入
out = torch.unique(x) #所有参数都设置为默认的
print(out)#将处理结果打印出来
#结果如下：
#tensor([1,2,3])   #将x中的不重复元素挑了出来，并且默认为生序排列
 
out = torch.unique(x,sorted=False)#将默认的生序排列改为False
print(out)
#输出结果如下：
#tensor([1,3,2])  #将x中的独立元素找了出来，就按照原始顺序输出
 
out = torch.unique(x,return_inverse=True)#将原始数据中的每个元素在新生成的独立元素张量中的索引输出
print(out)
#输出结果如下：
#(tensor([1, 2, 3]), tensor([[0,2],[0,1],[0,2]]))  #第一个张量是排序后输出的独立张量，第二个结果对应着原始数据中的每个元素在新的独立无重复张量中的索引，比如x[0]=4,在新的张量中的索引为4, x[1]=0,在新的张量中的索引为0，x[6]=3,在新的张量中的索引为3
 
out = torch.unique(x,return_counts=True) #返回每个独立元素的个数
print(out)
#输出结果如下
#(tensor([1, 2, 3]), tensor([3,1,2]))  #这里不清楚
out = torch.unique(x,dim=0) #按照第一个维度去重
tensor([[1,2],[1,3]])
```

