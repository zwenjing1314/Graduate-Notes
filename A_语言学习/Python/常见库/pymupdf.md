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
