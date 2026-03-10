# 参见命令参数

## mkdir

```
mkdir -p ~/workspace/admin
{
  date
  echo "===== OS ====="
  lsb_release -a
  echo "===== KERNEL ====="
  uname -a
  echo "===== GPU ====="
  nvidia-smi
  echo "===== CUDA TOOLKIT ====="
  nvcc --version || true
  echo "===== CONDA ====="
  conda info
  echo "===== CONDA ENVS ====="
  conda env list
  echo "===== PYTHON ====="
  python --version
  which python
  echo "===== DISK ====="
  df -h
  echo "===== MEMORY ====="
  free -h
} > ~/workspace/admin/machine_audit_2026-03-09.txt 2>&1
```

1. **`mkdir -p ~/workspace/admin`**
   - **命令的作用是递归创建多级目录结构，自动创建路径中不存在的父目录。**‌
   
   - `-p`（参数）：`--parents` 的缩写，表示**递归创建目录**。如果父目录不存在，会自动创建；如果目标目录已存在，也不会报错。
   
2. **`{ ... }`（大括号）**

   - 作用：将多个命令组合成一个**命令组**，在当前 Shell 中顺序执行。组内所有命令的标准输出和错误输出可以被统一重定向。

   - 语法：`{ command1; command2; }`（注意大括号前后需要有空格，命令之间用分号 `;` 或换行分隔）。

3. ** 命令组内的具体命令**

   - **`date`**：打印当前日期和时间。

   - **`echo "===== OS ====="`**：输出一个分隔标题，标识操作系统信息部分。

   - **`lsb_release -a`**：显示 Linux 发行版的详细信息（如发行版名称、版本号、代号等）。`-a` 表示显示所有可用信息。

   - **`echo "===== KERNEL ====="`**：输出另一个标题，但注意后面**没有**实际获取内核版本的内核命令（如 `uname -a`），这可能是遗漏或只是作为占位。

4. **重定向部分 `> > ~/workspace/admin/1.txt 2>&1`**
   - **`>`**：标准输出重定向符号。如果单独使用，如 `command > file`，会将命令的标准输出写入到文件中（覆盖原内容）
   - **`>>`**：标准输出**追加**重定向符号，将输出追加到文件末尾。
   - **`2>&1`**：将标准错误输出（文件描述符 2）重定向到标准输出（文件描述符 1）所在的位置。这里结合前面的重定向，意味着**无论标准输出还是标准错误，都会写入同一个文件**。

## ls

```
ls -lah
```

**`-l`**‌：以长格式（详细信息）显示文件和目录

**`-a`**‌：显示‌**所有文件**‌，包括以 `.` 开头的‌**隐藏文件**‌

**`-h`**‌：与 `-l` 配合使用时，将文件大小转换为‌**人类可读格式**‌（如 KB）

**常见组合用法**

- `ls -la`：列出所有文件（含隐藏）的详细信息（大小为字节）
- `ls -lh`：列出详细信息，大小人性化显示（不含隐藏文件）
- `ls -ltr`：按修改时间升序排列（最旧在前）
- `ls -lS`：按文件大小降序排列（最大在前）

## cd

```
cd ~/workspace
```

## find

`find`命令用于在文件系统中查找文件和目录

```
find ~/workspace -maxdepth 2 -type d

/home/wenjing/workspace
/home/wenjing/workspace/admin
```

深度不超过2的所有目录，可以使用`-maxdepth`选项

使用`-type d`确保只查找目录（不包含文件）

## grep

`grep`命令用于搜索文件中匹配指定模式的行

```
grep -n "CUDA" ~/workspace/admin/machine_audit_2026-03-09.txt

12:| NVIDIA-SMI 570.195.03             Driver Version: 570.195.03     CUDA Version: 12.8     |
35:===== CUDA TOOLKIT =====
```

如果你想要在目录中递归搜索包含"CUDA"的文件，你可以使用`grep`的`-r`或`-R`选项

## tail

`tail` 命令用于输出文件的最后部分内容

```
tail -n 30 ~/workspace/admin/machine_audit_2026-03-09.txt
```

**-n**‌ 选项后面跟数字，指定要显示的行数。

# tree

一个用于以树状图形式递归列出目录内容的命令行工具

```
tree -L 2 ~/workspace
```

**`-L 2`**‌：限制显示深度为 ‌**2 层**‌，即只显示 `~/workspace` 本身及其直接子目录（第一层）和这些子目录下的内容（第二层），不再深入。

**`~/workspace`**‌：指定要遍历的起始目录，即当前用户主目录下的 `workspace` 文件夹。

若想将结果保存到文件，可使用重定向：

```bash
tree -L 2 ~/workspace > workspace_structure.txt
```

**常用相关选项（补充）**

- `-a`：显示所有文件，包括隐藏文件（以 `.` 开头）。
- `-d`：仅显示目录，不显示文件。
- `-f`：显示每个文件的完整路径。
- `-h`：以人类可读方式显示文件大小（如 KB、MB）。
