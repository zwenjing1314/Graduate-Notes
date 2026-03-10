win终端输入

cmd进入cmd命令，再输入exit退出到powershell

# cmd命令

## 基础指令

### 在PowerShell中删除目录

```cmd
rmdir /s /q "C:\Users\WenJing\Documents\PytorchProjects\CLIP_Stu\artifacts"
```

-  /s 表示删除指定目录及其中的所有子目录和文件
-  /q 表示静默模式，不询问确认

# PowerShell命令

## 基础指令

### 在PowerShell中删除目录

使用 `Remove-Item` 命令并带上 `-Recurse`（递归删除子目录和文件）和 `-Force`（强制删除只读文件）参数：

```powershell
Remove-Item -Recurse -Force "C:\Users\WenJing\Documents\PytorchProjects\CLIP_Stu\artifacts"
```

或者使用短别名和参数缩写：

```powershell
rm -r -fo "C:\Users\WenJing\Documents\PytorchProjects\CLIP_Stu\artifacts"
```

- `-r` 或 `-Recurse`：删除目录及其所有内容
- `-fo` 或 `-Force`：强制删除，包括隐藏和只读文件