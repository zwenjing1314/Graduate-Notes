# 介绍

fitz 是 PyMuPDF 库的模块名，它是一个用于处理 PDF 和 XPS 文档的 Python 库。PyMuPDF 基于 MuPDF 渲染引擎，提供了非常强大的 PDF 操作功能。

# 常见API

## open()

### 创建/打开 PDF 文档

```py
doc = fitz.open()
```

fitz.open() 不带参数时，会创建一个新的空白 PDF 文档对象
这是一个 Document 类的实例

```py
doc = fitz.open(pdf_path)
```

带路径参数时，会打开已有的 PDF 文件
返回一个可迭代的文档对象，包含所有页面

### 创建新页面

```py
doc = fitz.open()
page1 = doc.new_page()
```

doc.new_page() 在文档末尾添加一个新页面
默认页面大小为 A4（595x842 点）
可以指定参数：doc.new_page(width=595, height=842)

### 在页面上插入文本

```py
page1.insert_text(
    (72, 72),  # 坐标点 (x, y)，单位是点（1/72 英寸）
    "A Small Paper Title\n\nAbstract\nThis is a short abstract for testing."
)
```

insert_text(point, text) 在指定坐标插入文本
坐标 (72, 72) 表示距离左下角 1 英寸的位置（PDF 坐标系原点在左下角）
支持换行符 \n

### 保存文档

```
doc.save(pdf_path)
```

将文档保存到指定路径
如果是新建的文档，这会将其写入磁盘
