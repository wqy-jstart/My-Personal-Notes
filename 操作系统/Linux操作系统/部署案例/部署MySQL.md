# 部署MySQL

安装：

```sh
yum install -y mariadb mariadb-server   # 安装两个组件 mariadb mariadb-server
```

服务管理：

```sh
systemctl start mariadb            # 启动服务器
systemctl stop mariadb             # 停止服务器
systemctl restart mariadb          # 重新启动服务器
systemctl enable mariadb           # 设置开机自动启动数据库
systemctl disable mariadb          # 设置开机不启动数据库
systemctl status mariadb           # 检查服务器工作状态，q退出
```

设置MySQL编码实验步骤：

登录到tom用户,提示符号`$`, 命令略

切换到管理员：

```sh
su
```

提示输入密码时候，输入密码：

```sh
Password:
```

切换到root用户后，提示符是 `#`

登录MySQL（MariaDB 服务已经启动）：

```sh
mysql
```

登录以后：

```sh
[root@iZ2zebjmyftm9yp868oy0aZ ~]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 10
Server version: 10.3.35-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

提示符号 MariaDB [(none)]> 表示已经在MySQL中。

如果没有启动MySQL会出现错误, 请启动MySQL以后再登录：

```text
ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/lib/mysql/mysql.sock' (2)
```

使用MySQL命令，显示数据库的编码：

```sh
show variables like 'char%';
```

显示：

```sh
MariaDB [(none)]> show variables like 'char%';
+--------------------------+------------------------------+
| Variable_name            | Value                        |
+--------------------------+------------------------------+
| character_set_client     | utf8                         |
| character_set_connection | utf8                         |
| character_set_database   | latin1                       |
| character_set_filesystem | binary                       |
| character_set_results    | utf8                         |
| character_set_server     | latin1                       |
| character_set_system     | utf8                         |
| character_sets_dir       | /usr/share/mariadb/charsets/ |
+--------------------------+------------------------------+
8 rows in set (0.001 sec)

MariaDB [(none)]>
```

退出MySQL 

```sql
exit
```

查看MySQL配置文件：

```sh
cd /etc
ls my*
```

结果是， 说明有配置文件 my.cnf,  配置文件夹 my.cnf.d 和 文件夹中已经存在的文件：

```text
my.cnf

my.cnf.d:
auth_gssapi.cnf           mariadb-server.cnf
client.cnf                mysql-clients.cnf
enable_encryption.preset
```

查看MySQL配置文件内容(cat命令用于显示文件的内容)：

```sh
cat my.cnf
```

文件内容：

```text
#
# This group is read both both by the client and the server
# use it for options that affect everything
#
[client-server]

#
# include all files from the config directory
#
!includedir /etc/my.cnf.d
```

include all files from the config directory 这句话的意思：my.cnf 会自动包含配置文件夹中全部的文件，只需要再配置文件夹添加文件，就可以对mysql进行配置。

