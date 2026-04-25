# 介绍

PyMuPDF是一个高性能的Python库，用于PDF（及其他）文档的数据提取、分析、转换和操作。它具有极速处理能力、全格式支持和精准布局保留等核心优势。PyMuPDF特别适合科研文献分析和金融报告自动化等场景，能显著提升文档处理效率。

https://pymupdf.cn/en/latest/the-basics.html

# 常见API

## open()

打开一个文件

```py
import pymupdf

doc = pymupdf.open("a.pdf") # open a document
```

返回值: 一个 Document 对象（在 PyMuPDF 中通常称为 fitz.Document）

返回的 doc 对象的核心特性

(1) 可迭代性 - 遍历页面

```py
for page_index, page in enumerate(doc):
    # 每个 page 是一个 Page 对象
    pix = page.get_pixmap(matrix=matrix, alpha=False)
```

doc 支持直接迭代，每次返回一个 Page 对象
可以使用 enumerate() 获取页码索引

(2) 页面数量

```py
len(doc)  # 获取总页数
```

(3) 通过索引访问特定页

```py
page = doc[0]      # 第一页
page = doc[-1]     # 最后一页
```

(4) 必须关闭资源

```py
try:
    # 处理文档...
finally:
    doc.close()  # 释放文件句柄和内存
```

重要: 使用完毕后必须调用 close() 方法
代码中使用 try-finally 确保即使出错也能正确关闭

### Page 对象的完整结构

```py
# 基本属性
page.number              # 页码（从0开始）
page.parent              # 所属的 Document 对象
page.rect                # 页面矩形 (Rect对象)，包含 width, height
page.mediabox            # PDF媒体框（原始尺寸，不随旋转变化）
page.cropbox             # 裁剪框（可见区域）
page.rotation            # 旋转角度（0, 90, 180, 270）

# 矩阵变换相关
page.rotation_matrix     # 旋转矩阵（用于坐标转换）
page.derotation_matrix   # 反旋转矩阵
page.transformation_matrix  # PDF与MuPDF空间转换矩阵

# 内容访问（生成器）
page.annots()            # 批注迭代器
page.widgets()           # 表单字段迭代器
page.links()             # 链接迭代器
```

2. Rect 对象结构

```py
rect = page.rect
rect.x0, rect.y0         # 左上角坐标
rect.x1, rect.y1         # 右下角坐标
rect.width               # 宽度
rect.height              # 高度
rect.tl                  # 左上角点 (x0, y0)
rect.br                  # 右下角点 (x1, y1)
```

3. 常用方法分类

```py
# ========== 渲染相关 ==========
page.get_pixmap()        # 渲染为位图（Raster Image）
page.get_svg_image()     # 渲染为矢量图（SVG）

# ========== 文本提取 ==========
page.get_text("text")    # 纯文本
page.get_text("blocks")  # 文本块
page.get_text("words")   # 词级信息
page.get_text("dict")    # 完整结构化数据

# ========== 图像提取 ==========
page.get_images()        # 获取嵌入的图像列表
page.get_image_info()    # 图像元信息
page.get_image_bbox()    # 图像位置

# ========== 图形提取 ==========
page.get_drawings()      # 矢量图形
page.find_tables()       # 检测表格

# ========== 修改操作（仅PDF）==========
page.insert_image()      # 插入图片
page.insert_text()       # 插入文字
page.draw_rect()         # 绘制矩形
# ... 更多绘图方法
```

### get_pixmap()

1. 函数签名

```py
page.get_pixmap(
    *,
    matrix=fitz.Identity,     # 变换矩阵（缩放、旋转、平移）
    dpi=None,                  # 目标DPI（会覆盖matrix）
    colorspace=fitz.csRGB,     # 颜色空间（RGB/GRAY/CMYK等）
    clip=None,                 # 裁剪区域（只渲染部分页面）
    alpha=False,               # 是否包含透明通道
    annots=True                # 是否渲染批注
)
```

2. 工作原理（底层流程）

```py
┌─────────────────────────────────────────────────┐
│  PDF 页面（矢量格式）                              │
│  - 文字（字体+编码）                               │
│  - 路径（贝塞尔曲线）                              │
│  - 图像（嵌入的bitmap）                           │
│  - 图形状态（颜色、线宽等）                        │
└──────────────┬──────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────┐
│  1. 应用 Matrix 变换                              │
│  - 缩放：zoom × zoom                             │
│  - 旋转：如果有 rotation                         │
│  - 平移：如果需要偏移                             │
└──────────────┬──────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────┐
│  2. MuPDF 引擎光栅化（Rasterization）             │
│  - 将矢量指令转换为像素                           │
│  - 抗锯齿处理                                    │
│  - 字体渲染（Hinting）                           │
│  - 颜色空间转换                                  │
└──────────────┬──────────────────────────────────┘
               │
               ▼
┌─────────────────────────────────────────────────┐
│  3. 生成 Pixmap 对象                             │
│  - samples: bytes 数组（原始像素数据）            │
│  - width, height: 像素尺寸                       │
│  - stride: 每行字节数                            │
│  - n: 每个像素的通道数（RGB=3, RGBA=4）          │
└─────────────────────────────────────────────────┘
```

3. 返回的 Pixmap 对象结构

```py
pix = page.get_pixmap(matrix=matrix, alpha=False)

# Pixmap 的核心属性
pix.width              # 图像宽度（像素）
pix.height             # 图像高度（像素）
pix.stride             # 每行的字节数（可能包含padding）
pix.n                  # 每个像素的通道数（3=RGB, 4=RGBA）
pix.alpha              # 是否有透明通道（0或1）
pix.colorspace         # 颜色空间对象
pix.samples            # 原始像素数据（bytes对象）
pix.xres, pix.yres     # 水平和垂直分辨率（DPI）

# Pixmap 的方法
pix.save("output.png")           # 保存为文件
pix.pil_save("output.png")       # 使用Pillow保存
pix.tobytes()                    # 转换为bytes
pix.pixel(x, y)                  # 获取某点颜色
pix.set_pixel(x, y, color)       # 设置某点颜色
```

4. samples 数据的内存布局

```py
# 假设 RGB 图像，width=100, height=50
# stride = width × 3 = 300 bytes（可能有padding对齐）

samples 内存布局：
┌──────────────────────────────────────┐
│ Row 0: [R,G,B, R,G,B, ..., R,G,B]  │  ← 300 bytes
│ Row 1: [R,G,B, R,G,B, ..., R,G,B]  │  ← 300 bytes
│ ...                                  │
│ Row 49:[R,G,B, R,G,B, ..., R,G,B]  │  ← 300 bytes
└──────────────────────────────────────┘
总大小 = stride × height = 300 × 50 = 15000 bytes
```



## Matrix()

1. Matrix 的本质:仿射变换矩阵
PyMuPDF 的 Matrix 是一个 3×3 仿射变换矩阵,用于坐标系统之间的转换:
```text
| a  b  e |
| c  d  f |
| 0  0  1 |
```

在代码中创建时:
```python
matrix = pymupdf.Matrix(zoom, zoom)
# 等价于:
matrix = pymupdf.Matrix(a=zoom, b=0, c=0, d=zoom, e=0, f=0)
```

完整形式:
```text
| zoom  0    0  |
| 0     zoom 0  |
| 0     0    1  |
```

2. 六个参数的含义
```python
pymupdf.Matrix(a, b, c, d, e, f)
```
a: X 轴缩放(水平)
b: Y 轴倾斜(垂直剪切)
c: X 轴倾斜(水平剪切)
d: Y 轴缩放(垂直)
e: X 轴平移(水平移动)
f: Y 轴平移(垂直移动)

3. 工作原理:坐标变换

当你调用 page.get_pixmap(matrix=matrix) 时,PyMuPDF 执行以下操作:
```python
# 伪代码展示内部流程
for each_point_in_pdf(x_pdf, y_pdf):
    # 应用矩阵变换
    x_pixel = a * x_pdf + b * y_pdf + e
    y_pixel = c * x_pdf + d * y_pdf + f
    
    # 渲染到像素图的对应位置
    pixmap.set_pixel(x_pixel, y_pixel, color)
```

实际例子:
```python
# PDF 中的一个点 (100, 200)
# 使用 Matrix(2.78, 2.78) 变换
x_pixel = 2.78 * 100 + 0 * 200 + 0 = 278
y_pixel = 0 * 100 + 2.78 * 200 + 0 = 556

# 结果:PDF 的点 (100, 200) → 像素图的点 (278, 556)
```

4. Matrix 的强大功能
除了缩放,Matrix 还可以做
```python
# 1. 旋转 90 度
rotate_90 = pymupdf.Matrix(0, 1, -1, 0, 0, 0)

# 2. 平移 100 单位
translate = pymupdf.Matrix(1, 0, 0, 1, 100, 100)

# 3. 组合变换:先放大 2 倍,再平移 100
combined = pymupdf.Matrix(2, 0, 0, 2, 100, 100)

# 4. 非等比缩放(拉伸)
stretch = pymupdf.Matrix(2, 0, 0, 1, 0, 0)  # X 轴放大 2 倍,Y 轴不变
```

