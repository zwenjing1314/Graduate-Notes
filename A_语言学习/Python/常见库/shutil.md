# 常见用法

## shutil.copy(src, dst, follow_symlinks=True)

- **src** (str): 源文件的路径（必须是一个文件，不能是目录）。
- **dst** (str): 目标路径，可以是一个文件路径或目录路径。
  - 如果 `dst` 是一个目录，则会将文件复制到该目录下，并保留原文件名。
  - 如果 `dst` 是一个文件路径，则会将源文件复制并重命名为该文件名（若文件已存在则覆盖）。
- **follow_symlinks** (bool): 默认为 `True`。若为 `True`，且 `src` 是符号链接，则复制链接指向的文件；若为 `False`，则创建一个新的符号链接（指向原链接的目标）。

#### 复制文件到另一个文件（重命名）

```py
import shutil

shutil.copy('source.txt', 'dest.txt')  # 将 source.txt 复制为 dest.txt
```

#### 复制文件到目录

```py
import shutil

shutil.copy('source.txt', '/path/to/destination/')  # 复制到目录，保持原文件名
# 注意：路径末尾的斜杠可选，但建议加上以明确表示目录
```

