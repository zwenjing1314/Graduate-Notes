

# 介绍

标准库中的一个模块，专门用于执行高级文件操作。它提供了一系列用于复制、移动、删除和压缩文件和文件夹的函数，补充了 os 模块的功能。

# 常见API

## copy()

### shutil.copy(src, dst, follow_symlinks=True)

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

## rmtree()

### shutil.rmtree(job.upload_dir, ignore_errors=True)

 删除目录及其所有内容（类似 rm -rf）

示例

```py
import shutil
from pathlib import Path

# 假设目录结构：
# uploads/b86a192c21f7/
# ├── source.pdf
# └── some_other_file.txt

upload_dir = Path("uploads/b86a192c21f7")

# ❌ os.remove() 只能删除文件
os.remove(upload_dir)  # IsADirectoryError: 不能删除目录

# ❌ os.rmdir() 只能删除空目录
os.rmdir(upload_dir)  # OSError: 目录不为空

# ✅ shutil.rmtree() 递归删除整个目录树
shutil.rmtree(upload_dir)  # 成功删除目录及所有文件
```

### ignore_errors=True 的作用

```py
shutil.rmtree(job.upload_dir, ignore_errors=True)
#                          ↑
#                   忽略所有错误
```

为什么需要这个参数？

示例

```py
# 场景 1：任务失败，从未创建上传目录
job.upload_dir = Path("uploads/failed_job")
# 目录不存在

shutil.rmtree(job.upload_dir, ignore_errors=False)
# FileNotFoundError: [Errno 2] No such file or directory: 'uploads/failed_job'

shutil.rmtree(job.upload_dir, ignore_errors=True)
# 静默跳过，程序继续运行 ✓

# 场景 2：并发请求，另一个线程已经删除了目录
# 线程A: 开始清理
# 线程B: 也开始清理（竞态条件）
# 线程A: 删除成功
# 线程B: 目录已不存在

shutil.rmtree(job.upload_dir, ignore_errors=True)
# 静默跳过，不会崩溃 ✓
```

