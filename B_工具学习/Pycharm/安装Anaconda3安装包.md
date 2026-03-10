1. 安装Anaconda3安装包
地址  https://www.anaconda.com/download
在下载文件目录下打开终端terminal，使用下面的bash执行安装命令
bash Anaconda3-2023.09-0-Linux-x86_64.sh 
过程全部yes

如果希望 conda 的基础环境在启动时不被激活，请将 auto_activate_base 参数设置为 false，命令如下：
conda config --set auto_activate_base false

当然这一条命令执行完毕后，想要再次进入conda的base环境，只需要使用对应的conda指令即可，如下：
conda activate base

创建新环境
conda create -n your_env_name python=x.x

删除已有的虚拟环境
conda remove -n your_env_name --all

启动Anaconda Navigator
    激活Anaconda环境
    执行命令：
    source ~/anaconda3/bin/activate root
启动图形化界面
    运行： 
    anaconda-navigator





查看Ubuntu是x86还是arm

在 Ubuntu 中，你可以使用以下命令来查看你的系统架构：
uname -m

另外，你也可以使用以下命令来获取更详细的系统信息：
lscpu

安装软件
cd /path/to/your/deb/file
sudo dpkg -i yourpackage.deb

bash Anaconda3-2023.09-0-Linux-x86_64.sh 

如果安装过程中提示缺少依赖，可以使用apt命令来自动解决这些依赖：
sudo apt-get install -f

使用 which 命令查找可执行文件的位置：
which qq
which anaconda
whereis <文件名>
whereis qq
whereis anaconda



在终端打开图形化工具的方法

首先在终端进入相应的文件夹下
cd /path/to/your/folder
nautilus .

cd /path/to/your/folder
xdg-open .