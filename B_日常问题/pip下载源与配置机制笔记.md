# pip 下载源与配置机制笔记

## 这次改动落在哪个文件

这台机器上，`pip` 当前生效的用户级配置文件是：

- `~/.config/pip/pip.conf`

对应到你的实际路径就是：

- `/home/wenjing/.config/pip/pip.conf`

当前内容是：

```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

这表示：`pip` 默认不再从官方 `PyPI` 主站拉包，而是优先去清华镜像站取包索引。

## `pip` 是怎么决定“去哪个源下载”的

`pip install fastapi` 这类命令执行时，大致会做这几步：

1. 启动时先读取配置。
2. 把命令行参数、环境变量、配置文件里的设置合并。
3. 得到最终有效的 `index-url`。
4. 到这个源的 `simple` 索引接口查包。
5. 找到符合当前 Python 版本、平台、架构的 wheel 或源码包。
6. 再根据链接去下载对应文件并安装。

其中最关键的是这项配置：

- `index-url`

它定义“默认主下载源”。

比如：

```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

当你执行：

```bash
pip install requests
```

`pip` 实际会先访问类似这样的地址：

```text
https://pypi.tuna.tsinghua.edu.cn/simple/requests/
```

然后从页面或接口里挑选可用安装包。

## 为什么这次改动会生效

因为 `pip` 每次运行时，都会按固定顺序查找配置来源。只要它在某个配置文件里读到了 `index-url`，后续下载就会按这个值走。

你这台机器上，`pip config debug` 显示它会检查这些位置：

### 1. 全局配置

- `/etc/xdg/xdg-ubuntu/pip/pip.conf`
- `/etc/xdg/pip/pip.conf`
- `/etc/pip.conf`

这类配置通常影响整台机器上的所有用户。

### 2. 用户级配置

- `~/.pip/pip.conf`
- `~/.config/pip/pip.conf`

这类配置只影响当前用户，最常用，也最安全。

### 3. 当前环境级配置

- `/home/wenjing/anaconda3/envs/artisanal/pip.conf`

这类配置只影响某个虚拟环境或某个解释器环境。

这次我执行的是：

```bash
python -m pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
```

实际写入结果是：

- `/home/wenjing/.config/pip/pip.conf`

所以之后你无论在这个项目里，还是在别的项目里，只要还是当前这个用户在运行 `pip`，都会默认使用这个镜像。

## 为什么 `pip config set` 默认写到 `~/.config/pip/pip.conf`

这个行为不是随机的，而是 `pip config` 命令自己的默认规则。

`pip config set`、`pip config unset`、`pip config edit` 这类“修改配置”的命令，和普通 `pip install` 的“读取配置”稍微不同。它们不是把所有配置文件都改一遍，而是只会选择一个目标文件来写。

`pip` 内置帮助给出的规则是：

- 如果没有显式传 `--user`、`--global`、`--site`
- 并且当前处于虚拟环境中
- 且这个虚拟环境对应的配置文件已经存在
- 那么优先改这个虚拟环境的配置文件
- 否则，默认改用户级配置文件

也就是说，默认写入目标的选择逻辑可以简化成：

1. 你手动指定了 `--global`，就写全局配置。
2. 你手动指定了 `--user`，就写用户级配置。
3. 你手动指定了 `--site`，就写当前环境配置。
4. 你什么都没指定时：
   - 如果当前虚拟环境配置文件存在，就写环境级配置。
   - 如果不存在，就回退到用户级配置。

## 为什么你这次正好写到了 `~/.config/pip/pip.conf`

你这次的实际环境是：

- 当前 Python 解释器来自 conda 环境：`/home/wenjing/anaconda3/envs/artisanal/bin/python`
- `pip` 检查过环境级配置文件：`/home/wenjing/anaconda3/envs/artisanal/pip.conf`
- 这个文件不存在
- 你执行命令时没有传 `--site`
- 也没有传 `--user`
- 也没有传 `--global`

所以它按默认规则回退到了用户级配置文件。

而在 Linux 上，用户级配置的当前标准位置就是：

- `$HOME/.config/pip/pip.conf`

你的 `HOME` 是 `/home/wenjing`，所以最终写入路径就是：

- `/home/wenjing/.config/pip/pip.conf`

## 这里为什么不是写到 `~/.pip/pip.conf`

因为 `~/.pip/pip.conf` 是旧版兼容路径，`~/.config/pip/pip.conf` 才是当前标准位置。

在读取配置时，`pip` 两个位置都会看：

- 先看旧位置 `~/.pip/pip.conf`
- 再看当前标准位置 `~/.config/pip/pip.conf`

如果两个文件都存在，标准位置里的值通常会覆盖旧位置。

而在“写配置”这件事上，`pip config set` 默认会写到当前标准用户配置文件，而不是优先写旧兼容路径。

## 用户级配置文件路径是“写死”的吗

不是“把 `/home/wenjing/.config/pip/pip.conf` 这个完整字符串写死在程序里”。

更准确地说，`pip` 按操作系统约定和环境变量，推导出一组“标准查找位置”。

在 Linux / Unix 下，用户级配置的当前标准位置是：

- `$HOME/.config/pip/pip.conf`

如果设置了 `XDG_CONFIG_HOME`，那么它会变成：

- `$XDG_CONFIG_HOME/pip/pip.conf`

所以你现在看到的：

- `/home/wenjing/.config/pip/pip.conf`

本质上只是因为：

- 你的 `HOME` 是 `/home/wenjing`
- 当前使用的是默认的 XDG 配置目录

也就是说，目录规则是固定的，但路径字符串不是死写成只能是 `/home/wenjing/...`。

## `pip` 只能从几个指定位置读配置吗

默认情况下，基本是的。

`pip` 不会任意扫描整个磁盘去找名为 `pip.conf` 的文件，而是只会读取它认可的那几类位置：

1. 全局配置位置
2. 用户级配置位置
3. 当前虚拟环境的位置
4. `PIP_CONFIG_FILE` 明确指定的位置

这也是为什么你随便在别的目录放一个 `pip.conf`，`pip` 并不会自动读取。

## 哪些路径是“默认可读取”的

以 Linux / Unix 为例，官方定义的默认位置是：

### 全局配置

- `XDG_CONFIG_DIRS` 中每个目录下的 `pip/pip.conf`
- `/etc/pip.conf`

例如：

- `/etc/xdg/pip/pip.conf`
- `/etc/pip.conf`

### 用户级配置

- `$HOME/.config/pip/pip.conf`
- 兼容旧写法的 `$HOME/.pip/pip.conf`

如果两个都存在，当前标准位置通常覆盖旧位置。

### 环境级配置

- `$VIRTUAL_ENV/pip.conf`

也就是当前虚拟环境自己的配置文件。

## 有没有办法让 `pip` 读取一个任意路径的配置文件

有，用环境变量：

```bash
export PIP_CONFIG_FILE=/path/to/your/pip.conf
```

这个文件会在最后加载，所以它的优先级最高，会覆盖前面那些默认位置里的同名配置。

这等于告诉 `pip`：

- “除了默认位置之外，再额外读取这个指定文件”

在某些情况下，它甚至会影响是否加载用户配置文件。

## 读取配置和写入配置，不是同一套问题

这一点很容易混淆，最好分开记。

### 读取配置时关注的是“会从哪些地方加载”

也就是：

- 全局配置
- 用户级配置
- 环境级配置
- `PIP_CONFIG_FILE` 指定配置

这件事决定的是：

- “最终哪些值生效”

### 写入配置时关注的是“这次修改要落到哪个单一文件”

也就是 `pip config set` 在决定：

- “把新值保存到哪一个文件”

这件事决定的是：

- “你执行完命令后，哪个文件被改了”

所以不要把下面两件事混为一谈：

- `pip install` 运行时会读取多个配置来源
- `pip config set` 修改时只会选择一个目标文件写入

## 所以结论是什么

可以把它记成一句话：

- `pip` 默认只认一小组官方规定的位置，但这些位置里的部分路径会根据 `HOME`、`XDG_CONFIG_HOME`、`VIRTUAL_ENV`、`PIP_CONFIG_FILE` 这样的环境变量动态变化。 

## 配置的覆盖顺序

同一个配置项如果在多个地方都定义了，后面的高优先级会覆盖前面的低优先级。可以把它简单理解成：

1. 命令行参数最高优先级
2. 环境变量其次
3. 配置文件最后

常见例子：

### 命令行临时覆盖

```bash
pip install fastapi -i https://mirrors.aliyun.com/pypi/simple/
```

这次安装只对这一条命令生效，不会改配置文件。

### 环境变量覆盖

```bash
export PIP_INDEX_URL=https://mirrors.aliyun.com/pypi/simple/
```

只要这个 shell 会话还在，`pip` 会优先用这个值。

### 配置文件长期生效

```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

这是持久化配置。每次新开终端执行 `pip`，它都会读取。

## “下载源”本质上是什么

所谓“源”，本质上是一个 Python 包索引服务。

官方默认源是：

```text
https://pypi.org/simple
```

清华、阿里这些镜像站做的事情是：

- 定期同步官方包索引
- 在国内提供更近的访问节点
- 减少跨境网络带来的超时、丢包、握手失败

所以改源的核心收益不是“改变安装逻辑”，而是“让相同的安装逻辑去访问更稳定、更近的服务器”。

## 为什么有时候改了源还是会报“网络不可达”

因为“换源”只能解决一部分问题，它解决的是：

- 访问官方源慢
- DNS 解析慢
- TLS 握手不稳定
- 跨境链路质量差

但它解决不了下面这些问题：

- 本机完全断网
- DNS 本身坏了
- 代理配置错误
- 公司或校园网限制外连
- conda / pip 使用了不同网络配置
- IPv6 路由异常

所以如果报错是“`Network is unreachable`”，它有时并不是源地址的问题，而是更底层的网络栈没通。

## 为什么推荐改“用户级配置”而不是全局配置

优点有三个：

1. 不需要管理员权限。
2. 只影响你自己，不会改坏系统其他用户的环境。
3. 后续容易回滚，删除或修改一个文件就行。

## 常见下载源写法

### 清华源

```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

### 阿里云源

```ini
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/
```

## 如何切换

### 切到阿里云

把文件改成：

```ini
[global]
index-url = https://mirrors.aliyun.com/pypi/simple/
```

### 切回清华

把文件改成：

```ini
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
```

## 如何检查当前到底生效了哪个配置

### 查看配置来源

```bash
python -m pip config debug
```

它会告诉你：

- 会查哪些配置文件
- 哪些文件存在，哪些不存在

### 查看当前有效配置

```bash
python -m pip config list -v
```

它会告诉你最终读到了哪些配置项，比如：

```text
global.index-url='https://pypi.tuna.tsinghua.edu.cn/simple'
```

## 适合记住的一句话

`pip` 不是“下载时临时猜一个源”，而是“启动时先读取本地配置，再按最终生效的 `index-url` 去索引站拉包”。

所以你要改下载源，本质上就是在改 `pip` 的配置来源，而不是改项目代码。
