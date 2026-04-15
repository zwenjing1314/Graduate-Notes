# 初次安装

检查是否安装mysql

```bash
dpkg -l mysql-server mysql-client mysql-client-core-8.0 mariadb-server mariadb-client-core 2>/dev/null
```

安装mysql

```bash
  sudo apt update
  sudo apt install mysql-server mysql-client
```

如果提示找不到 mysql-client，就用：

```bash
sudo apt install mysql-server mysql-client-core-8.0
```

## 创建一个数据库

安装完成后，启动 MySQL：

```bash
sudo systemctl enable --now mysql
```

然后进入 MySQL：

```bash
sudo mysql
```

注意这里不是 mysql -u root -p，Ubuntu 默认经常是用 sudo mysql 进入 root 管理模式。

进入后创建数据库：

```mysql
CREATE DATABASE FastAPI_first;
SHOW DATABASES;
```

如果你看到 FastAPI_first，说明创建成功。

 如果你之后想用 root 和密码 123456 登录，可以在 sudo mysql 里面再执行：

```mysql
ALTER USER 'root'@'localhost' IDENTIFIED WITH caching_sha2_password BY '123456';
FLUSH PRIVILEGES;
```

退出 MySQL：

以后就可以这样登录：

mysql -u root -p

  然后输入密码：

  123456

不过做 FastAPI 项目时，更推荐单独创建一个数据库用户，而不是直接用 root。学习阶段先这样也可以。

开机自启

```pysql
systemctl enable --now mysql
```

 - enable：设置开机自启
  - --now：立刻启动服务

检查 MySQL 是不是真的已经启动成功。在终端输入：

```bash
sudo systemctl status mysql --no-pager
```

如果你看到类似下面这行：

  Active: active (running)

  那就说明已经启动好了。接着直接进入 MySQL

## pycharm 连接 mysql

sudo mysql 通常不是“启动数据库”，而是“以系统 root 身份进入 MySQL 客户端”。
PyCharm 不能直接复用这个 sudo 免密登录方式，所以你一般需要先创建一个带密码的 MySQL 用户，再让 PyCharm 用这个用户连 127.0.0.1:3306。

 怎么让 PyCharm 连上

 先在终端里创建一个正常的账号和密码。最稳妥的是单独建开发用户，不直接拿 root 给 PyCharm 用。

```mysql
sudo mysql
CREATE USER 'jing'@'localhost' IDENTIFIED BY '123456';
GRANT ALL PRIVILEGES ON FastAPI.* TO 'jing'@'localhost';
FLUSH PRIVILEGES;
```

如果你准备在 PyCharm 里填 127.0.0.1，再补一个：

```mysql
CREATE USER 'jing'@'127.0.0.1' IDENTIFIED BY '123456';
GRANT ALL PRIVILEGES ON FastAPI.* TO 'jing'@'127.0.0.1';
FLUSH PRIVILEGES;
```

一开始是在终端通过 sudo mysql 进入root权限的MySql客户端，并创建了一个FastAPI_first数据库。但是在上一步

```mysql
问题点已经很明确：你刚才授权的是 FastAPI.*，不是 FastAPI_first.*。而且你在 PyCharm 里执行 CREATE DATABASE FastAPI_first; 又因为 jing 没有全局建库权限失
  败了，所以 FastAPI_first 很可能根本还没有被创建成功。

• 你这次授权的对象是：

GRANT ALL PRIVILEGES ON FastAPI.* TO 'jing'@'127.0.0.1';

  这只授权 jing 访问 FastAPI 这个数据库，不包括 FastAPI_first。而你在 PyCharm 里创建 FastAPI_first 又失败了，所以它很可能根本不存在。

  用 sudo mysql 重新执行这一组：

SHOW DATABASES LIKE 'FastAPI%';

CREATE DATABASE IF NOT EXISTS `FastAPI_first`
CHARACTER SET utf8mb4
COLLATE utf8mb4_unicode_ci;

GRANT ALL PRIVILEGES ON `FastAPI_first`.* TO 'jing'@'127.0.0.1';
GRANT ALL PRIVILEGES ON `FastAPI_first`.* TO 'jing'@'localhost';

FLUSH PRIVILEGES;

SHOW GRANTS FOR 'jing'@'127.0.0.1';

  如果你真正想用的是之前的 FastAPI 数据库，而不是 FastAPI_first，那就不要新建 FastAPI_first，在 PyCharm 里选 FastAPI 即可。FastAPI 和 FastAPI_first 是两
  个完全不同的数据库名。
```



# 对数据库的理解

 所以 MySQL 不是 conda 那种“进入某个环境，再在环境里装东西”的结构。它更像这样：

  1. 一个 MySQL 实例
  2. 实例下面有多个数据库（database，在 MySQL 里基本等同于 schema）
  3. 每个数据库下面有表、视图、存储过程等对象
  4. 用户账号和权限是实例级的，不属于某个数据库目

 可以把它理解成：

  - MySQL 实例 = 一栋楼
  - 数据库 = 楼里的房间
  - 表 = 房间里的柜子
  - root = 拿着总钥匙的人

## MySQL 的数据库和表怎么存储？

以现在常见的 MySQL 8.0 + InnoDB 为例：

  - 整个实例有一个数据目录 datadir
  - 每个数据库通常对应 datadir 下的一个目录
  - 这个数据库里的每张表，通常对应一个 .ibd 文件
  - 还有一些全局文件，比如 redo log、undo log、binlog、系统表空间等

 大致像这样：

```
  datadir/
    mysql/          # 系统库
    base/           # 你的一个数据库
      user.ibd
      order.ibd
    ibdata1         # 共享表空间（某些内容）
    undo_001
    undo_002
    #ib_redo...
    binlog.000001
```

不过要注意几点：

  - “数据库=一个文件夹”这个理解只适合粗略理解
  - MySQL 8.0 的很多元数据已经放进 InnoDB 数据字典里了，不再是早期那种很多独立元数据文件
  - information_schema 这类库很多是逻辑视图，不完全等于普通目录里的业务数据

## 它的设计理念是什么？

  核心理念其实是这几个：

  - 它是“关系型数据库管理系统”，不是对象仓库
  - 数据按“表”组织，表按“数据库/schema”分组
  - database/schema 主要是命名空间和权限边界，不是独立运行环境
  - 用户、权限、事务、锁、日志这些由整个实例统一管理
  - 存储细节交给存储引擎处理，最常见的是 InnoDB