## conda 环境

```bash
conda create -n forgeflow python=3.11
conda activate forgeflow
pip install -e '.[dev]'
uvicorn app.main:app --reload
```



## ven 环境

```py
python3 -m venv .venv
source .venv/bin/activate
pip install -e '.[dev]'
uvicorn app.main:app --reload
```

查看pytorch版本

在终端中查看 PyTorch 版本，可以使用以下任一命令：

```
python -c "import torch; print(torch.__version__)"
```

或者通过 pip 查看：

```
pip show torch
```

如果使用 Conda 环境，也可以运行：

```
conda list torch
```

### 显示下载的包的版本

```bash
pip show 包名
```

### 升级包到最新版本

```bash
pip install --upgrade 包名
```

### 指定版本号升级

```bash
pip install --upgrade numpy==2.1.3
```

### 查看cuda版本

```
nvcc --version
```

```
nvidia-smi
```

### pip install -e .

*pip install -e .* 是一个用于在开发环境中安装 Python 包的命令。这个命令中的 *-e* 选项表示“editable”，即可编辑模式，而 *.* 表示当前目录。

可编辑模式

在可编辑模式下安装包意味着你可以在不重新安装的情况下对包进行修改。这对于开发者来说非常有用，因为它允许你在开发过程中实时测试和调试代码，而不需要每次修改后都重新安装包。·

