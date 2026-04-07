## Conda 环境的克隆

### 先做一个基础环境，然后克隆

这在本机上通常最快。`conda` 官方支持直接克隆现有环境：

```bash
conda create -n torch-base python=3.11
conda activate torch-base
# 装好 torch / torchvision / torchaudio / facenet-pytorch 等

conda create -n face-proj --clone torch-base
```

这样新环境通常不需要再从头下载一遍大包。

### **要跨机器复用，打包整个环境**

 如果你想把一个“完整可运行环境”搬到另一台机器，`conda-pack` 很适合。它就是专门把现有 conda 环境打成归档包，然后在别处解包使用的。

```bash
pip install conda-pack
conda pack -n torch-base -o torch-base.tar.gz
```

到另一台机器解压后即可用，比重新下载一堆包更省时间。

### **只要可复现，不一定要“整包搬运”**

 可以导出环境文件，再创建：

```bash
conda export -n torch-base --format=environment-yaml > environment.yml
conda env create -f environment.yml
```

这个方法更适合“记录依赖”，但如果目标机器没有缓存，还是会下载。

