# 介绍

**orchvision** 是 PyTorch 生态系统中专门为 **计算机视觉任务** 提供支持的扩展库。它极大地简化了图像数据的处理、模型训练和推理流程。

# 常见API

## tv_tensors

`tv_tensors` 是 **Torchvision v2+** 引入的一类特殊张量类型（`torch.Tensor` 的子类），主要用于 **图像、视频、边界框、分割掩码等数据的类型标记与元数据管理**，方便 `torchvision.transforms.v2` 自动选择合适的处理逻辑。

**`tv_tensors`**（TorchVision Tensors）是带有额外语义信息的张量。

### 1. 基本概念

常见类型：

- `Image`：图像张量（支持元数据如颜色空间）
- `Video`：视频张量
- `BoundingBoxes`：边界框张量（带格式和画布大小信息）
- `Mask`：分割掩码

作用：

- 让 `transforms.v2` 根据类型自动调用正确的变换（例如 `RandomHorizontalFlip` 会自动翻转图像和对应的边界框）。

### 2. 创建 `tv_tensors`

```py
import torch
from torchvision.tv_tensors import Image, BoundingBoxes

# 创建 Image 类型
img_tensor = torch.randint(0, 256, (3, 224, 224), dtype=torch.uint8)
img = Image(img_tensor)  # 包装成 Image 类型

# 创建 BoundingBoxes 类型
boxes_tensor = torch.tensor([[10, 20, 100, 200], [50, 60, 150, 180]], dtype=torch.float32)
boxes = BoundingBoxes(
    boxes_tensor,
    format="XYXY",          # 边界框格式
    canvas_size=(224, 224)  # 图像尺寸
)
```

### 3. 在 `transforms.v2` 中使用

```py
from torchvision.transforms import v2

transform = v2.Compose([
    v2.RandomHorizontalFlip(p=1.0),  # 100% 概率翻转
    v2.Resize((112, 112))
])

img_out, boxes_out = transform(img, boxes)

print(type(img_out))   # <class 'torchvision.tv_tensors.Image'>
print(type(boxes_out)) # <class 'torchvision.tv_tensors.BoundingBoxes'>
```

**好处**：`RandomHorizontalFlip` 会自动翻转图像和边界框，无需手动同步。

### 4. 从普通 Tensor 转换

```py
from torchvision.tv_tensors import wrap

# 将普通 Tensor 转为 Image
img2 = wrap(img_tensor, like=img)  # 继承 img 的元数据
```

# transforms

## RandomResizedCrop(224)

随机裁剪并缩放

```
原始图片 (任意大小)
    ↓
【步骤 1】随机裁剪一个矩形区域（保持原图内容的一部分）
    ↓  
【步骤 2】将这个区域缩放到 224×224 像素
    ↓
输出：224×224 的图片
```

为什么要这样做？

​	数据增强：每次训练时裁剪不同的区域，相当于创造了新样本
​	防止过拟合：网络不会记住图片的特定位置特征
​	统一尺寸：确保所有输入图片大小一致

```
# 示例：不同裁剪效果
原图：一只猫在左上角
第一次裁剪：→ 只保留猫的头部 → 缩放为 224×224
第二次裁剪：→ 只保留猫的身体 → 缩放为 224×224
第三次裁剪：→ 保留整只猫但背景不同 → 缩放为 224×224
```

##  RandomHorizontalFlip()

 随机水平翻转

作用： 以 50% 的概率水平翻转图片

```
# 示例：
原图：[猫头朝左]  --50%概率-->  [猫头朝右]
                            或
                      [猫头朝左] (不变)
```

为什么只水平不垂直？

​	现实世界中，物体左右对称很常见（猫从左看 vs 从右看）
​	但上下颠倒很少见（倒立的猫不符合常理）

##  ToTensor()

转换为张量

作用： 将 PIL 图片或 NumPy 数组转换成 PyTorch 张量

```
# 转换过程：
PIL 图片 (H×W×C, 范围 [0, 255])
    ↓
Tensor (C×H×W, 范围 [0.0, 1.0])
```

两个变化：

​	维度顺序改变：(高，宽，通道) → (通道，高，宽)
​	数值归一化：[0, 255] → [0.0, 1.0] (除以 255)

##  Normalize([0.485, 0.456, 0.406], [0.229, 0.224, 0.225]) 

 标准化

作用： 对每个颜色通道进行标准化处理

```
output[channel] = (input[channel] - mean[channel]) / std[channel]
```

具体计算过程：

``` 
# 假设某像素的 RGB 值为 [0.5, 0.3, 0.7] (已经在 0-1 范围)

# R 通道：(0.5 - 0.485) / 0.229 = 0.0655
# G 通道：(0.3 - 0.456) / 0.224 = -0.6964
# B 通道：(0.7 - 0.406) / 0.225 = 1.3067

# 结果：标准化后的值分布在 0 附近，标准差约为 1
```

##  Resize(256)

```
# 示例：
原图 300×200  → 短边缩放到 256 → 384×256
原图 200×400  → 短边缩放到 256  → 256×512
```

## CenterCrop(224)

```
# 从图片中心裁剪出 224×224 的区域
384×256 的图片 → 从正中间裁剪 → 224×224
```

## v2

是 **PyTorch 0.15+** 引入的新一代数据增强 API，相比旧版 `torchvision.transforms`，它支持 **张量原生操作**、**多类型输入**（PIL、Tensor、视频、分割掩码、边界框等）以及 **批量处理**，并且在 GPU 上也能运行。

### 1. 基本用法

```py
import torch
from torchvision import tv_tensors
from torchvision.transforms import v2
from PIL import Image

# 读取图片（PIL 格式）
img = Image.open("example.jpg").convert("RGB")

# 定义 transforms 管道
transform = v2.Compose([
    v2.Resize((256, 256)),          # 调整大小
    v2.RandomHorizontalFlip(p=0.5), # 随机水平翻转
    v2.RandomRotation(15),          # 随机旋转 ±15°
    v2.ToImage(),                   # 转为张量 (C,H,W)，值范围 [0,255]
    v2.ToDtype(torch.float32, scale=True),  # 转 float32 并归一化到 [0,1]
    v2.Normalize(mean=[0.485, 0.456, 0.406], 
                 std=[0.229, 0.224, 0.225]) # 标准化
])

# 应用变换
img_tensor = transform(img)  # 输出: torch.Size([3, 256, 256])
print(img_tensor.shape, img_tensor.dtype)
```

### 2. 同时处理图像和标签（如分割掩码）

`v2` 支持一次性对多种数据类型做同步变换（保证图像和标签对齐）。

```py
# 模拟一张图像和对应的分割掩码
image = Image.open("example.jpg").convert("RGB")
mask = Image.open("mask.png").convert("L")  # 单通道

# 转为 tv_tensors，方便 v2 识别类型
image = tv_tensors.Image(image)
mask = tv_tensors.Mask(mask)

transform = v2.Compose([
    v2.Resize((256, 256)),
    v2.RandomHorizontalFlip(p=0.5),
])

# 同时变换
image_t, mask_t = transform(image, mask)
print(type(image_t), type(mask_t))  # tv_tensors.Image, tv_tensors.Mask
```

### 3. 处理边界框（目标检测）

```py
from torchvision.tv_tensors import BoundingBoxes

# 模拟一张图像和对应的边界框
image = tv_tensors.Image(torch.randint(0, 255, (3, 300, 400), dtype=torch.uint8))
boxes = BoundingBoxes([[50, 60, 200, 220]], format="XYXY", canvas_size=(300, 400))

transform = v2.Compose([
    v2.Resize((256, 256)),
    v2.RandomHorizontalFlip(p=0.5),
])

image_t, boxes_t = transform(image, boxes)
print(boxes_t)
```

# datasets

## ImageFolder()

它通常用于加载图像数据集，尤其是文件夹结构化的数据集。每个子文件夹代表一个类别，图像文件保存在这些子文件夹中。`ImageFolder` 将这些文件夹作为类标签进行处理，并自动将每个图像与其标签配对。

**加载数据**：

使用 `ImageFolder` 加载数据集。你需要提供数据集的根目录路径，以及定义的转换操作。

```py
dataset = datasets.ImageFolder(root="path_to_your_dataset", transform=transform)
```

目录结构要求：

`ImageFolder` 要求数据集的目录结构如下：

```bash
root/
    class_1/
        img1.jpg
        img2.jpg
        ...
    class_2/
        img1.jpg
        img2.jpg
        ...
    ...
```

示例

```py
# 使用 ImageFolder 加载测试图像数据集
# ImageFolder 会自动读取 ./datas/test_images 目录下的所有子文件夹作为不同类别
dataset = datasets.ImageFolder('./data')

# 构建反向映射字典：将类别索引映射回类别名称（人名）
# dataset.class_to_idx 是 {'person1': 0, 'person2': 1, ...} 的格式
# 我们需要反转它为 {0: 'person1', 1: 'person2', ...} 方便后续查找
dataset.idx_to_class = {i: c for c, i in dataset.class_to_idx.items()}
```



# utils



# io

### read_image()

内部实现基于 PIL (Python Imaging Library)

```py
# torchvision 的默认图像后端是 PIL
_image_backend = 'PIL'

# read_image 的核心逻辑（简化版）
def read_image(path, mode=ImageReadMode.RGB):
    from PIL import Image
    
    # 1. 使用 PIL 打开图片文件
    img = Image.open(path)
    
    # 2. 根据 mode 参数转换图片模式
    if mode == ImageReadMode.RGB:
        img = img.convert('RGB')
    elif mode == ImageReadMode.GRAY:
        img = img.convert('L')
    
    # 3. 将 PIL Image 转换为 numpy 数组
    img_array = np.array(img)
    
    # 4. 将 numpy 数组转换为 torch.Tensor
    # 注意：维度从 (H, W, C) 转换为 (C, H, W)
    tensor = torch.from_numpy(img_array).permute(2, 0, 1)
    
    return tensor
```



# ops

torchvision.ops是PyTorch的一个辅助库，主要用于提供一些通用的图像操作和处理功能。它封装了常用的图像变换操作，如图像裁剪、旋转、颜色调整、灰度变换等，同时还支持随机发生的一些操作。此外，它还包括一些通用操作，如非极大值抑制、Box面积过滤、图像仿射变换等。这些功能可以用于图像分类、目标检测、图像生成等计算机视觉任务。 

## boxes

### masks_to_boxes

`torchvision.ops.masks_to_boxes` 是 **PyTorch Torchvision** 提供的一个实用函数，用于将二值分割掩码（mask）转换为对应的 **边界框坐标**。

```py
torchvision.ops.masks_to_boxes(masks: torch.Tensor) -> torch.Tensor
```

#### **参数**

- `masks`

  :

  - 类型：`torch.Tensor`
  - 形状：[N, H, W]
    - `N`：掩码数量（batch size）
    - `H`：图像高度
    - `W`：图像宽度
  - 每个掩码是一个二值（0/1）或布尔张量，表示目标区域。

#### **返回值**

**`boxes`**:

- 类型：`torch.Tensor`
- 形状：`[N, 4]`
- 每行格式为 `(x1, y1, x2, y2)`，表示左上角和右下角坐标。
- 坐标是 **浮点数**，且满足 `0 <= x1 <= x2` 和 `0 <= y1 <= y2`。

**基本用法示例**

```py
import torch
from torchvision.ops import masks_to_boxes

# 假设有两个 5x5 的二值掩码
masks = torch.tensor([
    [
        [0, 0, 0, 0, 0],
        [0, 1, 1, 0, 0],
        [0, 1, 1, 0, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0]
    ],
    [
        [0, 0, 0, 0, 0],
        [0, 0, 1, 1, 0],
        [0, 0, 1, 1, 0],
        [0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0]
    ]
], dtype=torch.uint8)

# 计算边界框
boxes = masks_to_boxes(masks)

print(boxes)

"""
tensor([[1., 1., 2., 2.],
        [2., 1., 3., 2.]])
"""
```

解释：

- 第一个掩码的目标区域在 `(x1=1, y1=1)` 到 `(x2=2, y2=2)`。
- 第二个掩码的目标区域在 `(x1=2, y1=1)` 到 `(x2=3, y2=2)`。
