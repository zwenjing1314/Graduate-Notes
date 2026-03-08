# 安装

```bash
pip install pillow
```

# Pillow库的简单应用

Pillow 库提供了丰富的功能模块，如 *Image* 图像处理模块、*ImageFont* 添加文本模块、*ImageColor* 颜色处理模块、*ImageDraw* 绘图模块等。以下是一些常见的图像处理操作示例：

**读取和保存图片**

```py
from PIL import Image
# 读取图片
image = Image.open('example.jpg')
# 保存图片
image.save('output.png')
```

**调整图像大小**