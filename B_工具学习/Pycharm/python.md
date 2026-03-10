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

