# 介绍

Pillow 库提供了丰富的功能模块，如 *Image* 图像处理模块、*ImageFont* 添加文本模块、*ImageColor* 颜色处理模块、*ImageDraw* 绘图模块等。

安装

```bash
pip install pillow
```

# 常见API

# Image库

## open()

#### **Image.open() 的详细工作原理**

```py
image = Image.open(image_path).convert("RGB")
```

功能：从磁盘加载图片文件到内存

数据流向与内部处理

```py
磁盘上的图片文件 (PNG/JPG/etc.)
  ↓
【1. 文件读取】
读取文件头（File Header）
  ├─ PNG: 前 8 字节 = 89 50 4E 47 0D 0A 1A 0A
  ├─ JPEG: 前 2 字节 = FF D8
  └─ BMP: 前 2 字节 = 42 4D
  ↓
【2. 格式检测】
根据文件头识别图片格式
  ↓
【3. 选择解码器】
PNG → PNG Decoder
JPEG → JPEG Decoder (libjpeg)
BMP → BMP Decoder
  ↓
【4. 解析元数据】
读取图片信息（不解码像素数据）：
  ├─ width: 图片宽度（像素）
  ├─ height: 图片高度（像素）
  ├─ mode: 颜色模式（RGB, RGBA, L, CMYK等）
  ├─ format: 文件格式（PNG, JPEG等）
  └─ info: 额外信息（EXIF, DPI, 注释等）
  ↓
【5. 创建 Image 对象】
返回一个 "lazy" 对象（延迟加载）
  - 此时像素数据可能还未完全解码
  - 只有在需要时才真正解码
  ↓
输出：PIL.Image.Image 对象
```

关键特性：延迟加载（Lazy Loading）

```py
# 第 1 步：打开文件（很快，因为还没解码像素）
image = Image.open("page_001.png")
# 耗时：~0.001s

# 此时 image 对象只有元数据
print(image.width)   # ✅ 可以访问
print(image.height)  # ✅ 可以访问
print(image.mode)    # ✅ 可以访问

# 第 2 步：首次访问像素数据时，才真正解码
pixels = image.load()  # ← 这时才解码所有像素
# 耗时：~0.05s（取决于图片大小）

# 或者转换为数组时也会触发解码
import numpy as np
arr = np.array(image)  # ← 触发解码
```

为什么这样设计？
✅ 节省内存：如果只读取元数据，不需要加载整个图片
✅ 提高速度：快速判断图片属性，避免不必要的解码
⚠️ 注意：文件句柄保持打开状态，直到 image 对象被销毁或调用 image.close()

步骤 2：.convert("RGB")

功能：将图片转换为 RGB 颜色模式

为什么这样设计？
✅ 节省内存：如果只读取元数据，不需要加载整个图片
✅ 提高速度：快速判断图片属性，避免不必要的解码
⚠️ 注意：文件句柄保持打开状态，直到 image 对象被销毁或调用 image.close()

步骤 2：.convert("RGB")
功能：将图片转换为 RGB 颜色模式
为什么要转换？

```py
# 原始图片可能有不同的颜色模式：

# 情况 1：RGBA（带透明通道）
mode = "RGBA"  # 4 通道：Red, Green, Blue, Alpha
# 问题：Tesseract 不支持透明通道
# 解决：转换为 RGB，丢弃 Alpha 通道

# 情况 2：L（灰度）
mode = "L"  # 1 通道：Gray (0-255)
# 问题：虽然 Tesseract 支持灰度，但统一为 RGB 更稳定
# 解决：复制灰度值到 R,G,B 三个通道

# 情况 3：P（调色板模式）
mode = "P"  # 索引颜色，最多 256 色
# 问题：需要查表才能得到真实颜色
# 解决：转换为 RGB，展开为真实颜色

# 情况 4：CMYK（印刷色彩）
mode = "CMYK"  # 4 通道：Cyan, Magenta, Yellow, Key(Black)
# 问题：颜色空间不同，需要转换
# 解决：转换为 RGB

# 情况 5：1（二值图像）
mode = "1"  # 黑白，1 bit per pixel
# 问题：位深度太低
# 解决：转换为 RGB（0→黑色, 255→白色）
```

转换过程详解

```py
# 伪代码展示 convert("RGB") 的内部逻辑

def convert_to_rgb(image):
    """将任意模式的图片转换为 RGB"""
    
    if image.mode == "RGB":
        # 已经是 RGB，直接返回（或创建副本）
        return image.copy()
    
    elif image.mode == "RGBA":
        # RGBA → RGB：丢弃 Alpha 通道
        # 方法：在白色背景上合成
        background = Image.new("RGB", image.size, (255, 255, 255))
        background.paste(image, mask=image.split()[3])  # 使用 Alpha 作为掩码
        return background
        
        # 或者简单粗暴：直接丢弃 Alpha
        # r, g, b, a = image.split()
        # return Image.merge("RGB", (r, g, b))
    
    elif image.mode == "L":
        # 灰度 → RGB：复制到三个通道
        # gray = 128
        # → R=128, G=128, B=128
        width, height = image.size
        rgb_image = Image.new("RGB", (width, height))
        rgb_image.paste(image)  # 自动复制到所有通道
        return rgb_image
    
    elif image.mode == "P":
        # 调色板 → RGB：查表转换
        # palette[0] = (255, 0, 0)    # 索引 0 → 红色
        # palette[1] = (0, 255, 0)    # 索引 1 → 绿色
        # 
        # 像素值 [0, 1, 0, 1]
        # → [(255,0,0), (0,255,0), (255,0,0), (0,255,0)]
        return image.convert("RGB")  # 内置方法自动查表
    
    elif image.mode == "CMYK":
        # CMYK → RGB：颜色空间转换
        # C=0, M=0, Y=0, K=0 → R=255, G=255, B=255 (白色)
        # C=100%, M=100%, Y=100%, K=100% → R=0, G=0, B=0 (黑色)
        #
        # 公式（简化）：
        # R = 255 × (1-C) × (1-K)
        # G = 255 × (1-M) × (1-K)
        # B = 255 × (1-Y) × (1-K)
        return image.convert("RGB")
    
    elif image.mode == "1":
        # 二值 → RGB：0→(0,0,0), 1→(255,255,255)
        return image.convert("RGB")
```

实际例子

```py
# 假设原始图片是 PNG 格式，带透明背景
image_path = "page_001.png"

# 打开图片
image = Image.open(image_path)
print(f"Mode: {image.mode}")      # 输出: "RGBA"
print(f"Size: {image.size}")      # 输出: (1654, 2339)
print(f"Format: {image.format}")  # 输出: "PNG"

# 转换为 RGB
rgb_image = image.convert("RGB")
print(f"Mode: {rgb_image.mode}")  # 输出: "RGB"

# 内存中的变化
# RGBA: 1654 × 2339 × 4 = 15,462,904 bytes (~15 MB)
# RGB:  1654 × 2339 × 3 = 11,597,178 bytes (~11 MB)
# 节省了 25% 内存
```

#### **Image 对象的数据结构**

内部表示

```py
image = Image.open("page_001.png").convert("RGB")

# Image 对象的核心属性
image.width      # 1654 (像素)
image.height     # 2339 (像素)
image.mode       # "RGB"
image.format     # "PNG"
image.info       # {'dpi': (200, 200), ...}

# 像素数据的访问方式
```

像素数据的三种访问方式

方式 1：getpixel() / putpixel()（慢，适合调试）

```py
# 读取单个像素
pixel = image.getpixel((100, 200))  # (x, y)
print(pixel)  # 输出: (255, 255, 255)  # 白色

# 修改单个像素
image.putpixel((100, 200), (0, 0, 0))  # 改为黑色
```

方式 2：load()（较快）

```py
pixels = image.load()  # 返回 PixelAccess 对象

# 读取
pixel = pixels[100, 200]
print(pixel)  # (255, 255, 255)

# 修改
pixels[100, 200] = (0, 0, 0)
```

方式 3：转换为 NumPy 数组（最快，适合批量处理）

```
import numpy as np

# 转换为数组
arr = np.array(image)
print(arr.shape)  # (2339, 1654, 3)  # (height, width, channels)

# 访问像素
pixel = arr[200, 100]  # [y, x] 注意顺序！
print(pixel)  # [255, 255, 255]

# 批量操作（例如：二值化）
gray = np.mean(arr, axis=2)  # 转为灰度
binary = (gray < 128).astype(np.uint8) * 255

# 转回 Image 对象
binary_image = Image.fromarray(binary)
```

## new()

函数签名

```py
Image.new( mode, size, color )
```

**mode**：图片的模式。"1", "CMYK", "F", "HSV", "I", "L", "LAB", "P", "RGB", "RGBA", "RGBX", "YCbCr"

**size**：一个含有图片 宽，高 的元组；图片的尺寸(width, height) 

**color**：图片的颜色；其默认值为0，即 黑色；

**特别情况：当设置图片的 mode 为 ‘RGBA’ 时，如果不填 color 参数的话，图片是 透明底！** 
 即以下方法，建立的图片对象为 透明底！如果添加文字后，保存为 png 格式，你会得到一张透明底的文字图片！需要绘制透明底图的时，你会需要的。

## paste()

`Image.Image.paste()` 方法主要用于将一张图片或一个纯色块粘贴到另一张图片的指定位置上，是图像合成、添加水印、制作拼贴画等操作的基础工具

#### 1. `im` 参数：粘贴图像或纯色块

**粘贴图像**：`im`可以是一个通过`Image.open()`打开的PIL图像对象

```py
from PIL import Image

# 打开背景图和前景图
background = Image.open("background.jpg")
foreground = Image.open("sticker.png")
# 将前景图粘贴到背景图的 (50, 50) 位置
background.paste(foreground, (50, 50))
background.save("output.jpg")
```

- **粘贴纯色块**：`im`也可以是表示颜色的元组 (如RGB `(255, 0, 0)`)、颜色名 (如`"red"`) 或灰度值。**注意**：使用颜色时，**必须**通过`box`参数定义一个4元组来指定填充区域。

```py
from PIL import Image

# 创建一个纯白画布 (RGB模式)
canvas = Image.new("RGB", (200, 200), "white")
# 将左上角 100x100 的区域填充为红色
canvas.paste((255, 0, 0), (0, 0, 100, 100))
canvas.save("color_block.jpg")
```

#### 2. `box` 参数：控制粘贴位置与尺寸

`box`参数决定了`im`被放置在哪里，以及如何被放置。它有几种不同的用法，关键在于理解`im`与`box`定义的区域之间的关系。

粘贴图像（2元组 vs. 4元组）

```py
from PIL import Image

background = Image.new("RGB", (400, 300), "blue")
foreground = Image.new("RGB", (100, 100), "red")

# 方式一：使用2元组指定左上角位置，前景图保持原尺寸粘贴
background.paste(foreground, (50, 50))

# 方式二：使用4元组定义区域，前景图会被调整（缩放/拉伸）以适应区域
# 这里定义的区域为200x100，前景图将被拉伸以适应
background.paste(foreground, (150, 150, 350, 250))

background.save("paste_box_example.jpg")
```

 **重要细节**：当`box`为4元组时，其表示的坐标遵循Python左闭右开的原则。例如，`(0, 0, 100, 100)`定义了一个宽度为 `100 - 0 = 100`，高度为 `100 - 0 = 100` 的区域，但其实际像素范围是0到99。因此，一个100x100的图像恰好能完美填充。

#### 3. `mask` 参数：实现高级透明度控制

`mask`参数需要传入一个PIL `Image`对象，通常为"L"（灰度）或"1"（二值）模式。它的作用是充当一个“模板”：

- `mask`中像素值为**255**（或非零）的地方，表示“**不透明**”，前景图的对应像素会被粘贴上去。
- `mask`中像素值为**0**的地方，表示“**完全透明**”，前景图的对应像素**不会**被粘贴，背景图的像素得以保留。
- 介于0-255之间的值，可以产生半透明的过渡效果。

##### 常见场景：粘贴带透明通道的PNG图片

如果你粘贴一个具有透明背景的PNG（模式为'RGBA'），**必须**正确使用`mask`参数才能保持透明效果。最简单的方法是将同一个图像作为`mask`参数传入。

```py
from PIL import Image

# 打开背景 (RGB模式) 和带有透明通道的PNG前景 (RGBA模式)
background = Image.open("scene.jpg").convert("RGB")
watermark = Image.open("logo.png")

# 错误用法：不带mask参数粘贴，透明部分会显示为黑色或白色[reference:27]
# background.paste(watermark, (10, 10))

# 正确用法：将前景图自身作为mask参数，以保留其透明度
background.paste(watermark, (10, 10), mask=watermark)
background.save("watermarked_scene.jpg")
```

##### 进阶用法：使用独立蒙版

你可以创建一个独立的蒙版图像，更精细地控制粘贴区域的形状。

```py
from PIL import Image

background = Image.open("landscape.jpg")
foreground = Image.open("texture.jpg")
mask = Image.open("circle_mask.png").convert("L")  # 一个灰度图，白色圆形在黑色背景上

# 将纹理图以圆形区域粘贴到风景图上
background.paste(foreground, (50, 50), mask=mask)
background.save("masked_texture.jpg")
```

### ✨ 应用场景

- **添加水印**：在图片角落或中心添加Logo或文字水印，常用于版权保护。
- **制作拼贴画**：将多张图片裁剪并排列在一张大画布上，制作照片墙或艺术拼贴。
- **图像合成与修复**：替换图片中的特定区域，比如修复旧照片的破损部分或移除不需要的物体。
- **数据增强**：在机器学习中，将目标物体粘贴到不同的背景中，以扩充训练数据集。

```
 2x2 例子可以这样看。背景全是白色 RGB=(255,255,255)：

  背景 dst
  [(255,255,255)   (255,255,255)]
  [(255,255,255)   (255,255,255)]

  前景 src (RGBA)
  [(255,  0,  0,  0)   (255,  0,  0,128)]
  [(255,  0,  0,255)   (  0,128,255,128)]

  mask = alpha
  [  0   128]
  [255   128]

  逐像素算：

  (0,0): alpha = 0
  R = (255*0 + 255*255) / 255 = 255
  G = (  0*0 + 255*255) / 255 = 255
  B = (  0*0 + 255*255) / 255 = 255
  结果 = (255,255,255)

  (1,0): alpha = 128
  R = floor((255*128 + 255*127) / 255) = 255
  G = floor((  0*128 + 255*127) / 255) = 127
  B = floor((  0*128 + 255*127) / 255) = 127
  结果 = (255,127,127)

  (0,1): alpha = 255
  R = (255*255 + 255*0) / 255 = 255
  G = (  0*255 + 255*0) / 255 = 0
  B = (  0*255 + 255*0) / 255 = 0
  结果 = (255,0,0)

  (1,1): alpha = 128
  R = floor((  0*128 + 255*127) / 255) = 127
  G = floor((128*128 + 255*127) / 255) = 191
  B = floor((255*128 + 255*127) / 255) = 255
  结果 = (127,191,255)

  所以最后输出图就是：

  [(255,255,255)   (255,127,127)]
  [(255,  0,  0)   (127,191,255)]
```



## save()

```py
canvas = image.copy().convert("RGB")
draw = ImageDraw.Draw(canvas)

canvas.save(overlay_path)
```

canvas.save(overlay_path) 看起来只是一行简单的代码，但在底层（C 语言和 Pillow 的核心库中），它经历了一个复杂的**“编码-压缩-写入”**过程。

由于 canvas 是一个 PIL (Pillow) 的 Image 对象，它的 save 方法并不是直接把内存里的像素 dump 到硬盘上，而是根据文件后缀名（这里是 .png）选择对应的编码器。
以下是源码级别的机理分析：

1. Python 层：调度与参数准备

当你调用 canvas.save("page_001_overlay.png") 时：

- 格式探测：Pillow 会检查文件名后缀 .png，确定目标格式是 PNG。
- 查找编码器：它在内部的 SAVE 字典中找到处理 PNG 的函数（通常关联到 _png 模块）。
- 预处理：它会检查图像的模式（我们之前做了 convert("RGB")），确保数据布局符合编码器的要求。

2. C 语言层：核心编码流程 (基于 libpng)

Pillow 的底层大量使用 C 语言编写以追求性能。对于 PNG 保存，它调用了系统安装的 libpng 库和 zlib 库。

第一步：初始化 PNG 结构

```c
// 伪代码：Pillow 内部 C 实现逻辑
png_structp png_ptr = png_create_write_struct(...);
png_infop info_ptr = png_create_info_struct(...);
FILE *fp = fopen("page_001_overlay.png", "wb");
png_init_io(png_ptr, fp);
```

第二步：写入头部信息 (IHDR Chunk)
PNG 文件开头必须包含图片的基本属性。

```py
// 写入宽、高、位深(8bit)、颜色类型(RGB)、压缩方式等
png_set_IHDR(png_ptr, info_ptr, 
    width, height, 8, 
    PNG_COLOR_TYPE_RGB, 
    PNG_INTERLACE_NONE, ...);
png_write_info(png_ptr, info_ptr);
```

第三步：数据转换与逐行处理 (关键步骤)

这是最耗时的部分。Pillow 不会一次性把所有像素扔给 libpng，而是**逐行（Scanline）**处理。

1. 获取原始数据：从 canvas 的 image->data 指针处获取一行像素的 RGB 字节流。
2. 滤波 (Filtering)：PNG 为了提高压缩率，会在压缩前对每一行进行“滤波”处理（如 Sub filter, Up filter 等）。它会计算当前像素与前一个像素或上方像素的差值。
3. 原理：如果背景是白色的，相邻像素的值非常接近，差值会产生大量的 0，这非常利于后续的压缩。
   写入行数据：

```c
    for (int y = 0; y < height; y++) {
        unsigned char *row = get_row_from_canvas(canvas, y);
        png_write_row(png_ptr, row); // 交给 libpng 处理
    }
```

第四步：压缩与写入 (IDAT Chunk)

在 png_write_row 内部，zlib 算法开始工作：

- Deflate 算法：它将经过滤波处理的行数据进行 LZ77 压缩和霍夫曼编码。
- 分块存储：压缩后的数据被放入 PNG 的 IDAT 数据块中。这就是为什么 PNG 文件比原始 BMP 小得多的原因。

第五步：结束标记 (IEND Chunk)

```c
png_write_end(png_ptr, NULL);
fclose(fp);
```

写入一个空的 IEND 块，告诉阅读器图片结束了。

<<<<<<< HEAD
## crop()

```py
Image.crop(box=None)
```

**参数 `box`**：这是一个包含 4 个数字的**元组 (Tuple)**，格式为 `(left, upper, right, lower)`。

**返回值**：返回一个新的 `Image` 对象，原图不会被修改。
=======
## histogram()

一、核心作用

histogram() 返回一个包含 256 个整数的列表，表示图像中每个灰度级（0-255）的像素数量。

二、返回值详解

1. 基本结构

```py
from PIL import Image

# 打开一张灰度图或转为灰度
image = Image.open("test.png").convert("L")  # "L" 模式 = 灰度图

# 获取直方图
histogram = image.histogram()

print(f"类型: {type(histogram)}")  # <class 'list'>
print(f"长度: {len(histogram)}")   # 256
print(f"内容: {histogram[:10]}")   # 前10个值
```

2. 数据含义

```py
# histogram 是一个长度为 256 的列表
# 索引 i 表示灰度值 i
# 值 histogram[i] 表示灰度值为 i 的像素数量

# 示例：
histogram = [
    100,   # histogram[0]   → 有 100 个像素的灰度值是 0（纯黑）
    50,    # histogram[1]   → 有 50 个像素的灰度值是 1
    30,    # histogram[2]   → 有 30 个像素的灰度值是 2
    ...,
    200,   # histogram[127] → 有 200 个像素的灰度值是 127（中灰）
    ...,
    150,   # histogram[255] → 有 150 个像素的灰度值是 255（纯白）
]

# 所有值的总和 = 图像的总像素数
total_pixels = sum(histogram)
print(f"总像素数: {total_pixels}")  # 例如: 1000000 (1000x1000 图片)
```
>>>>>>> e1ba353 (Ubuntu 4.25)



# ImageDraw 库

## draw()

```py
    canvas = image.copy().convert("RGB")
    draw = ImageDraw.Draw(canvas)
```

这是 Pillow 库中用于在图像上进行 2D 绘图的标准接口。
	设计模式：它采用了一种“画笔”或“上下文”的设计模式。
	工作原理：
		draw = ImageDraw.Draw(canvas)：这一步并不是创建了一张新图，而是创建了一个绘图工具对象，这个对象手			里拿着对 canvas 内存区域的引用。
		draw.rectangle(...)：当你调用这个方法时，这个工具会直接去修改 canvas 对应坐标位置的像素数据。

类比理解：
	canvas 是一张画布。
	ImageDraw.Draw(canvas) 是你手里的画笔套装。
	draw.rectangle() 是你用画笔在画布上涂颜料的动作。

# ImageOps库

## exif_transpose()

一、核心作用

根据图片的 EXIF 元数据中的方向标记（Orientation Tag），自动将图片旋转到正确的显示方向。

二、为什么需要这个函数？

问题背景：手机拍照的方向问题

```py
场景：你用手机竖着拍了一张照片
       ↓
手机传感器实际是横向的（landscape）
       ↓
照片保存时：
  - 像素数据：横向存储（宽 > 高）
  - EXIF 标记：Orientation = 6（旋转90度）
       ↓
在不支持 EXIF 的软件中打开：
  ❌ 图片显示为横向（需要手动旋转）
  
在支持 EXIF 的软件中打开：
  ✅ 图片自动旋转为竖向（正确方向）
```

三、EXIF Orientation 详解

8 种可能的方向值

```py
# EXIF Orientation 标签的值及其含义

Orientation = 1  # 正常（不需要旋转）
┌─────┐
│ ABC │  ← 文字正向
└─────┘

Orientation = 2  # 水平翻转
┌─────┐
│ CBA │  ← 左右镜像
└─────┘

Orientation = 3  # 旋转 180 度
┌─────┐
│ ∀ᗺƆ │  ← 倒置
└─────┘

Orientation = 4  # 垂直翻转
┌─────┐
│ ∀ᗺƆ │  ← 上下镜像
└─────┘

Orientation = 5  # 逆时针 90 度 + 水平翻转
┌─┐
│A│
│B│
│C│
└─┘

Orientation = 6  # 顺时针 90 度（最常见！）
┌─┐
│C│
│B│
│A│
└─┘
↑ 手机竖拍时的典型值

Orientation = 7  # 逆时针 90 度 + 垂直翻转
┌─┐
│C│
│B│
│A│
└─┘

Orientation = 8  # 逆时针 90 度
┌─┐
│A│
│B│
│C│
└─┘
```

