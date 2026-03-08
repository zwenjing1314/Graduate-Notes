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

## parent()

以字符串的形式返回作为参数传递的路径的父目录

```py
from pathlib import Path

path = Path(r"C:\Users\WenJing\Documents\Share\PythonProjects\DailyUse\month3\demo02_concateenate.py")
print(path.parent)

# C:\Users\WenJing\Documents\Share\PythonProjects\DailyUse\month3
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
