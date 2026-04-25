# 介绍

**torch.nn** 是 PyTorch 中专门用于构建神经网络的模块。它提供了各种神经网络层（如卷积层、全连接层）、激活函数、损失函数等。它的核心是 **nn.Module** 类，所有的神经网络模型都需要继承这个类。

# 常见API

## DataParallel() 

### 基本用法

参数说明

```py
nn.DataParallel(module, device_ids=None, output_device=None, dim=0)
```

示例代码

```py
netD = nn.DataParallel(netD, list(range(ngpu)))
```

这里：

- module = netD —— 要被并行化的模型（判别器）
- device_ids = list(range(ngpu)) —— 使用的 GPU 设备 ID 列表
  - 如果 ngpu = 4，则 device_ids = [0, 1, 2, 3]
  - 表示使用 GPU 0、1、2、3 共 4 张显卡
- output_device（可选）—— 输出所在的设备，默认为 device_ids[0]
- dim（可选）—— 批次维度，默认为 0

### DataParallel 的工作原理（详细数据流）

1. 核心思想：数据并行（Data Parallelism）
    DataParallel 采用单进程多线程的方式，在多个 GPU 上并行处理同一个模型的不同数据批次。



2. 完整的数据流向
   假设你有 4 张 GPU，批次大小为 128：

```
主进程（CPU / GPU 0）
         ↓
┌────────────────────────────────┐
│   输入数据 (128, 3, 64, 64)    │
└────────────────────────────────┘
         ↓
    【步骤 1：散射 Scatter】
    将批次数据均分到各个 GPU
         ↓
┌─────────────┬─────────────┬─────────────┬─────────────┐
│  GPU 0      │  GPU 1      │  GPU 2      │  GPU 3      │
│ (32,3,64,64)│ (32,3,64,64)│ (32,3,64,64)│ (32,3,64,64)│
│             │             │             │             │
│  模型副本   │  模型副本   │  模型副本   │  模型副本   │
│  (相同权重) │  (相同权重) │  (相同权重) │  (相同权重) │
│             │             │             │             │
│  前向传播   │  前向传播   │  前向传播   │  前向传播   │
│             │             │             │             │
│  输出       │  输出       │  输出       │  输出       │
│ (32,1,1,1)  │ (32,1,1,1)  │ (32,1,1,1)  │ (32,1,1,1)  │
└─────────────┴─────────────┴─────────────┴─────────────┘
         ↓
    【步骤 2：收集 Gather】
    将所有 GPU 的输出合并到 GPU 0
         ↓
┌────────────────────────────────┐
│   输出数据 (128, 1, 1, 1)      │
│   （在 GPU 0 上）               │
└────────────────────────────────┘
         ↓
    【步骤 3：计算损失】
    在 GPU 0 上计算损失函数
         ↓
    【步骤 4：反向传播】
    自动在各 GPU 上计算梯度
         ↓
    【步骤 5：梯度归约 Reduce】
    将所有 GPU 的梯度累加到 GPU 0
         ↓
    【步骤 6：更新权重】
    优化器在 GPU 0 上更新模型权重
         ↓
    【步骤 7：广播 Broadcast】
    将更新后的权重同步到其他 GPU
```

3. 逐步详解
步骤 1：Scatter（数据分散）

```py
# 原始输入
input_batch = torch.randn(128, 3, 64, 64).cuda()

# DataParallel 自动将数据切分
# GPU 0: input_batch[0:32]
# GPU 1: input_batch[32:64]
# GPU 2: input_batch[64:96]
# GPU 3: input_batch[96:128]
```

注意：

- 按第 0 维（批次维）均匀分割
- 如果批次大小不能被 GPU 数量整除，最后一个 GPU 会少一些数据
- 例如：130 个样本分给 4 个 GPU → [33, 33, 32, 32]

步骤 2：Replicate（模型复制）

```
# 在每个 GPU 上创建模型的副本
# 所有副本共享相同的初始权重
GPU_0: Discriminator(weights=W)
GPU_1: Discriminator(weights=W)
GPU_2: Discriminator(weights=W)
GPU_3: Discriminator(weights=W)
```

关键点：

- 模型副本是临时创建的，不是永久存储
- 每次前向传播都会重新复制（有性能开销）

步骤 3：Parallel Apply（并行前向传播）
每个 GPU 独立执行前向传播：

```py
# 伪代码
outputs = []
for i, gpu in enumerate(gpus):
    output_i = model_replica_i(input_chunk_i)
    outputs.append(output_i)

# 实际是并行执行的，速度更快
```

步骤 4：Gather（结果收集）

```py
# 将所有 GPU 的输出收集到主 GPU（默认 GPU 0）
output_gathered = torch.cat(outputs, dim=0)
# 形状: (128, 1, 1, 1)
```

步骤 5-7：反向传播和权重更新

```py
# 计算损失（在 GPU 0 上）
loss = criterion(output_gathered, labels)

# 反向传播
loss.backward()
# 这一步会自动：
# 1. 在每个 GPU 上计算局部梯度
# 2. 将所有梯度累加到 GPU 0 的主模型上

# 优化器更新权重（在 GPU 0 上）
optimizer.step()

# DataParallel 自动将更新后的权重广播到其他 GPU
```

# Module

## apply()

一、apply() 方法的来源

apply() 是 nn.Module 类的方法

```py
# PyTorch 源码位置：torch/nn/modules/module.py
class Module:
    def apply(self, fn):
        """
        将函数 fn 递归地应用到模块及其所有子模块上
        """
        for module in self.children():
            module.apply(fn)  # 递归调用子模块
        fn(self)              # 对当前模块应用函数
        return self
```

继承关系：

```
nn.Module (基类)
    ↑
    |── Generator(nn.Module)      # 你的生成器
    |── Discriminator(nn.Module)  # 你的判别器
    |── nn.Sequential             # 容器模块
    |── nn.Conv2d                 # 卷积层
    |── nn.BatchNorm2d            # 批归一化层
    |── ... 所有 PyTorch 模块
```

二、apply() 的作用机制（详细数据流）

1. 核心功能：递归遍历

apply() 会深度优先遍历模型的所有子模块，并对每个模块执行传入的函数。

2. 在你的代码中的执行过程

```py
def weights_init(m):
    classname = m.__class__.__name__
    if classname.find('Conv') != -1:
        nn.init.normal_(m.weight.data, 0.0, 0.02)
    elif classname.find('BatchNorm') != -1:
        nn.init.normal_(m.weight.data, 1.0, 0.02)
        nn.init.constant_(m.bias.data, 0)

netG.apply(weights_init)
```

遍历顺序（深度优先）：

```
netG (Generator)
  ↓
  netG.main (Sequential)
    ↓
    [0] ConvTranspose2d(nz→512, k=4, s=1, p=0)
        → weights_init() 检测到 "Conv" → 初始化 weight
    ↓
    [1] BatchNorm2d(512)
        → weights_init() 检测到 "BatchNorm" → 初始化 weight 和 bias
    ↓
    [2] ReLU(inplace)
        → weights_init() 检测到无 Conv/BatchNorm → 跳过
    ↓
    [3] ConvTranspose2d(512→256, k=4, s=2, p=1)
        → weights_init() 检测到 "Conv" → 初始化 weight
    ↓
    [4] BatchNorm2d(256)
        → weights_init() 检测到 "BatchNorm" → 初始化 weight 和 bias
    ↓
    [5] ReLU(inplace)
        → 跳过
    ↓
    [6] ConvTranspose2d(256→128, k=4, s=2, p=1)
        → 初始化 weight
    ↓
    [7] BatchNorm2d(128)
        → 初始化 weight 和 bias
    ↓
    [8] ReLU(inplace)
        → 跳过
    ↓
    [9] ConvTranspose2d(128→64, k=4, s=2, p=1)
        → 初始化 weight
    ↓
    [10] BatchNorm2d(64)
         → 初始化 weight 和 bias
    ↓
    [11] ReLU(inplace)
         → 跳过
    ↓
    [12] ConvTranspose2d(64→3, k=4, s=2, p=1)
         → 初始化 weight
    ↓
    [13] Tanh()
         → 跳过（无参数）
```

三、底层实现原理（源码级解析）

1. apply() 的完整源码

```py
# torch/nn/modules/module.py
class Module:
    def apply(self: T, fn: Callable[['Module'], None]) -> T:
        r"""
        将函数 fn 递归地应用到模块及其所有子模块上
        
        Args:
            fn: 一个接受 Module 作为参数的函数
            
        Returns:
            Module: self
        """
        for module in self.children():
            module.apply(fn)  # ← 递归调用子模块
        fn(self)              # ← 对当前模块应用函数
        return self
    
    def children(self) -> Iterator['Module']:
        """返回直接子模块的迭代器"""
        for name, module in self._modules.items():
            if module is not None:
                yield module
```

2. 递归调用的执行栈

让我们模拟 netG.apply(weights_init) 的执行过程：

```
# 第 1 层调用
apply(netG)
  ├── 遍历 netG.children() → [netG.main]
  │     ↓
  │   # 第 2 层调用
  │   apply(netG.main)
  │     ├── 遍历 netG.main.children() → [ConvTranspose2d, BatchNorm2d, ReLU, ...]
  │     │     ↓
  │     │   # 第 3 层调用（以第一个 ConvTranspose2d 为例）
  │     │   apply(ConvTranspose2d_0)
  │     │     ├── 遍历 children() → []  (叶子节点，无子模块)
  │     │     └── weights_init(ConvTranspose2d_0)  ← 执行初始化
  │     │           ├── classname = "ConvTranspose2d"
  │     │           ├── 检测到 "Conv" → True
  │     │           └── nn.init.normal_(weight.data, 0.0, 0.02)
  │     │
  │     │     ↓
  │     │   # 第 3 层调用（第二个模块 BatchNorm2d）
  │     │   apply(BatchNorm2d_0)
  │     │     ├── 遍历 children() → []
  │     │     └── weights_init(BatchNorm2d_0)
  │     │           ├── classname = "BatchNorm2d"
  │     │           ├── 检测到 "Conv" → False
  │     │           ├── 检测到 "BatchNorm" → True
  │     │           ├── nn.init.normal_(weight.data, 1.0, 0.02)
  │     │           └── nn.init.constant_(bias.data, 0)
  │     │
  │     │     ↓
  │     │   # 继续处理后续模块...
  │     │
  │     └── weights_init(netG.main)  ← Sequential 本身无参数，跳过
  │
  └── weights_init(netG)  ← Generator 本身无直接参数，跳过
```

3. 关键设计模式：访问者模式（Visitor Pattern）

apply() 实现了经典的访问者设计模式

```
┌──────────────┐
│   Visitor    │  weights_init 函数（访问者）
│  (Function)  │
└──────┬───────┘
       │ 访问
       ↓
┌──────────────┐
│  Element     │  nn.Module 及其子类（被访问者）
│  (Module)    │
└──────┬───────┘
       │ 递归遍历
       ↓
┌──────────────┐
│  Children    │  子模块列表
│  (Modules)   │
└──────────────┘
```

优势：
✅ 解耦：初始化逻辑与模型结构分离
✅ 灵活：可以传入不同的初始化函数
✅ 通用：适用于任何 nn.Module 子类

四、weights_init 函数的详细分析

1. 类型检测机制

```py
def weights_init(m):
    classname = m.__class__.__name__  # 获取类名字符串
    
    # 方法 1：字符串匹配（你的代码使用的方式）
    if classname.find('Conv') != -1:
        # 匹配所有包含 "Conv" 的类名
        # 例如: Conv2d, ConvTranspose2d, Conv3d, etc.
        nn.init.normal_(m.weight.data, 0.0, 0.02)
    
    elif classname.find('BatchNorm') != -1:
        # 匹配所有包含 "BatchNorm" 的类名
        # 例如: BatchNorm1d, BatchNorm2d, BatchNorm3d
        nn.init.normal_(m.weight.data, 1.0, 0.02)
        nn.init.constant_(m.bias.data, 0)
```

2. 初始化函数的底层实现

nn.init.normal_() 的实现

```py
# torch/nn/init.py
def normal_(tensor, mean=0.0, std=1.0):
    """
    用正态分布填充张量（原地操作）
    
    Args:
        tensor: 要初始化的张量
        mean: 正态分布的均值
        std: 正态分布的标准差
    """
    with torch.no_grad():  # 不记录梯度
        return tensor.normal_(mean, std)  # 调用 Tensor 的原地方法
```

数学原理：

```
weight ~ N(μ=0.0, σ²=0.02²)

即从均值为 0、标准差为 0.02 的正态分布中采样
```

nn.init.constant_() 的实现

```py
def constant_(tensor, val=0):
    """
    用常数值填充张量（原地操作）
    """
    with torch.no_grad():
        return tensor.fill_(val)
```

五、为什么要自定义权重初始化？

1. 默认初始化的问题

PyTorch 的默认初始化方式因层而异：

```py
# Conv2d 的默认初始化
# 使用 Kaiming Uniform 初始化
bound = 1 / sqrt(fan_in)
weight ~ Uniform(-bound, bound)

# BatchNorm 的默认初始化
weight = 1.0  (全 1)
bias = 0.0    (全 0)
```

2. DCGAN 为什么需要特殊初始化？

DCGAN 论文明确指出：
	 "All weights were initialized from a zero-centered Normal distribution with standard deviation 0.02."
原因：

（1）稳定 GAN 训练

GAN 对初始化非常敏感：

- 权重过大：梯度爆炸，训练不稳定
- 权重过小：梯度消失，学习缓慢
- 标准差 0.02：经验值，能平衡稳定性和学习能力

（2）BatchNorm 的特殊初始化

```py
nn.init.normal_(m.weight.data, 1.0, 0.02)  # γ 初始化为 1
nn.init.constant_(m.bias.data, 0)          # β 初始化为 0
```

为什么 γ 初始化为 1？

- BatchNorm 的输出：y = γ·x̂ + β
- 如果 γ=1, β=0，则 y = x̂（恒等映射）
- 这样在训练初期，BatchNorm 不会干扰网络的学习

（3）对比实验

```py
# 方案 1：默认初始化
netG_default = Generator(ngpu)
# 结果：训练不稳定，可能模式崩溃

# 方案 2：DCGAN 初始化
netG_dcgan = Generator(ngpu)
netG_dcgan.apply(weights_init)
# 结果：训练稳定，生成质量高
```

六、apply() 的其他应用场景

1. 冻结某些层

```py
def freeze_bn(m):
    if isinstance(m, nn.BatchNorm2d):
        m.eval()  # 设置为评估模式
        for param in m.parameters():
            param.requires_grad = False  # 冻结参数

netG.apply(freeze_bn)
```

2. 打印模型信息

```py
def print_module_info(m):
    classname = m.__class__.__name__
    if hasattr(m, 'weight'):
        print(f"{classname}: weight shape = {m.weight.shape}")
    if hasattr(m, 'bias') and m.bias is not None:
        print(f"{classname}: bias shape = {m.bias.shape}")

netG.apply(print_module_info)
# 输出:
# ConvTranspose2d: weight shape = torch.Size([100, 512, 4, 4])
# BatchNorm2d: weight shape = torch.Size([512])
# BatchNorm2d: bias shape = torch.Size([512])
# ...
```

3. 计算参数量

```py
def count_params(m):
    if hasattr(m, 'weight'):
        m.num_params = m.weight.numel()
        if hasattr(m, 'bias') and m.bias is not None:
            m.num_params += m.bias.numel()
    else:
        m.num_params = 0

netG.apply(count_params)
total_params = sum(m.num_params for m in netG.modules())
print(f"Total parameters: {total_params:,}")
```

4. 应用不同的初始化策略

```py
def xavier_init(m):
    if isinstance(m, (nn.Conv2d, nn.ConvTranspose2d, nn.Linear)):
        nn.init.xavier_uniform_(m.weight.data)
        if m.bias is not None:
            nn.init.zeros_(m.bias.data)

netG.apply(xavier_init)  # 使用 Xavier 初始化
```



# 卷积

## Conv2d()

用于在多个输入平面上应用二维卷积

### 基本用法

参数说明

```py
nn.Conv2d(in_channels, out_channels, kernel_size, stride=1, padding=0, dilation=1, groups=1, bias=True)
```

示例代码

```py
nn.Conv2d(nc, ndf, 4, 2, 1, bias=False)
```

根据上下文，这里：

- nc = 3（输入通道数，RGB三通道）
- ndf = 64（输出通道数，即卷积核数量）
- kernel_size = 4（卷积核大小 4×4）
- stride = 2（步长为2）
- padding = 1（填充1个像素）
- bias = False（不使用偏置项，因为后面接了 BatchNorm）

数据流向
输入 → 卷积层 → 输出

- 输入形状: (batch_size, 3, 64, 64) —— 批次大小×3通道×64高×64宽
- 输出形状: (batch_size, 64, 32, 32) —— 批次大小×64通道×32高×32宽

### **数学实现原理**

1. 卷积运算的核心思想

卷积是通过滑动窗口的方式，用一组可学习的滤波器（卷积核）在输入特征图上提取局部特征。

2. 具体计算步骤假设我们有一个简化的例子来理解这个过程：输入数据

假设我们有一个简化的例子来理解这个过程：

输入数据

- 输入张量形状：(C_in, H_in, W_in) = (3, 64, 64)
- 这表示一张 64×64 的 RGB 彩色图像

卷积核（滤波器）

- 每个卷积核的形状：(C_in, K, K) = (3, 4, 4)
  - 为什么是 3 通道？因为要和输入的 3 个通道对应
    - 4×4 是卷积核的空间尺寸
  - 卷积核数量：C_out = 64 个
  - 所以总共有 64 个 4×4×3 的卷积核

输出特征图

- 输出形状计算公式：

```
  H_out = ⌊(H_in + 2×padding - kernel_size) / stride⌋ + 1
  W_out = ⌊(W_in + 2×padding - kernel_size) / stride⌋ + 1
```

代入数值：

```
  H_out = ⌊(64 + 2×1 - 4) / 2⌋ + 1 = ⌊62/2⌋ + 1 = 31 + 1 = 32
  W_out = ⌊(64 + 2×1 - 4) / 2⌋ + 1 = 32
```

最终输出：(64, 32, 32) —— 64个通道的32×32特征图

**为什么设置 bias=False？**

在这个 DCGAN 的判别器中，卷积层后面紧跟 BatchNorm2d：

```py
nn.Conv2d(nc, ndf, 4, 2, 1, bias=False),  # 无偏置
nn.LeakyReLU(0.2, inplace=True),
```

原因：

- BatchNorm 会对输出进行归一化：y = γ·(x - μ) / σ + β
- 这里的 β 已经是可学习的偏置项
- 如果卷积层也有偏置 b，那么：BatchNorm(Conv(x) + b)
- 由于 BatchNorm 会减去均值 μ，卷积的偏置 b 会被抵消掉
- 因此卷积层的偏置是冗余的，去掉可以减少参数量

## ConvTranspose2d()

**nn.ConvTranspose2d** 是PyTorch中用于进行二维转置卷积操作的模块，也被称为反卷积（deconvolution）。这个模块可以看作是对 **nn.Conv2d** 的梯度操作，或者说是对其输入的一种逆操作。转置卷积通常用于将数据从较低维空间映射到较高维空间，常见于生成模型和某些类型的解码器中。

### 基本用法

参数说明

```py
torch.nn.ConvTranspose2d(
    in_channels,        # 输入通道数
    out_channels,       # 输出通道数
    kernel_size,        # 卷积核大小（int 或 tuple）
    stride=1,           # 步长，默认 1
    padding=0,          # 输入填充，默认 0
    output_padding=0,   # 输出额外填充，默认 0
    groups=1,           # 分组卷积，默认 1
    bias=True,          # 是否使用偏置，默认 True
    dilation=1,         # 空洞卷积率，默认 1
    padding_mode='zeros', # 填充模式，默认 'zeros'
    device=None,
    dtype=None
)
```

示例代码

```py
nn.ConvTranspose2d(nz, ngf * 8, 4, 1, 0, bias=False)
```

根据上下文：

- nz = 100（输入通道数，即噪声向量的维度）
- ngf * 8 = 512（输出通道数）
- kernel_size = 4（卷积核大小 4×4）
- stride = 1（步长为 1）
- padding = 0（无填充）
- bias = False（不使用偏置，因为后面接了 BatchNorm）

### 数学原理（核心概念详解）

1. 什么是转置卷积？

转置卷积（Transposed Convolution），也被称为：
❌ 反卷积（Deconvolution）—— 这个名称不准确，容易误解
✅ 上采样卷积（UpSampling Convolution）
✅ 分数步长卷积（Fractionally-strided Convolution）
核心作用：将小尺寸的特征图放大为大尺寸的特征图，是普通卷积的"逆操作"。

2. 与普通卷积的对比

普通卷积（Conv2d）：下采样

```
# 判别器中使用
nn.Conv2d(in_channels=3, out_channels=64, kernel_size=4, stride=2, padding=1)

输入: (N, 3, 64, 64)   →   输出: (N, 64, 32, 32)
         ↓                          ↓
      大尺寸特征图              小尺寸特征图
      (空间信息压缩)           (提取高级特征)
```

输出尺寸公式：

```
H_out = ⌊(H_in + 2×padding - kernel_size) / stride⌋ + 1
```

转置卷积（ConvTranspose2d）：上采样

```
# 生成器中使用
nn.ConvTranspose2d(in_channels=100, out_channels=512, kernel_size=4, stride=1, padding=0)

输入: (N, 100, 1, 1)   →   输出: (N, 512, 4, 4)
         ↓                          ↓
      小尺寸特征图              大尺寸特征图
      ( latent 空间)          (开始生成图像)
```

输出尺寸公式：

```
H_out = (H_in - 1) × stride - 2×padding + kernel_size + output_padding
```

### 转置卷积的数学实现（逐步推导）

方法一：通过普通卷积的逆运算理解

转置卷积的本质是：找到一个操作，使得它的正向传播等价于普通卷积的反向传播。

1. 普通卷积的前向和反向

假设有一个普通卷积层：

```py
conv = nn.Conv2d(1, 1, kernel_size=3, stride=2, padding=1)
```

前向传播：

```py
y = conv(x)  # x 是大尺寸，y 是小尺寸
```

反向传播（计算梯度）：

```py
∂L/∂x = conv_transpose(∂L/∂y)  # ∂L/∂y 是小尺寸，∂L/∂x 是大尺寸
```

注意到：在反向传播时，梯度的形状从 y 变回了 x 的形状，这个过程就是上采样！
关键洞察：转置卷积的前向传播 = 普通卷积的反向传播

方法二：通过矩阵乘法理解（更直观）

1. 将卷积表示为矩阵乘法

任何卷积操作都可以表示为稀疏矩阵乘法：

```
y = C · x
```

其中：

- x 是输入展平后的向量
- y 是输出展平后的向量
- C 是由卷积核构成的稀疏矩阵（Toeplitz 矩阵）

具体例子：
假设：

- 输入：x 形状为 (1, 4, 4)（1 通道，4×4）
- 卷积核：k 形状为 (1, 1, 3, 3)（3×3）
- 步长：stride = 2
- 填充：padding = 1

普通卷积的输出尺寸为：

```
H_out = ⌊(4 + 2×1 - 3) / 2⌋ + 1 = ⌊3/2⌋ + 1 = 1 + 1 = 2
```

输出：(1, 2, 2)
展开为矩阵形式：

输入展平：x_vec 长度为 4×4 = 16
 输出展平：y_vec 长度为 2×2 = 4

卷积矩阵 C 的形状为 (4, 16)：

```
y_vec = C · x_vec
  ↓       ↓     ↓
 (4,)   (4,16) (16,)
```

矩阵 C 的结构（简化示意）：

```
     x₁  x₂  x₃  x₄  x₅  ...  x₁₆
    ┌──────────────────────────┐
y₁  │ k₁  k₂  k₃   0  k₄  ...   0 │
y₂  │  0  k₁  k₂  k₃   0  ...   0 │
y₃  │  0   0   0   0  k₁  ...   0 │
y₄  │  0   0   0   0   0  ...  k₉ │
    └──────────────────────────┘
```

每个输出元素是输入局部区域与卷积核的点积。

2. 转置卷积的矩阵表示

转置卷积使用转置矩阵 C^T：

```
x_reconstructed = C^T · y
       ↓            ↓    ↓
     (16,)        (16,4) (4,)
```

关键特性：

- C 的形状：(4, 16) —— 下采样
- C^T 的形状：(16, 4) —— 上采样

这就是为什么叫"转置"卷积！

方法三：通过插零和卷积实现（最实用的理解方式）

这是 PyTorch 实际实现的思路，也是最容易可视化的方法。

转置卷积的实现步骤

步骤 1：在输入中插入零（零填充）

对于 stride = s 的转置卷积：

- 在输入特征图的每两个元素之间插入 s-1 个零
- 这称为"dilation of input"或"zero insertion"

步骤 2：旋转卷积核 180°
将卷积核上下翻转、左右翻转（等价于旋转 180°）。
步骤 3：执行普通卷积
用旋转后的卷积核对插零后的输入进行普通卷积。

### 具体数值示例（完整计算过程）

让我们用一个简化的例子来演示整个过程。

示例设置

```py
# 转置卷积层
conv_t = nn.ConvTranspose2d(
    in_channels=1,
    out_channels=1,
    kernel_size=3,
    stride=2,
    padding=1,
    bias=False
)

# 手动设置卷积核权重（方便追踪计算）
conv_t.weight.data = torch.tensor([[[[1.0, 2.0, 3.0],
                                      [4.0, 5.0, 6.0],
                                      [7.0, 8.0, 9.0]]]])

# 输入：2×2 的特征图
input_tensor = torch.tensor([[[[1.0, 2.0],
                               [3.0, 4.0]]]])
# 形状: (1, 1, 2, 2)
```

计算输出尺寸

```py
H_out = (H_in - 1) × stride - 2×padding + kernel_size + output_padding
      = (2 - 1) × 2 - 2×1 + 3 + 0
      = 1 × 2 - 2 + 3
      = 3

W_out = 同理 = 3
```

输出形状：(1, 1, 3, 3)

逐步计算过程

步骤 1：在输入中插零

原始输入 (2, 2)：

```
[1.0, 2.0]
[3.0, 4.0]
```

因为 stride = 2，在每两个元素之间插入 2-1 = 1 个零：
插零后 (5, 5)：

```
[1.0, 0.0, 2.0, 0.0, 0.0]
[0.0, 0.0, 0.0, 0.0, 0.0]
[3.0, 0.0, 4.0, 0.0, 0.0]
[0.0, 0.0, 0.0, 0.0, 0.0]
[0.0, 0.0, 0.0, 0.0, 0.0]
```

注意：实际上还会考虑 padding，这里简化展示核心思想。

步骤 2：旋转卷积核 180°

原始卷积核：

```
[1.0, 2.0, 3.0]
[4.0, 5.0, 6.0]
[7.0, 8.0, 9.0]
```

旋转 180° 后：

```
[9.0, 8.0, 7.0]
[6.0, 5.0, 4.0]
[3.0, 2.0, 1.0]
```

步骤 3：执行普通卷积

用旋转后的卷积核在插零后的输入上滑动，计算每个位置的点积。
以输出位置 (0, 0) 为例：

```
输出[0,0] = 1.0×9.0 + 0.0×8.0 + 2.0×7.0 +
           0.0×6.0 + 0.0×5.0 + 0.0×4.0 +
           3.0×3.0 + 0.0×2.0 + 4.0×1.0
         = 9.0 + 0 + 14.0 + 0 + 0 + 0 + 9.0 + 0 + 4.0
         = 36.0
```

继续计算所有位置，最终得到 (3, 3) 的输出。

# 归一化

## BatchNormd2d()

通常用于卷积神经网络的卷积层之后，以对数据进行归一化处理，从而提高网络的稳定性和性能。

### 基本用法

参数说明

```py
nn.BatchNorm2d(num_features, eps=1e-05, momentum=0.1, affine=True, track_running_stats=True)
```

示例代码

```py
nn.BatchNorm2d(ndf * 2)
```

根据上下文，ndf = 64，所以：

- num_features = 128 —— 输入特征图的通道数（即卷积层输出的通道数）
- 其他参数使用默认值

在判别器中的位置

```py
nn.Conv2d(ndf, ndf * 2, 4, 2, 1, bias=False),  # 输出 128 通道
nn.BatchNorm2d(ndf * 2),                         # ← 对 128 个通道归一化
nn.LeakyReLU(0.2, inplace=True),                 # 激活函数
```

### 数学原理（详细推导过程）

1. BatchNorm 的核心思想
    BatchNorm（批归一化）的目的是将每一层的输入分布标准化为均值为 0、方差为 1 的标准正态分布，然后通过可学习参数进行缩放和平移。
    这样做的好处：
    ✅ 加速训练收敛（允许使用更大的学习率）
    ✅ 减少对初始化的敏感性
    ✅ 有一定的正则化效果（减少过拟合）
    ✅ 缓解梯度消失/爆炸问题
2. 训练阶段的数学公式

假设输入是一个批次的数据，形状为 (N, C, H, W)：

- N = batch size（批次大小）
- C = channels（通道数）
- H = height（高度）
- W = width（宽度）

在你的代码中，第 155 行的 BatchNorm 接收的输入形状为：

```py
# 假设 batch_size = 128
input_shape = (128, 128, 32, 32)
#              N     C    H   W
```

步骤 1：计算每个通道的均值和方差
重要：BatchNorm 是按通道进行归一化的，每个通道有独立的统计量。
对于第 c 个通道（c = 0, 1, ..., C-1）：
均值计算：

```
        1
μ_c = ────────  Σ  Σ  Σ  x[n, c, h, w]
      N×H×W   n  h  w
```

展开理解：

- 遍历整个批次中所有样本的第 c 个通道
- 遍历该通道的所有空间位置 (h, w)
- 求和后除以总元素数 N × H × W

方差计算：

```
         1
σ²_c = ────────  Σ  Σ  Σ  (x[n, c, h, w] - μ_c)²
       N×H×W     n  h  w
```

具体数值例子
假设简化情况：

- N = 2（批次大小为 2）
- C = 3（3 个通道）
- H = 2, W = 2（2×2 的空间尺寸）

输入张量（仅看第 0 个通道）：

```py
# 样本 0 的第 0 通道
channel_0_sample_0 = [[1.0, 2.0],
                      [3.0, 4.0]]

# 样本 1 的第 0 通道
channel_0_sample_1 = [[5.0, 6.0],
                      [7.0, 8.0]]
```

计算第 0 通道的均值：

```
μ_0 = (1.0 + 2.0 + 3.0 + 4.0 + 5.0 + 6.0 + 7.0 + 8.0) / (2 × 2 × 2)
    = 36.0 / 8
    = 4.5
```

计算第 0 通道的方差：

```
σ²_0 = [(1.0-4.5)² + (2.0-4.5)² + (3.0-4.5)² + (4.0-4.5)² +
        (5.0-4.5)² + (6.0-4.5)² + (7.0-4.5)² + (8.0-4.5)²] / 8

     = [12.25 + 6.25 + 2.25 + 0.25 + 0.25 + 2.25 + 6.25 + 12.25] / 8
     = 42.0 / 8
     = 5.25
```

步骤 2：归一化（标准化）
对每个元素进行标准化：

```
               x[n, c, h, w] - μ_c
x̂[n, c, h, w] = ─────────────────
                  √(σ²_c + ε)
```

其中 ε（epsilon）是一个很小的常数（默认 1e-5），用于防止除零错误。
继续上面的例子：

```
√(σ²_0 + ε) = √(5.25 + 0.00001) ≈ 2.2913

x̂[0, 0, 0, 0] = (1.0 - 4.5) / 2.2913 ≈ -1.5275
x̂[0, 0, 0, 1] = (2.0 - 4.5) / 2.2913 ≈ -1.0911
x̂[0, 0, 1, 0] = (3.0 - 4.5) / 2.2913 ≈ -0.6547
x̂[0, 0, 1, 1] = (4.0 - 4.5) / 2.2913 ≈ -0.2182
...
```

归一化后的特点：

- 均值 ≈ 0
- 方差 ≈ 1

步骤 3：缩放和平移（仿射变换）

归一化后，通过可学习参数进行线性变换：

```
y[n, c, h, w] = γ_c · x̂[n, c, h, w] + β_c
```

其中：

- γ_c（gamma）：缩放参数，初始值为 1_
- _β_c（beta）：平移参数，初始值为 0
- 这两个参数是每个通道独立的可学习参数

为什么需要这一步？

- 如果直接标准化，可能会限制网络的表达能力
- 例如，某些层可能需要保持较大的方差
- 通过 γ 和 β，网络可以学习到最优的分布形式
- 如果 γ_c = σ_c 且 β_c = μ_c，就能恢复原始分布

继续例子： 假设学习到的参数：

```
γ_0 = 1.5  （第 0 通道的缩放因子）
β_0 = 0.3  （第 0 通道的偏置）

y[0, 0, 0, 0] = 1.5 × (-1.5275) + 0.3 ≈ -1.9913
y[0, 0, 0, 1] = 1.5 × (-1.0911) + 0.3 ≈ -1.3367
...
```

3. 推理（测试）阶段的处理

在训练时，我们使用当前批次的统计量（均值和方差）。但在推理时：

- 可能只有一个样本（batch_size = 1）
- 批次统计量不稳定

解决方案：使用移动平均统计量
在训练过程中，维护全局的运行均值和运行方差：

```py
running_mean = (1 - momentum) × running_mean + momentum × μ_batch
running_var  = (1 - momentum) × running_var  + momentum × σ²_batch
```

默认 momentum = 0.1，这意味着：

- 新批次的统计量占 10% 权重
- 历史统计量占 90% 权重

推理时的归一化：

```
            x[c, h, w] - running_mean_c
x̂[c, h, w] = ───────────────────────────
               √(running_var_c + ε)

y[c, h, w] = γ_c · x̂[c, h, w] + β_c
```

可视化单个通道的处理流程

```
输入特征图（通道 c）
┌─────────────────────┐
│  3.2  -1.5   0.8    │
│ -0.3   2.1  -2.4    │
│  1.7   0.5   1.2    │
└─────────────────────┘
         ↓
  计算统计量
  μ_c = 0.589
  σ²_c = 3.124
         ↓
  归一化 (减均值，除标准差)
┌─────────────────────┐
│  1.48  -1.18  0.12  │
│ -0.50   0.86 -1.69  │
│  0.63  -0.05  0.35  │
└─────────────────────┘
  均值≈0, 方差≈1
         ↓
  缩放和平移 (γ_c=1.2, β_c=0.1)
┌─────────────────────┐
│  1.88  -1.32  0.24  │
│ -0.50   1.13 -1.93  │
│  0.86   0.04  0.52  │
└─────────────────────┘
         ↓
  输出到下一层
```

## LayerNorm()

## InstanceNorm()

## GroupNorm()

## SwitchableNorm()

## RMSNorm()

## WightNorm()

## SpectralNorm()

## BatchRenorm()

## CosineNorm()

## FeatureNorm()

## DyT()

# 激活函数

## Sigmoid()

**Sigmoid函数** 是一种常见的激活函数，广泛应用于神经网络中

### 基本用法

参数说明

```py
nn.Sigmoid()
```

特点：
❌ 没有参数 —— Sigmoid 是固定的数学函数，无需学习
✅ 逐元素操作 —— 对输入张量的每个元素独立应用
✅ 输出范围 —— 将任意实数映射到 (0, 1) 区间
你选中的代码：

```py
nn.Conv2d(ndf * 8, 1, 4, 1, 0, bias=False),  # 输出 1 通道
nn.Sigmoid()                                    # ← 将输出压缩到 (0, 1)
```

### 数学实现原理（详细推导）

1. Sigmoid 函数的数学定义

Sigmoid 函数（也称为 Logistic 函数）的公式为：

```
            1
σ(x) = ──────────────
         1 + e^(-x)
```

其中：

- e 是自然常数（≈ 2.71828）
- x 是输入值（以是任意实数）
- σ(x) 是输出值（范围在 0 到 1 之间）

2. 函数特性分析

（1）输出范围

```
当 x → +∞ 时：e^(-x) → 0     →  σ(x) → 1/(1+0) = 1
当 x → -∞ 时：e^(-x) → +∞   →  σ(x) → 1/(1+∞) = 0
当 x = 0 时：   e^(-0) = 1   →  σ(0) = 1/(1+1) = 0.5
```

结论：Sigmoid 将整个实数轴压缩到 (0, 1) 开区间。

（2）对称性

```
σ(-x) = 1 - σ(x)
```

证明：

```
          1            e^x           e^x         1 + e^x - 1
σ(-x) = ────────── = ─────────── = ────────── = ──────────────
        1 + e^x        e^x + 1       1 + e^x        1 + e^x

      e^x + 1       1
    = ─────── - ───────── = 1 - σ(x)
      1 + e^x     1 + e^x
```

这意味着函数关于点 (0, 0.5) 中心对称。

（3）导数（梯度）

Sigmoid 的导数有一个非常优美的形式：

```
dσ(x)
───── = σ(x) · [1 - σ(x)]
 dx
```

推导过程：
使用商法则求导：

```
         d  ⎛    1     ⎞
σ'(x) = ─── ⎜──────────⎟ 
         dx ⎝1 + e^(-x)⎠

      -(1 + e^(-x))'
    = ────────────────
        (1 + e^(-x))²

      -(-e^(-x))         e^(-x)
    = ──────────── = ────────────
      (1 + e^(-x))²    (1 + e^(-x))²
```

巧妙变形：

```
      e^(-x)           1      
    = ────────── · ──────────
      1 + e^(-x)    1 + e^(-x)

      ⎛   1     ⎞  ⎛   e^(-x)     ⎞
    = ⎜──────────⎟·⎜──────────────⎟
      ⎝1 + e^(-x)⎠  ⎝1 + e^(-x)   ⎠

      ⎛   1     ⎞   ⎛1 + e^(-x) - 1  ⎞
    = ⎜──────────⎟·⎜─────────────────⎟
      ⎝1 + e^(-x)⎠  ⎝  1 + e^(-x)    ⎠

    = σ(x) · [1 - σ(x)]
```

这个性质的重要性：

- 计算梯度时不需要重新计算指数，只需利用前向传播的结果
- 大大提高了计算效率

3. 具体数值计算示例

让我们手动计算几个典型值：

示例 1：x = 0

```
σ(0) = 1 / (1 + e^0) = 1 / (1 + 1) = 1/2 = 0.5
```

示例 2：x = 2

```
e^(-2) ≈ 0.1353
σ(2) = 1 / (1 + 0.1353) ≈ 1 / 1.1353 ≈ 0.8808
```

## ReLU()

### 基本用法

参数说明

```py
nn.ReLU(inplace=False)
```

示例代码

```py
nn.ReLU(True)
```

这里：

- inplace = True —— 原地操作，直接修改输入张量，节省内存
- 等价于 nn.ReLU(inplace=True)

在生成器中的位置

```py
nn.ConvTranspose2d(nz, ngf * 8, 4, 1, 0, bias=False),  # 转置卷积
nn.BatchNorm2d(ngf * 8),                                # 批归一化
nn.ReLU(True),                                          # ← ReLU 激活
```

### 数学原理（详细推导）

1. ReLU 的数学定义

ReLU（Rectified Linear Unit，线性整流单元）是一个分段线性函数：

```
            ⎧  x    if x > 0
ReLU(x) =   ⎨
            ⎩  0    if x ≤ 0
```

也可以写成简洁形式：

```py
ReLU(x) = max(0, x)
```

2. 函数特性分析

（1）输出范围

```
当 x > 0 时：ReLU(x) = x     →  输出为正数
当 x ≤ 0 时：ReLU(x) = 0    →  输出为零
```

（2）非线性特性

虽然 ReLU 在每个分段上是线性的，但整体是非线性函数，这使得神经网络能够学习复杂的模式。

（3）导数（梯度）

ReLU 的导数为：

```
              ⎧  1    if x > 0
ReLU'(x) =    ⎨
              ⎩  0    if x ≤ 0
```

在 x = 0 处的不可导性：

- 严格来说，ReLU 在 x = 0 处不可导（左导数为 0，右导数为 1）
- 在实际实现中，通常定义 ReLU'(0) = 0 或 ReLU'(0) = 1
- PyTorch 使用 次梯度（subgradient），通常取 0 或 1 都可以

3. 具体数值计算示例

假设 BatchNorm 输出的特征图为：

```py
input_tensor = torch.tensor([
    [[ 2.5, -1.3,  0.8],
     [-0.5,  3.2, -2.1],
     [ 1.7, -0.9,  0.0]]
])
```

经过 nn.ReLU(inplace=True) 处理：
逐元素计算过程：

```
位置 (0,0): x =  2.5 > 0  →  output =  2.5    (保持不变)
位置 (0,1): x = -1.3 ≤ 0  →  output =  0.0    (截断为0)
位置 (0,2): x =  0.8 > 0  →  output =  0.8    (保持不变)

位置 (1,0): x = -0.5 ≤ 0  →  output =  0.0
位置 (1,1): x =  3.2 > 0  →  output =  3.2
位置 (1,2): x = -2.1 ≤ 0  →  output =  0.0

位置 (2,0): x =  1.7 > 0  →  output =  1.7
位置 (2,1): x = -0.9 ≤ 0  →  output =  0.0
位置 (2,2): x =  0.0 ≤ 0  →  output =  0.0
```

输出结果：

```
output_tensor = torch.tensor([
    [[ 2.5,  0.0,  0.8],
     [ 0.0,  3.2,  0.0],
     [ 1.7,  0.0,  0.0]]
])
```

### 反向传播时的梯度流动

1. 链式法则应用

假设损失函数为 L，ReLU 的输入为 x，输出为 y = ReLU(x)。
根据链式法则：

```
∂L/∂x = ∂L/∂y · ∂y/∂x
```

其中 ∂y/∂x 就是 ReLU 的导数：

```
              ⎧  ∂L/∂y · 1 = ∂L/∂y    if x > 0
∂L/∂x =       ⎨
              ⎩  ∂L/∂y · 0 = 0        if x ≤ 0
```

2. 具体数值例子

假设后一层传回的梯度为 dL/dy = [0.5, -0.3, 0.8]：

```
前向传播时的输入: x = [2.5, -1.3, 0.8]
前向传播时的输出: y = [2.5,  0.0, 0.8]

反向传播：
位置 0: x =  2.5 > 0  →  dL/dx = 0.5 × 1 =  0.5   (梯度完整传递)
位置 1: x = -1.3 ≤ 0  →  dL/dx = -0.3 × 0 =  0.0   (梯度被阻断)
位置 2: x =  0.8 > 0  →  dL/dx = 0.8 × 1 =  0.8   (梯度完整传递)
```



## LeakyReLU()

nn.LeakyReLU 是 PyTorch 中的 Leaky Rectified Linear Unit（ReLU）激活函数的实现。Leaky ReLU 是一种修正线性单元，它在非负数部分保持线性，而在负数部分引入一个小的斜率（通常是一个小的正数），以防止梯度消失问题。

### 基本用法

参数说明

```py
nn.LeakyReLU(negative_slope=0.01, inplace=False)
```

示例代码：

```py
nn.LeakyReLU(0.2, inplace=True)
```

这里：

- negative_slope = 0.2 —— 负半轴的斜率（默认值是 0.01）
- inplace = True —— 原地操作，直接修改输入张量，节省内存

### 数学实现原理（详细过程）

1. LeakyReLU 的数学定义

LeakyReLU（带泄露的线性整流单元）是一个分段函数：

```
            ⎧  x          if x > 0
f(x) =      ⎨
            ⎩  α·x        if x ≤ 0
```

其中：

- x 是输入值
- α (alpha) 是 negative_slope 参数，在你的代码中 α = 0.2

2. 与 ReLU 的对比

为了理解 LeakyReLU 的优势，我们先看标准 ReLU：
标准 ReLU：

```
            ⎧  x          if x > 0
ReLU(x) =   ⎨
            ⎩  0          if x ≤ 0
```

问题：当 x ≤ 0 时，输出恒为 0，梯度也为 0，导致"神经元死亡"（Dead Neuron Problem）。
LeakyReLU 的改进：

优势：即使输入为负数，也有非零的输出和梯度，避免神经元完全失效。
3. 数据流向详解

假设卷积层输出的特征图为：

```py
# 输入张量形状：(batch_size, 64, 32, 32)
input_tensor = torch.tensor([
    [[ 2.5, -1.3,  0.8],
     [-0.5,  3.2, -2.1],
     [ 1.7, -0.9,  0.0]]
])
```

经过 nn.LeakyReLU(0.2, inplace=True) 处理：
逐元素计算过程：

```py
位置 (0,0): x =  2.5 > 0  →  output =  2.5        (保持不变)
位置 (0,1): x = -1.3 ≤ 0  →  output =  0.2×(-1.3) = -0.26  (缩小到20%)
位置 (0,2): x =  0.8 > 0  →  output =  0.8        (保持不变)

位置 (1,0): x = -0.5 ≤ 0  →  output =  0.2×(-0.5) = -0.1
位置 (1,1): x =  3.2 > 0  →  output =  3.2
位置 (1,2): x = -2.1 ≤ 0  →  output =  0.2×(-2.1) = -0.42

位置 (2,0): x =  1.7 > 0  →  output =  1.7
位置 (2,1): x = -0.9 ≤ 0  →  output =  0.2×(-0.9) = -0.18
位置 (2,2): x =  0.0 ≤ 0  →  output =  0.2×0.0    =  0.0
```

输出结果：

```py
output_tensor = torch.tensor([
    [[ 2.5,  -0.26,  0.8 ],
     [-0.1,   3.2,  -0.42],
     [ 1.7,  -0.18,  0.0 ]]
])
```

5. 反向传播时的梯度计算

在训练过程中，需要计算损失函数对输入的梯度。LeakyReLU 的导数为：

```py
              ⎧  1          if x > 0
f'(x) =       ⎨
              ⎩  α          if x ≤ 0
```

具体例子：
假设后一层传回的梯度为 dL/dy = [0.5, -0.3, 0.8]，根据链式法则：

```
位置 0: x =  2.5 > 0  →  dL/dx = dL/dy × 1   =  0.5 × 1   =  0.5
位置 1: x = -1.3 ≤ 0  →  dL/dx = dL/dy × 0.2 = -0.3 × 0.2 = -0.06
位置 2: x =  0.8 > 0  →  dL/dx = dL/dy × 1   =  0.8 × 1   =  0.8
```

关键优势：即使 x < 0，梯度也不会消失（保持为 α），这使得网络能够持续学习。

# 损失函数

## BCELoss()

### 基本用法

参数说明

```py
nn.BCELoss(weight=None, size_average=None, reduce=None, reduction='mean')
```

示例代码

```py
criterion = nn.BCELoss()
```

使用默认参数：

- reduction = 'mean' —— 对批次中所有元素的损失求平均
- weight = None —— 不使用权重加权

### 数学原理（详细推导）

1. BCE 的数学定义

BCE（Binary Cross Entropy，二元交叉熵）用于二分类问题的损失函数。
公式：

```
               1   N
BCE(ŷ, y) = - ────  Σ  [yᵢ·log(ŷᵢ) + (1-yᵢ)·log(1-ŷᵢ)]
               N  i=1

```

其中：

- N = 样本数量（批次大小）
- yᵢ = 真实标签（0 或 1）
- ŷᵢ = 预测概率（范围 (0, 1)，通常由 Sigmoid 输出）
- log = 自然对数（以 e 为底）

2. 单个样本的损失

对于单个样本，损失为：

```
L(ŷ, y) = -[y·log(ŷ) + (1-y)·log(1-ŷ)]
```

这是一个分段函数：
情况 1：真实标签 y = 1（真实图像）

```
L(ŷ, 1) = -[1·log(ŷ) + (1-1)·log(1-ŷ)]
        = -log(ŷ)
```

特性：

- 当 ŷ → 1 时：L → -log(1) = 0 ✅（预测正确，损失为 0）
- 当 ŷ → 0 时：L → -log(0) → +∞ ❌（预测错误，损失无穷大）

情况 2：真实标签 y = 0（假图像）

```
L(ŷ, 0) = -[0·log(ŷ) + (1-0)·log(1-ŷ)]
        = -log(1-ŷ)
```

特性：

- 当 ŷ → 0 时：L → -log(1) = 0 ✅（预测正确，损失为 0）
- 当 ŷ → 1 时：L → -log(0) → +∞ ❌（预测错误，损失无穷大）

3. 具体数值计算示例

假设判别器对一个批次的 4 个样本的输出为：

```py
# 判别器输出（经过 Sigmoid 后的概率）
predictions = torch.tensor([0.9, 0.8, 0.3, 0.2])

# 真实标签
labels = torch.tensor([1.0, 1.0, 0.0, 0.0])
#          ↑ 真实图像  ↑ 假图像
```

逐个样本计算损失：

```
样本 0: y=1, ŷ=0.9
  L₀ = -log(0.9) ≈ 0.1054

样本 1: y=1, ŷ=0.8
  L₁ = -log(0.8) ≈ 0.2231

样本 2: y=0, ŷ=0.3
  L₂ = -log(1-0.3) = -log(0.7) ≈ 0.3567

样本 3: y=0, ŷ=0.2
  L₃ = -log(1-0.2) = -log(0.8) ≈ 0.2231
```

批次平均损失：

```
BCE = (L₀ + L₁ + L₂ + L₃) / 4
    = (0.1054 + 0.2231 + 0.3567 + 0.2231) / 4
    = 0.9083 / 4
    = 0.2271
```

### 梯度推导（核心数学）

1. 为什么 BCE 配合 Sigmoid 效果好？

让我们推导 BCE 对 Sigmoid 输入的梯度。

设定：

- z = Sigmoid 前的 logits（卷积层的原始输出）
- ŷ = σ(z) = 1/(1+e^(-z)) = Sigmoid 输出
- y = 真实标签（0 或 1）

BCE 损失：

```
L = -[y·log(ŷ) + (1-y)·log(1-ŷ)]
```

求梯度 ∂L/∂z：
根据链式法则：

```
∂L/∂z = ∂L/∂ŷ · ∂ŷ/∂z
```

步骤 1：计算 ∂L/∂ŷ

```
∂L/∂ŷ = -[y/ŷ - (1-y)/(1-ŷ)]
      = -y/ŷ + (1-y)/(1-ŷ)
```

步骤 2：计算 ∂ŷ/∂z（Sigmoid 的导数）

```
∂ŷ/∂z = σ'(z) = ŷ(1-ŷ)
```

步骤 3：相乘

```
∂L/∂z = [-y/ŷ + (1-y)/(1-ŷ)] · ŷ(1-ŷ)
      = -y(1-ŷ) + (1-y)ŷ
      = -y + y·ŷ + ŷ - y·ŷ
      = ŷ - y
```

结论：

```
∂L/∂z = ŷ - y
```

这是一个极其简洁的梯度形式！

# 优化器

