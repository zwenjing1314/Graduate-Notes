# Ubuntu 新手笔记：把源码编译成 `.deb` 安装包

本文记录的是我在 **Ubuntu 24.04.3 LTS** 上，把 `CuteTranslation` 这份源码打包成 `.deb` 安装包的完整过程。以后遇到类似的 Qt / Debian 打包项目，可以按这份笔记自己排查。

## 1. 先判断源码能不能直接打包成 `.deb`

进入源码目录后，先看项目里有没有 `debian/` 目录：

```bash
cd ~/下载/CuteTranslation-master
find . -maxdepth 2 -type f | sort
```

这次项目里已经自带这些文件：

- `debian/control`
- `debian/rules`
- `debian/changelog`
- `debian/com.github.cute-translation.desktop`

这说明：

- 这不是从零开始打包
- 作者已经写过 Debian 打包规则
- 一般可以直接尝试 `dpkg-buildpackage`

如果一个项目 **没有** `debian/` 目录，就不能直接这样打包，需要另外补打包元数据。

## 2. 先检查当前系统版本

不同 Ubuntu / Debian 版本里，Qt 包名可能不一样。先确认系统版本：

```bash
sed -n '1,20p' /etc/os-release
```

我这次的系统是：

- `Ubuntu 24.04.3 LTS`
- 代号：`noble`

这个信息很重要，因为有些老项目里写的依赖包名，在新系统里已经废弃了。

## 3. 查看 Debian 打包依赖

先看 `debian/control`：

```bash
sed -n '1,120p' debian/control
```

原来这里面有一个比较老的依赖：

- `qt5-default`

在 Ubuntu 24.04 中，这个包已经不可用了，所以需要改成新的 Qt5 开发包组合。

我最后整理后的关键依赖是：

- `debhelper`
- `devscripts`
- `qtbase5-dev`
- `qt5-qmake`
- `qtbase5-dev-tools`
- `qtdeclarative5-dev`
- `libxtst-dev`
- `libxcb-util0-dev`
- `libqt5x11extras5-dev`
- `qtmultimedia5-dev`

## 4. 先检查缺了哪些构建依赖

最有用的命令之一：

```bash
dpkg-checkbuilddeps
```

它会直接告诉你当前源码包缺哪些依赖。

这次它提示缺少：

```text
debhelper (>= 11)
libxtst-dev
libxcb-util0-dev
qtbase5-dev
qt5-qmake
qtbase5-dev-tools
libqt5x11extras5-dev
qtmultimedia5-dev
qtdeclarative5-dev
```

如果你以后打包别的项目，优先跑这个命令，能省很多时间。

## 5. 安装构建依赖

这次最终安装命令可以写成：

```bash
sudo apt-get install -y \
  debhelper devscripts \
  qtbase5-dev qt5-qmake qtbase5-dev-tools qtdeclarative5-dev \
  libxtst-dev libxcb-util0-dev libqt5x11extras5-dev qtmultimedia5-dev
```

如果项目运行时还依赖额外工具，也可以顺手装上。这个项目里还用到了：

- `gnome-screenshot`
- `tidy`

可以一起装：

```bash
sudo apt-get install -y gnome-screenshot tidy
```

## 6. 注意 `qmake` 可能被 Anaconda 抢走

我这次机器上执行：

```bash
which qmake
qmake -v
```

结果发现优先用到的是：

```text
/home/wenjing/anaconda3/bin/qmake
```

这很危险，因为我们打包系统 `.deb` 时，应该优先使用系统 Qt，而不是 Anaconda 自带 Qt。

### 解决方法

修改 `debian/rules`，把系统 Qt5 的路径放到前面：

```makefile
export QT_SELECT := qt5
export PATH := /usr/lib/qt5/bin:$(PATH)
```

这样打包时会优先用系统的 `qmake`。

## 7. 旧项目常见问题：过时依赖或模块缺失

### 问题 1：`qt5-default` 已经过时

原项目的 `debian/control` 里用了 `qt5-default`，在 Ubuntu 24.04 下不可用。

解决：

把它改成：

- `qtbase5-dev`
- `qt5-qmake`
- `qtbase5-dev-tools`

### 问题 2：`Unknown module(s) in QT: qml`

第一次构建时出现：

```text
Project ERROR: Unknown module(s) in QT: qml
```

我一开始以为 `qml` 是多余的，后来继续查源码发现：

```cpp
#include <QJSEngine>
```

`QJSEngine` 属于 Qt QML 模块，所以 `qml` 不是多余，而是系统缺少对应开发包。

真正的解决办法是安装：

```bash
sudo apt-get install -y qtdeclarative5-dev
```

经验：

- `Unknown module(s) in QT: xxx)` 通常不是源码坏了
- 更常见的原因是缺少对应的 Qt 开发包
- 不要急着删 `.pro` 里的模块，先查源码有没有实际使用

## 8. 开始构建 `.deb`

依赖装好后，在源码目录执行：

```bash
cd ~/下载/CuteTranslation-master
dpkg-buildpackage -us -uc -b
```

参数含义：

- `-us`：不签源码包
- `-uc`：不签 `.changes`
- `-b`：只构建二进制包，不构建源码包

这对本地自己打包安装已经足够了。

## 9. 如何判断构建是否成功

看到下面这类输出，就说明成功了：

```text
dpkg-deb: 正在 '../com.github.cute-translation_0.5.0-1_amd64.deb' 中构建软件包 'com.github.cute-translation'
dpkg-buildpackage: info: binary-only upload (no source included)
```

这次生成的文件在源码目录的上一级，也就是 `~/下载/`：

- `com.github.cute-translation_0.5.0-1_amd64.deb`
- `com.github.cute-translation-dbgsym_0.5.0-1_amd64.ddeb`

其中真正要安装的是：

- `.deb`

`*.ddeb` 是调试符号包，普通用户通常不需要。

## 10. 编译过程中的 warning 要不要管？

这次构建时出现了很多：

- deprecated 警告
- unused parameter 警告
- `system()` 返回值未处理的警告

只要最后成功生成 `.deb`，这些 **warning 不会阻止打包成功**。

判断标准很简单：

- `warning` 通常可以先忽略
- `error` 才会导致构建失败

不过如果以后你要长期维护项目，建议慢慢把 warning 清掉，这样升级到新版本 Qt / GCC 时更稳。

## 11. 安装生成好的 `.deb`

推荐这样安装：

```bash
sudo apt install ~/下载/com.github.cute-translation_0.5.0-1_amd64.deb
```

如果依赖没装全，可以再执行：

```bash
sudo apt -f install
```

## 12. 为什么安装时会出现 `_apt` 权限提示

你这次看到的是：

```text
N: 由于文件'/home/wenjing/下载/com.github.cute-translation_0.5.0-1_amd64.deb'无法被用户'_apt'访问，已脱离沙盒并提权为根用户来进行下载。 - pkgAcquire::Run (13: 权限不够)
```

这不是报错，是提示。

意思是：

- `apt` 默认想用低权限用户 `_apt` 去读取本地 `.deb`
- 但 `_apt` 没权限访问你 `~/下载/` 里的文件
- 所以 `apt` 自动切换为 `root` 继续安装

### 这条提示要不要处理？

通常不用处理，安装已经成功。

如果你想避免这条提示，可以：

### 方法 1：改用 `dpkg`

```bash
sudo dpkg -i ~/下载/com.github.cute-translation_0.5.0-1_amd64.deb
sudo apt -f install
```

### 方法 2：先复制到 `/tmp`

```bash
cp ~/下载/com.github.cute-translation_0.5.0-1_amd64.deb /tmp/
chmod 644 /tmp/com.github.cute-translation_0.5.0-1_amd64.deb
sudo apt install /tmp/com.github.cute-translation_0.5.0-1_amd64.deb
```

## 13. 以后遇到类似情况，建议按这个顺序排查

### 第一步：看项目是什么类型

常见入口文件：

- `debian/`：说明有 Debian 打包规则
- `CMakeLists.txt`：CMake 项目
- `Makefile`
- `Cargo.toml`：Rust
- `package.json`：Node.js
- `pyproject.toml`：Python
- `*.pro`：Qt qmake 项目

### 第二步：先检查构建依赖

```bash
dpkg-checkbuilddeps
```

### 第三步：确认工具链是不是系统版本

比如 Qt 项目：

```bash
which qmake
qmake -v
```

如果跑到了 Conda / Anaconda / 自己手装的 Qt，要小心。

### 第四步：开始构建

```bash
dpkg-buildpackage -us -uc -b
```

### 第五步：只盯住真正的 `error`

- `warning` 先放一边
- 先解决第一个 `error`
- 解决后再重新构建
- 一次只处理一个关键错误

## 14. 这次实际修改过的文件

这次为了让项目能在 Ubuntu 24.04 上顺利打包，我改了这些内容：

### `debian/control`

调整了 `Build-Depends`，补上适合新系统的 Qt5 开发包，尤其是：

- 去掉过时的 `qt5-default`
- 增加 `qtbase5-dev`
- 增加 `qt5-qmake`
- 增加 `qtbase5-dev-tools`
- 增加 `qtdeclarative5-dev`

### `debian/rules`

增加：

```makefile
export PATH := /usr/lib/qt5/bin:$(PATH)
```

目的是强制优先使用系统 Qt5 的 `qmake`。

## 15. 这次成功构建的核心命令清单

如果以后你想快速照做，可以直接参考这一组：

```bash
cd ~/下载/CuteTranslation-master
sed -n '1,20p' /etc/os-release
sed -n '1,120p' debian/control
dpkg-checkbuilddeps
which qmake
qmake -v
sudo apt-get install -y \
  debhelper devscripts \
  qtbase5-dev qt5-qmake qtbase5-dev-tools qtdeclarative5-dev \
  libxtst-dev libxcb-util0-dev libqt5x11extras5-dev qtmultimedia5-dev \
  gnome-screenshot tidy
dpkg-buildpackage -us -uc -b
sudo apt install ~/下载/com.github.cute-translation_0.5.0-1_amd64.deb
```

## 16. 最后总结

以后把源码打成 `.deb`，你可以记住这几个关键原则：

1. 先看项目里有没有 `debian/`。
2. 先跑 `dpkg-checkbuilddeps`，不要盲猜缺什么。
3. Qt 项目要注意 `qmake` 到底来自系统还是 Conda / Anaconda。
4. 遇到 `Unknown module(s) in QT: xxx`，优先怀疑少了开发包。
5. 只要最后成功生成 `.deb`，中间的 warning 可以暂时不管。
6. `apt` 读取本地 `.deb` 时出现 `_apt` 权限提示，通常不是错误。

如果以后你想自己处理类似问题，最重要的是：

- 不要一次看全部输出
- 只抓住第一个真正导致失败的 `error`
- 解决一个，再重新编译一次

这样排查效率最高。
