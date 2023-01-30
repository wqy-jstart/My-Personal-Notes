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

## 维护：

#### 1.关于Linux服务器访问mysql时出现数据包过大的问题：

报错信息：

```sh
Packet for query is too large (1,068 > 1,024). You can change this value on the server by setting the 'max_allowed_packet' variable.
```

解决办法：

向mysql数据库发送查询命令时，默认性况下，一个包的大小是1024，实际一条语句的长度，就大小这个，所以执行语句报错。解决的方式，就是调整mysql中该参数的大小。

##### 1.查询默认查询数据包大小：

```sh
show variables like '%max_allowed_packet%';
```

```sh
MariaDB [(none)]> show variables like '%max_allowed_packet%';
+--------------------------+------------+
| Variable_name            | Value      |
+--------------------------+------------+
| max_allowed_packet       | 1024       |
| slave_max_allowed_packet | 1073741824 |
+--------------------------+------------+
2 rows in set (0.00 sec)
```

##### 2.修改查询数据包大小：

```sh
set global max_allowed_packet = 100*1024;
```

##### 3.重新启动数据库服务：

```sh
systemctl restart mariadb    # 重新启动服务
systemctl stop mariadb       # 停止服务 
```
