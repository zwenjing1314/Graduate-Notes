# 安装

```bash
pip install pathlib
```

# 常用方法

## iterdir()

列出这个目录下的**所有“第一层”子项**（包括文件和文件夹，不会递归进入子目录）。

```py
from pathlib import Path

dir_path = Path("../data")
for item in dir_path.iterdir():
    print(item)
```

## is_file()

判断某个路径是不是**普通文件**，排除目录。

```py
from pathlib import Path

item = Path("../data/images/1.jpeg")
print(item.is_file())

item = Path("../data/images")
print(item.is_file)  # False 因为images是一个文件夹
print(item.is_file)  # <bound method Path.is_file of WindowsPath('../data/images')>
print(type(item.is_file))  # <class 'method'>
```

## glob("*.jpeg")

列出当前目录下**所有匹配某种模式的文件/文件夹**（也只看第一层，不会进子目录）

```py
from pathlib import Path

dir_path = Path("../data/images")
for item in dir_path.glob("*.jpeg"):
    print(item)
```

## rglob("*.jpeg")

列出当前目录及子目录下**所有匹配某种模式的文件/文件夹**

```
from pathlib import Path

dir_path = Path("../data/images")
for item in dir_path.rglob("*.jpeg"):
    print(item)
```

## suffix()

**作用**：提取文件名的扩展名，并且**包含前面的点号（`.`）**。

**如果没有扩展名**：返回一个空字符串 `""`。

**如果有多个扩展名**（比如 `backup.tar.gz`）：它只会返回**最后一个**扩展名（即 `.gz`）。

```py
from pathlib import Path

print(Path("example.txt").suffix)       # 输出: .txt
print(Path("archive.tar.gz").suffix)    # 输出: .gz  (只取最后一个)
print(Path("my_folder/script.py").suffix) # 输出: .py
print(Path("README").suffix)            # 输出: ""   (没有后缀)
```

## mkdir()

主要用于在指定路径上创建目录

```py
from pathlib import path
path.mkdir(parents=True, exist_ok=True)
```

```
path.parent.mkdir(parents=True, exist_ok=True)
```

parents 如果为 `True`，则会创建缺失的父目录。如果为 `False`，则父目录必须已存在，否则抛出 `FileNotFoundError`。

exist_ok 如果为 `True`，则忽略目录已存在的情况，不会抛出异常。如果为 `False`，当目录已存在时抛出 `FileExistsError`。

## reslove()

作用：将路径转换为绝对路径，并解析所有符号链接和相对引用

```py
# __file__ 是一个特殊变量，表示当前脚本文件的路径

project_root = Path(__file__).resolve().parents[1]
```

实例

```py
from pathlib import Path

# 场景 1：处理相对路径
p1 = Path("scripts/inspect_docvqa_train.py")
print(p1)        # 输出：scripts/inspect_docvqa_train.py (相对路径)
print(p1.resolve())  # 输出：/home/wenjing/文档/PythonProjects/doc-multimodal-vqa/scripts/inspect_docvqa_train.py (绝对路径)

# 场景 2：处理包含 .. 和 . 的路径
p2 = Path("./scripts/../scripts/./inspect_docvqa_train.py")
print(p2)        # 输出：./scripts/../scripts/./inspect_docvqa_train.py
print(p2.resolve())  # 输出：/home/wenjing/文档/PythonProjects/doc-multimodal-vqa/scripts/inspect_docvqa_train.py
                   #       (自动简化了 .. 和 .)

# 场景 3：处理符号链接（类似快捷方式）
# 假设 /tmp/link 是指向 /home/user/real_file.py 的符号链接
p3 = Path("/tmp/link")
print(p3)           # 输出：/tmp/link
print(p3.resolve()) # 输出：/home/user/real_file.py (解析到真实文件)
```

# 常用属性

## parent

作用：获取所有父目录的序列（类似一个只读的列表）

```py
from pathlib import Path

p = Path("/home/wenjing/文档/PythonProjects/doc-multimodal-vqa/scripts/inspect_docvqa_train.py")

# parents 是一个序列，包含从直接父目录到根目录的所有上级目录
print(p.parents[0])  # 输出：/home/wenjing/文档/PythonProjects/doc-multimodal-vqa/scripts
                     #       (直接父目录，相当于 p.parent)

print(p.parents[1])  # 输出：/home/wenjing/文档/PythonProjects/doc-multimodal-vqa
                     #       (祖父目录，往上两级)

print(p.parents[2])  # 输出：/home/wenjing/文档/PythonProjects
                     #       (曾祖父目录，往上三级)

print(p.parents[3])  # 输出：/home/wenjing/文档

print(p.parents[4])  # 输出：/home/wenjing

print(p.parents[5])  # 输出：/home

print(p.parents[6])  # 输出：/ (根目录)

# 访问不存在的索引会报错
# print(p.parents[7])  # IndexError: tuple index out of range
```

##  name

获取路径的最后一部分，包含扩展名。

```py
image_name = "documents/xnbl0037_1.png"
image_name = Path(image_name).name  # xnbl0037_1.png
```



## stem

获取路径的最后一部分，但不包含扩展名（仅主文件名）。

```
image_name = "documents/xnbl0037_1.png"
ocr_name = Path(image_name).stem + ".json"  # xnbl0037_1.json
```

