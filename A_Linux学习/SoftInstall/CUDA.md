## 确认显卡支持的CUDA最高版本

打开终端执行命令

```bash
nvidia-smi
```

重点关注`CUDA Version`字段，示例显示为`12.6`，这说明当前显卡最高支持`CUDA 12.6`版本（重要！**后续下载必须等于或低于此版本**）

## 下载适配的CUDA版本

## 官网入口

访问 [NVIDIA CUDA Toolkit官网](https://developer.nvidia.com/cuda-toolkit)

## 历史版本获取（关键步骤！）

- 若官网默认展示的版本高于显卡支持版本（如示例中的`12.6`），需切换至历史版本页面。点击页面底部的`Archive of Previous CUDA Releases`链接

版本选择   

- 找到与显卡匹配的最新版本（如`12.6`）
- 选择对应操作系统和安装类型

![image-20251124185225820](/home/wenjing/.config/Typora/typora-user-images/image-20251124185225820.png)

根据官方提示在终端执行如下命令

```bash
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-ubuntu2404.pin

sudo mv cuda-ubuntu2404.pin /etc/apt/preferences.d/cuda-repository-pin-600

wget https://developer.download.nvidia.com/compute/cuda/13.0.0/local_installers/cuda-repo-ubuntu2404-13-0-local_13.0.0-580.65.06-1_amd64.deb

sudo dpkg -i cuda-repo-ubuntu2404-13-0-local_13.0.0-580.65.06-1_amd64.debsudo cp /var/cuda-repo-ubuntu2404-13-0-local/cuda-*-keyring.gpg /usr/share/keyrings/

sudo apt-get updatesudo 

apt-get -y install cuda-toolkit-13-0
```

## 执行过程中出现错误

`bzip2` 需要特定版本的 `libbz2-1.0`，但系统准备安装的版本不匹配。让我们一步步解决这个依赖冲突：

### 解决方案

#### 第一步：清理和更新软件源

```bash
# 清理缓存
sudo apt clean
sudo apt autoclean

# 更新软件包列表
sudo apt update

# 查看可用的libbz2-1.0版本
apt-cache policy libbz2-1.0
```

#### 第二步：尝试手动安装正确版本

```bash
# 尝试安装特定版本的libbz2-1.0
sudo apt install libbz2-1.0=1.0.8-5.1

# 如果成功，再安装bzip2
sudo apt install bzip2
```

## CUDA设置到环境变量中

### 第一步：检查CUDA是否真的安装了

```bash
# 检查CUDA相关包是否安装
dpkg -l | grep cuda

# 检查CUDA目录是否存在
ls /usr/local/cuda*
ls /usr/local/cuda-13.0/bin/nvcc 2>/dev/null
```

### 第二步：手动查找nvcc

```bash
# 在整个系统中搜索nvcc
sudo find / -name nvcc 2>/dev/null

# 检查常见的CUDA安装路径
ls /usr/bin/nvcc 2>/dev/null
ls /usr/local/cuda/bin/nvcc 2>/dev/null
ls /opt/cuda/bin/nvcc 2>/dev/null
```

### 第五步：设置环境变量

安装完成后，手动设置环境变量：

```bash
# 编辑bashrc文件
nano ~/.bashrc

# 在文件末尾添加以下内容：
export PATH=/usr/local/cuda-13.0/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-13.0/lib64:$LD_LIBRARY_PATH
export CUDA_HOME=/usr/local/cuda-13.0

# 保存并退出，然后使配置生效
source ~/.bashrc
```

### 第六步：验证安装

```bash
# 现在检查nvcc
nvcc --version

# 检查CUDA版本
nvidia-smi

# 检查CUDA目录
ls /usr/local/cuda-13.0/bin/nvcc
```

