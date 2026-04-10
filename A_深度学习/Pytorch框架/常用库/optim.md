# 介绍

**torch.optim是一个实现了多种优化算法的包**，大多数通用的方法都已支持，提供了丰富的接口调用，未来更多精炼的优化算法也将整合进来。 为了使用torch.optim，需先 构造一个优化器对象Optimizer，用来保存当前的状态，并能够根据计算得到的梯度来更新参数。 要构建一个优化器optimizer，你必须给它一个可进行迭代优化的包含了所有参数（所有的参数必须是变量s）的列表。

## 优化器的作用

### 参数更新（最核心的功能）

```py
import torch
import torch.nn as nn
import torch.optim as optim

model = nn.Linear(3, 2, bias=False)
optimizer = optim.SGD(model.parameters(), lr=0.01)

# 初始权重
print("更新前 weight:")
print(model.weight.data)
# tensor([[ 0.1, -0.2,  0.3],
#         [-0.1,  0.4, -0.5]])

# 前向 + 反向传播
x = torch.tensor([[1.0, 2.0, 3.0]])
target = torch.tensor([[1.0, 0.0]])
output = model(x)
loss = nn.MSELoss()(output, target)
loss.backward()

print("\n梯度:")
print(model.weight.grad)
# tensor([[-0.4, -0.8, -1.2],
#         [ 0.4,  0.8,  1.2]])

# optimizer.step() - 使用梯度更新参数
optimizer.step()

print("\n更新后 weight:")
print(model.weight.data)
# tensor([[ 0.104, -0.192,  0.312],  ← weight = weight - lr * grad
#         [-0.104,  0.392, -0.512]])
# 计算：0.1 - 0.01 * (-0.4) = 0.104
```

更新公式：

```py
# SGD 的更新规则
param_new = param_old - learning_rate * gradient

# 对应代码
model.weight.data = model.weight.data - 0.01 * model.weight.grad
```

### 实现不同的优化算法

不同的优化器有不同的参数更新策略：

SGD (随机梯度下降)

```py
# 基础 SGD
optimizer = optim.SGD(model.parameters(), lr=0.01)

# 更新公式：θ = θ - lr * g
# 简单但可能震荡
```

SGD with Momentum (动量)

```py
optimizer = optim.SGD(model.parameters(), lr=0.01, momentum=0.9)

# 更新公式：
# v_t = γ * v_{t-1} + lr * g_t  (速度累积)
# θ_t = θ_{t-1} - v_t           (位置更新)

# 好处：
# - 加速收敛
# - 减少震荡
# - 能冲出局部最优
```

Adam (自适应学习率)

```py
optimizer = optim.Adam(model.parameters(), lr=0.001)

# 结合了 Momentum 和 RMSprop
# - 为每个参数自适应调整学习率
# - 考虑梯度的一阶矩（均值）和二阶矩（未中心化的方差）
# - 通常是最快收敛的选择
```

不同优化器对比演示

```py
import numpy as np
import matplotlib.pyplot as plt

# 创建一个简单的损失函数 landscape
def loss_function(x, y):
    return (x ** 2 + y ** 2)  # 简单的碗状曲面

# 比较三种优化器
optimizers_config = [
    ('SGD', optim.SGD, {'lr': 0.1}),
    ('SGD+Momentum', optim.SGD, {'lr': 0.1, 'momentum': 0.9}),
    ('Adam', optim.Adam, {'lr': 0.1}),
]

results = {}

for name, OptClass, kwargs in optimizers_config:
    # 重置参数
    x = torch.tensor([5.0], requires_grad=True)
    y = torch.tensor([5.0], requires_grad=True)
    optimizer = OptClass([x, y], **kwargs)
    
    trajectory_x = [x.item()]
    trajectory_y = [y.item()]
    
    # 优化 20 步
    for _ in range(20):
        optimizer.zero_grad()
        loss = loss_function(x, y)
        loss.backward()
        optimizer.step()
        
        trajectory_x.append(x.item())
        trajectory_y.append(y.item())
    
    results[name] = (trajectory_x, trajectory_y)

# 可视化对比
plt.figure(figsize=(10, 8))
x_grid = np.linspace(-6, 6, 100)
y_grid = np.linspace(-6, 6, 100)
X, Y = np.meshgrid(x_grid, y_grid)
Z = X**2 + Y**2
plt.contourf(X, Y, Z, levels=50, cmap='viridis', alpha=0.6)

for name, (traj_x, traj_y) in results.items():
    plt.plot(traj_x, traj_y, 'o-', label=name, linewidth=2, markersize=8)

plt.legend()
plt.xlabel('x')
plt.ylabel('y')
plt.title('不同优化器的收敛路径对比')
plt.colorbar(label='Loss')
plt.show()
```

### 管理学习率调度

```py
# 学习率衰减策略
optimizer = optim.SGD(model.parameters(), lr=0.1)

# 方法 1: StepLR - 每 N 个 epoch 衰减一次
scheduler = optim.lr_scheduler.StepLR(optimizer, step_size=30, gamma=0.1)

for epoch in range(100):
    train(...)
    validate(...)
    scheduler.step()  # 更新学习率
    
    if epoch % 30 == 0:
        print(f"Epoch {epoch}, LR: {optimizer.param_groups[0]['lr']}")
# Epoch 0, LR: 0.1
# Epoch 30, LR: 0.01
# Epoch 60, LR: 0.001

# 方法 2: ReduceLROnPlateau - 根据验证集 loss 自动调整
scheduler = optim.lr_scheduler.ReduceLROnPlateau(
    optimizer, 
    mode='min',      # loss 越小越好
    factor=0.5,      # 每次减半
    patience=10      # 10 个 epoch 不改善就减小
)

for epoch in range(100):
    train_loss = train(...)
    val_loss = validate(...)
    
    scheduler.step(val_loss)  # 根据验证集 loss 调整
```

### 管理参数组（不同层不同策略）

```py
# 对不同层使用不同的学习率
model = nn.Sequential(
    nn.Conv2d(3, 64, 3),    # 特征提取层
    nn.Conv2d(64, 128, 3),
    nn.Linear(128, 10)      # 分类层
)

optimizer = optim.SGD([
    # 特征提取层：小学习率（微调）
    {
        'params': model[0].parameters(),
        'lr': 0.001,
        'weight_decay': 0.0001
    },
    {
        'params': model[1].parameters(),
        'lr': 0.001,
        'weight_decay': 0.0001
    },
    # 分类层：大学习率（从头训练）
    {
        'params': model[2].parameters(),
        'lr': 0.01,
        'weight_decay': 0.0001
    }
])

# 训练过程中可以单独调整某个参数组
for param_group in optimizer.param_groups:
    print(f"LR: {param_group['lr']}, "
          f"Params: {len(list(param_group['params']))}")

# 动态调整：如果验证集效果不好，降低特征层学习率
if val_acc < threshold:
    optimizer.param_groups[0]['lr'] *= 0.5
    optimizer.param_groups[1]['lr'] *= 0.5
```

###  实现权重衰减（正则化）

```py
# L2 正则化（权重衰减）
optimizer = optim.SGD(
    model.parameters(), 
    lr=0.01,
    weight_decay=1e-4  # L2 正则化系数
)

# 等价于在损失函数中加 L2 惩罚项：
# Loss = original_loss + weight_decay * sum(param^2)

# 更新公式变为：
# θ = θ - lr * (g + weight_decay * θ)

# 好处：
# - 防止过拟合
# - 限制权重大小
# - 提高泛化能力
```

### 梯度裁剪（防止梯度爆炸）

```py
# RNN/LSTM 中常用
optimizer = optim.LSTM(model.parameters(), lr=0.001)

for data, target in dataloader:
    optimizer.zero_grad()
    output = model(data)
    loss = criterion(output, target)
    loss.backward()
    
    # 梯度裁剪
    torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
    # 如果梯度的范数超过 1.0，就按比例缩小
    
    optimizer.step()

# 或者按值裁剪
torch.nn.utils.clip_grad_value_(model.parameters(), clip_value=0.01)
# 将梯度限制在 [-0.01, 0.01] 范围内
```

### 保存和恢复优化状态

```py
# 训练中途保存
torch.save({
    'epoch': epoch,
    'model_state_dict': model.state_dict(),
    'optimizer_state_dict': optimizer.state_dict(),  # 保存优化器状态
    'loss': loss,
}, 'checkpoint.pth')

# 恢复训练
checkpoint = torch.load('checkpoint.pth')
model.load_state_dict(checkpoint['model_state_dict'])
optimizer.load_state_dict(checkpoint['optimizer_state_dict'])  # 恢复优化器
epoch = checkpoint['epoch']

# 优化器状态包括：
# - 动量缓冲（如果有 momentum）
# - 自适应学习率统计量（Adam 的 running averages）
# - 参数组的当前学习率
# - 其他优化器特定的状态变量
```

### 梯度累积（模拟大 batch）

```py
# GPU 显存不够大时的技巧
optimizer = optim.SGD(model.parameters(), lr=0.01)
accumulation_steps = 4  # 累积 4 个 batch 再更新

for i, (inputs, labels) in enumerate(dataloader):
    outputs = model(inputs)
    loss = criterion(outputs, labels) / accumulation_steps  # 损失也要除
    loss.backward()
    
    # 每 accumulation_steps 步才更新一次
    if (i + 1) % accumulation_steps == 0:
        optimizer.step()      # 用累积的梯度更新
        optimizer.zero_grad() # 清零
        
# 效果等价于 batch_size 扩大了 4 倍
# 但显存占用不变
```



## zero_grad()

### optimizer.zero_grad() 的作用对象

核心概念

```
optimizer.zero_grad()
```

作用： 将模型所有可学习参数的梯度清零
作用对象： 优化器管理的所有模型参数（parameters）

### 梯度数据存储在哪里？

梯度的存储位置
在 PyTorch 中，每个可学习的参数（nn.Parameter）都有一个 .grad 属性来存储梯度：

```py
import torch
import torch.nn as nn

# 定义一个简单的模型
model = nn.Sequential(
    nn.Linear(10, 5),   # 第 1 层
    nn.ReLU(),
    nn.Linear(5, 2)     # 第 2 层
)

# 查看模型的参数
for name, param in model.named_parameters():
    print(f"{name}:")
    print(f"  参数形状：{param.shape}")
    print(f"  梯度形状：{param.grad.shape if param.grad is not None else None}")
    print(f"  梯度值：{param.grad}")
```

具体示例：梯度在哪里

```py
import torch
import torch.nn as nn
import torch.optim as optim

# 1. 创建模型
model = nn.Linear(3, 2, bias=True)
# 结构：y = x * W^T + b
# W shape: (2, 3), b shape: (2,)

# 2. 查看参数（权重）
print("=== 模型参数 ===")
print(f"weight:\n{model.weight}")
# tensor([[ 0.1, -0.2,  0.3],
#         [-0.1,  0.4, -0.5]], requires_grad=True)
print(f"bias:\n{model.bias}")
# tensor([0.2, -0.3], requires_grad=True)

# 3. 初始时梯度为 None
print("\n=== 初始梯度 ===")
print(f"weight.grad: {model.weight.grad}")  # None
print(f"bias.grad: {model.bias.grad}")      # None

# 4. 前向传播 + 反向传播
x = torch.tensor([[1.0, 2.0, 3.0]])  # 输入
target = torch.tensor([[1.0, 0.0]])  # 目标值

output = model(x)                    # 前向传播
loss = nn.MSELoss()(output, target)  # 计算损失
loss.backward()                      # 反向传播 ← 梯度计算并存储

# 5. 现在梯度有值了！
print("\n=== 反向传播后的梯度 ===")
print(f"weight.grad:\n{model.weight.grad}")
# tensor([[-0.4, -0.8, -1.2],
#         [ 0.4,  0.8,  1.2]])
print(f"bias.grad:\n{model.bias.grad}")
# tensor([-0.4, 0.4])

# 6. 梯度存储在参数的 .grad 属性中
print("\n=== 梯度的存储位置 ===")
print(f"梯度存储在 model.weight.grad: {id(model.weight.grad)}")
print(f"梯度存储在 model.bias.grad: {id(model.bias.grad)}")
```



### 梯度数据存储和管理

```
┌─────────────────────────────────────────────┐
│              Neural Network                 │
│                                             │
│  ┌──────────────┐    ┌──────────────┐      │
│  │  Layer 1     │    │  Layer 2     │      │
│  │  weight [5×10]│    │  weight [2×5] │      │
│  │  bias [5]    │    │  bias [2]    │      │
│  │  .grad ──────┼───→│  .grad ──────┼───→ │
│  └──────────────┘    └──────────────┘      │
│         ↓                    ↓               │
│    存储梯度             存储梯度             │
│    (5×10 张量)          (2×5 张量)           │
└─────────────────────────────────────────────┘
         ↓                    ↓
    ┌────────────────────────────────┐
    │       Optimizer                │
    │  param_groups: [               │
    │    {params: [weight1, bias1,   │
    │              weight2, bias2]}  │
    │  ]                             │
    └────────────────────────────────┘
              ↓
    optimizer.zero_grad()
              ↓
    遍历所有 param.grad
    调用 param.grad.zero_()
```

### 问题1

我想问一下，梯度数据在model.weight.grad中，那为什么用一个优化器optimizer.zero_grad()来给它清零？直接令model.weight.grad等于零不就行了吗？

原因 1：模型可能有成百上千个参数

```py
import torchvision.models as models

# 一个典型的 ResNet-18 模型
model = models.resnet18()

# 数一下有多少个参数层
param_count = 0
for name, param in model.named_parameters():
    param_count += 1
    print(f"{name}: shape={param.shape}, grad_shape={param.grad.shape if param.grad is not None else None}")

print(f"\n总参数层数：{param_count}")
# 大约 60+ 个参数层（每个 Conv2d、Linear 都有 weight 和 bias）

# ❌ 如果要手动清零，你需要写：
model.conv1.weight.grad = torch.zeros_like(model.conv1.weight.grad)
model.conv1.bias.grad = torch.zeros_like(model.conv1.bias.grad)
model.bn1.weight.grad = torch.zeros_like(model.bn1.weight.grad)
model.bn1.bias.grad = torch.zeros_like(model.bn1.bias.grad)
model.layer1[0].conv1.weight.grad = ...
model.layer1[0].conv1.bias.grad = ...
# ... 还要写 50+ 行 😱

# ✅ 用 optimizer.zero_grad() 一行搞定！
optimizer = optim.SGD(model.parameters(), lr=0.01)
optimizer.zero_grad()
```

原因 2：zero_grad() 是原地操作，更高效

```py
import time

model = nn.Sequential(*[nn.Linear(1000, 1000) for _ in range(10)])
optimizer = optim.SGD(model.parameters(), lr=0.01)

# 方法 1: 手动创建新张量（慢，占用更多内存）
def manual_zero_grad_v1(model):
    for param in model.parameters():
        if param.grad is not None:
            param.grad = torch.zeros_like(param.grad)  # 创建新张量

# 方法 2: 使用 zero_() 原地操作（快，不创建新对象）
def manual_zero_grad_v2(model):
    for param in model.parameters():
        if param.grad is not None:
            param.grad.zero_()  # 原地修改，不创建新张量

# 方法 3: 使用 optimizer.zero_grad()
optimizer.zero_grad()

# 性能对比
x = torch.randn(32, 1000)
target = torch.randn(32, 1000)

# 测试方法 1
start = time.time()
for _ in range(100):
    output = model(x)
    loss = ((output - target) ** 2).mean()
    loss.backward()
    manual_zero_grad_v1(model)
print(f"方法 1 (创建新张量): {time.time() - start:.4f}秒")

# 测试方法 2
start = time.time()
for _ in range(100):
    output = model(x)
    loss = ((output - target) ** 2).mean()
    loss.backward()
    manual_zero_grad_v2(model)
print(f"方法 2 (原地操作): {time.time() - start:.4f}秒")

# 测试方法 3
optimizer = optim.SGD(model.parameters(), lr=0.01)
start = time.time()
for _ in range(100):
    output = model(x)
    loss = ((output - target) ** 2).mean()
    loss.backward()
    optimizer.zero_grad()
print(f"方法 3 (optimizer): {time.time() - start:.4f}秒")

# 结果通常是：方法 2 ≈ 方法 3 > 方法 1
# 原地操作更快，因为不分配新内存
```

原因 3：优化器知道要管理哪些参数

```py
# 有些参数可能不需要梯度（冻结层）
model = nn.Sequential(
    nn.Linear(10, 5),
    nn.ReLU(),
    nn.Linear(5, 2)
)

# 冻结第一层
for param in model[0].parameters():
    param.requires_grad = False

# 创建优化器（只优化第二层）
optimizer = optim.SGD(model[2].parameters(), lr=0.01)

# 前向 + 反向
x = torch.randn(1, 10)
output = model(x)
loss = output.sum()
loss.backward()

# 检查梯度
print("第一层（冻结）:")
print(f"  weight.grad: {model[0].weight.grad}")  # 有梯度，但我们不想更新它
print("第二层（可训练）:")
print(f"  weight.grad: {model[2].weight.grad}")  # 有梯度

# ❌ 如果手动清零，你需要知道哪些要清哪些不要
if model[0].weight.grad is not None:
    model[0].weight.grad.zero_()  # 其实不需要清，因为我们不优化它
if model[2].weight.grad is not None:
    model[2].weight.grad.zero_()  # 这个才需要清

# ✅ 用 optimizer.zero_grad() 自动只清它管理的参数
optimizer.zero_grad()
# 只会清零 model[2] 的梯度，不会管 model[0]
```

原因 4：支持参数组（不同学习率）

```py
# 对不同层使用不同学习率
model = nn.Sequential(
    nn.Linear(10, 5),  # 特征提取层
    nn.Linear(5, 2)    # 分类层
)

optimizer = optim.SGD([
    {'params': model[0].parameters(), 'lr': 0.001},  # 小学习率
    {'params': model[1].parameters(), 'lr': 0.01}    # 大学习率
])

# optimizer.zero_grad() 会正确处理所有参数组
optimizer.zero_grad()

# 如果手动清零，你需要遍历所有参数组
for param_group in optimizer.param_groups:
    for param in param_group['params']:
        if param.grad is not None:
            param.grad.zero_()
# 这其实就是 optimizer.zero_grad() 的内部实现！
```

# 优化器

## Adam()

### 基本用法

参数说明

```py
optim.Adam(params, lr=0.001, betas=(0.9, 0.999), eps=1e-08, weight_decay=0, amsgrad=False)
```

示例代码

```py
optimizerD = optim.Adam(netD.parameters(), lr=lr, betas=(beta1, 0.999))
optimizerG = optim.Adam(netG.parameters(), lr=lr, betas=(beta1, 0.999))
```

根据上下文：

- params = netD.parameters() —— 判别器的所有可学习参数
- lr = 0.0002 —— 学习率（第 55 行定义）
- betas = (beta1, 0.999) = (0.5, 0.999) —— 指数衰减率（第 58 行 beta1=0.5）
- eps = 1e-08（默认）—— 数值稳定性常数
- weight_decay = 0（默认）—— L2 正则化系数
- amsgrad = False（默认）—— 不使用 AMSGrad 变体

### Adam 优化器的核心思想

1. Adam 的名称由来

Adam = Adaptive Moment Estimation（自适应矩估计）
它结合了两种优化算法的优点：

- Momentum（动量法）：利用梯度的一阶矩（均值）
- RMSProp：利用梯度的二阶矩（未中心化的方差）

2. 为什么需要 Adam？

让我们对比几种优化器的发展历程：

（1）SGD（随机梯度下降）

```py
# 简单但低效
w = w - lr * gradient
```

问题：
❌ 学习率固定，难以调整
❌ 容易陷入局部最优
❌ 在鞍点处停滞不前

（2）SGD with Momentum（带动量的 SGD

```
# 引入"惯性"
v = β₁ * v + (1 - β₁) * gradient
w = w - lr * v
```

改进：
✅ 加速收敛（积累同方向梯度）
✅ 减少震荡（平滑更新方向）
问题：
⚠️ 学习率仍需手动调整
⚠️ 对所有参数使用相同学习率

（3）RMSProp（均方根传播）

```
# 自适应学习率
s = β₂ * s + (1 - β₂) * gradient²
w = w - lr * gradient / (√s + ε)
```

改进：
✅ 每个参数有独立的学习率
✅ 梯度大的参数学习率小，梯度小的参数学习率大
问题：
⚠️ 没有利用动量信息

（4）Adam（集大成者）

```
# 同时利用一阶矩和二阶矩
m = β₁ * m + (1 - β₁) * gradient      # 一阶矩（动量）
v = β₂ * v + (1 - β₂) * gradient²     # 二阶矩（自适应学习率）
w = w - lr * m̂ / (√v̂ + ε)            # 结合两者
```

优势：
✅ 自适应学习率（每个参数独立）
✅ 动量加速（利用历史梯度）
✅ 偏差修正（解决初始偏差）
✅ 几乎无需调参（默认参数适用于大多数场景）

### Adam 的数学原理（详细推导）

1. 符号定义

|  符号   |                   含义                   |
| :-----: | :--------------------------------------: |
|    θ    |          模型参数（权重和偏置）          |
|   g_t   |           时刻 t 的梯度 ∂L/∂θ            |
|   m_t   |     一阶矩估计（梯度的指数移动平均）     |
|   v_t   |   二阶矩估计（梯度平方的指数移动平均）   |
|   β₁    | 一阶矩衰减率（默认 0.9，你的代码用 0.5） |
|   β₂    |        二阶矩衰减率（默认 0.999）        |
| lr 或 α |       学习率（你的代码用 0.0002）        |
|    ε    |      小常数，防止除零（默认 1e-8）       |
|    t    |            时间步（迭代次数）            |

2. Adam 算法的完整步骤

步骤 1：计算梯度

```
g_t = ∂L/∂θ_t
```

这是通过反向传播得到的当前参数的梯度。

步骤 2：更新一阶矩估计（动量）

```
m_t = β₁ · m_{t-1} + (1 - β₁) · g_t
```

物理意义：

- m_t 是梯度的指数移动平均（Exponential Moving Average, EMA）
- 类似于物理学中的"速度"或"动量"
- β₁ 控制历史梯度的影响程度

展开理解：

```
m_t = β₁ · m_{t-1} + (1 - β₁) · g_t
    = β₁ · [β₁ · m_{t-2} + (1 - β₁) · g_{t-1}] + (1 - β₁) · g_t
    = β₁² · m_{t-2} + β₁(1 - β₁) · g_{t-1} + (1 - β₁) · g_t
    = ...
    = (1 - β₁) · Σ β₁^{t-i} · g_i  （从 i=1 到 t）
```

这是一个加权平均，最近的梯度权重更大：

```
你的代码中 β₁ = 0.5：
m_t = 0.5 · m_{t-1} + 0.5 · g_t

权重分布：
g_t:     0.5
g_{t-1}: 0.25
g_{t-2}: 0.125
g_{t-3}: 0.0625
...

有效窗口大小 ≈ 1/(1-β₁) = 1/0.5 = 2 步
```

步骤 3：更新二阶矩估计（自适应学习率）

```
v_t = β₂ · v_{t-1} + (1 - β₂) · g_t²
```

注意：这里是 g_t²（逐元素平方），不是 g_t！
物理意义：

- v_t 是梯度平方的指数移动平均
- 近似于梯度的未中心化方差
- 用于自适应调整每个参数的学习率

展开理解：

```
你的代码中 β₂ = 0.999：
v_t = 0.999 · v_{t-1} + 0.001 · g_t²

有效窗口大小 ≈ 1/(1-β₂) = 1/0.001 = 1000 步
```

这意味着 v_t 会考虑最近约 1000 步的梯度信息。

步骤 4：偏差修正（Bias Correction）

```
m̂_t = m_t / (1 - β₁^t)
v̂_t = v_t / (1 - β₂^t)
```

为什么需要偏差修正？
在训练初期（t 很小），m_t 和 v_t 会偏向 0：

```
初始化：m_0 = 0, v_0 = 0

t = 1 时：
m_1 = β₁ · 0 + (1 - β₁) · g_1 = (1 - β₁) · g_1

如果 β₁ = 0.9：
m_1 = 0.1 · g_1  ← 远小于真实值 g_1！

偏差修正后：
m̂_1 = m_1 / (1 - β₁^1) = 0.1·g_1 / 0.1 = g_1  ✓
```

随着 t 增大，修正项趋近于 1：

```
β₁ = 0.9 时：
t=1:   1 - β₁^1  = 1 - 0.9    = 0.1
t=10:  1 - β₁^10 = 1 - 0.349  = 0.651
t=50:  1 - β₁^50 = 1 - 0.005  = 0.995  ← 接近 1，修正作用变小
t=100: 1 - β₁^100 ≈ 1         ← 几乎不需要修正
```

步骤 5：更新参数

```
θ_{t+1} = θ_t - lr · m̂_t / (√v̂_t + ε)
```

逐项分析：

```
更新量 Δθ = lr · m̂_t / (√v̂_t + ε)
           ↑    ↑        ↑
        学习率 动量   自适应缩放
```

- 分子 m̂_t：带方向的动量（决定更新方向）_
- 分母 √v̂_t + ε：自适应缩放因子（决定更新幅度）
- ε：防止除零，保证数值稳定

3. 具体数值示例

让我们用一个简化的例子演示 Adam 的工作过程。
设定：

- 单个参数 θ，初始值 θ_0 = 0
- 学习率 lr = 0.001
- β₁ = 0.9, β₂ = 0.999, ε = 1e-8
- 假设梯度序列：g = [0.1, 0.2, -0.1, 0.15, ...]

迭代 1（t=1）

```
梯度：g_1 = 0.1

步骤 1：更新一阶矩
m_1 = 0.9 × 0 + 0.1 × 0.1 = 0.01

步骤 2：更新二阶矩
v_1 = 0.999 × 0 + 0.001 × 0.1² = 0.00001

步骤 3：偏差修正
m̂_1 = 0.01 / (1 - 0.9^1) = 0.01 / 0.1 = 0.1
v̂_1 = 0.00001 / (1 - 0.999^1) = 0.00001 / 0.001 = 0.01

步骤 4：更新参数
θ_1 = 0 - 0.001 × 0.1 / (√0.01 + 1e-8)
    = 0 - 0.001 × 0.1 / 0.1
    = 0 - 0.001
    = -0.001
```

迭代 2（t=2）

```
梯度：g_2 = 0.2

步骤 1：更新一阶矩
m_2 = 0.9 × 0.01 + 0.1 × 0.2 = 0.009 + 0.02 = 0.029

步骤 2：更新二阶矩
v_2 = 0.999 × 0.00001 + 0.001 × 0.2²
    = 0.00000999 + 0.00004
    = 0.00004999

步骤 3：偏差修正
m̂_2 = 0.029 / (1 - 0.9^2) = 0.029 / 0.19 ≈ 0.1526
v̂_2 = 0.00004999 / (1 - 0.999^2) = 0.00004999 / 0.001999 ≈ 0.025

步骤 4：更新参数
θ_2 = -0.001 - 0.001 × 0.1526 / (√0.025 + 1e-8)
    = -0.001 - 0.001 × 0.1526 / 0.1581
    = -0.001 - 0.000965
    = -0.001965
```

迭代 3（t=3）

```
梯度：g_3 = -0.1（注意：梯度方向改变了！）

步骤 1：更新一阶矩
m_3 = 0.9 × 0.029 + 0.1 × (-0.1)
    = 0.0261 - 0.01
    = 0.0161

步骤 2：更新二阶矩
v_3 = 0.999 × 0.00004999 + 0.001 × (-0.1)²
    = 0.00004994 + 0.00001
    = 0.00005994

步骤 3：偏差修正
m̂_3 = 0.0161 / (1 - 0.9^3) = 0.0161 / 0.271 ≈ 0.0594
v̂_3 = 0.00005994 / (1 - 0.999^3) ≈ 0.00005994 / 0.002997 ≈ 0.02

步骤 4：更新参数
θ_3 = -0.001965 - 0.001 × 0.0594 / (√0.02 + 1e-8)
    = -0.001965 - 0.001 × 0.0594 / 0.1414
    = -0.001965 - 0.00042
    = -0.002385
```

观察：

- 虽然 g_3 = -0.1 是负梯度，但由于动量 m_3 仍为正（历史梯度累积），参数继续向负方向更新
- 但更新幅度减小了（从 0.000965 降到 0.00042），因为动量在衰减
