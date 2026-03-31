# 介绍

os.path 模块主要用于获取文件的属性。

**在现代 Python（3.4+）中，`pathlib` 已经成为官方推荐的路径处理方式**，它并不是直接“废弃”了 `os.path`，而是提供了一个**更面向对象、可读性更好、跨平台更方便**的替代方案。

# 常用API

## listdir()

`os.listdir()` 是 **os** 模块提供的一个函数，用于获取指定目录下的所有文件和文件夹名称（不递归进入子目录）。

### 基本语法

```py
import os

os.listdir(path='.')
```

**参数说明**：

- `path` *(可选)*：要列出内容的目录路径，默认为当前目录 `"."`。
- 返回值：**列表**（`list`），包含目录下的文件和文件夹名称（字符串），不包含 `.` 和 `..`。

### 基本示例

```py
import os

# 列出当前目录下的所有文件和文件夹
items = os.listdir('.')
print(items)
```

### 结合路径使用

```py
import os

folder_path = '/path/to/your/folder'

try:
    for name in os.listdir(folder_path):
        full_path = os.path.join(folder_path, name)  # 拼接完整路径
        if os.path.isfile(full_path):
            print(f"文件: {name}")
        elif os.path.isdir(full_path):
            print(f"文件夹: {name}")
except FileNotFoundError:
    print("目录不存在")
except PermissionError:
    print("没有权限访问该目录")
```

### 常见用途

**批量处理文件**（如批量重命名、格式转换）。

**过滤特定类型文件**：

```py
import os

files = [f for f in os.listdir('.') if f.endswith('.txt')]
print(files)
```

**检查目录内容**（如判断是否存在某个文件）。

### 注意事项

`os.listdir()` **不会递归**进入子目录，如果需要递归，可以用 `os.walk()` 或 `pathlib.Path.rglob()`。

返回的文件名是**不带路径**的，需要用 `os.path.join()` 拼接完整路径。

如果目录不存在，会抛出 `FileNotFoundError`；如果没有权限，会抛出 `PermissionError`。



# os.path 模块

### os.path.basename(path) 

返回文件名

### os.path.split(path) 

把路径分割成 dirname 和 basename，返回一个元组

### os.path.splitext(path) 

分割路径，返回路径名和文件扩展名的元组

### os.path.dirname(path) 

返回文件路径

### os.path.join(path1[, path2[, ...]]) 

把目录和文件名合成一个路径

示例

```py
#!/usr/bin/python
# -*- coding: UTF-8 -*-
 
import os
 
print( os.path.basename('/root/runoob.txt') )   # 返回文件名
print( os.path.dirname('/root/runoob.txt') )    # 返回目录路径
print( os.path.split('/root/runoob.txt') )      # 分割文件名与路径
print( os.path.join('root','test','runoob.txt') )  # 将目录和文件名合成一个路径

“”“
runoob.txt
/root
('/root', 'runoob.txt')
root/test/runoob.txt
“”“
```

