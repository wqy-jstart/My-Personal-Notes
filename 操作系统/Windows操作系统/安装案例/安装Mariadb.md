# 安装Mariadb

#### 1.下载

地址：https://mirrors.tuna.tsinghua.edu.cn/mariadb/mariadb-10.5.18/winx64-packages/mariadb-10.5.18-winx64.msi

#### 2.下载完成后即可根据步骤进行安装

#### 3.配置环境变量

打开资源管理器，右击我的电脑，进入环境变量

打开path变量，新增Mariadb的bin目录位置

例如：

```sh
C:\Program Files\MariaDB 10.5\bin
```

#### 4.打开dos窗口测试

```sh
mysql -uroot -proot
```

进入MySQL中即可！

```sh
C:\Users\21681>mysql -uroot -proot
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 8
Server version: 10.5.18-MariaDB mariadb.org binary distribution

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```