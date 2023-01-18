# MongoDB非关数据库

目标：在Linux中部署一个单机的MongoDB，作为生产环境下使用。

提示：和Windows下操作差不多。

步骤如下：

##### （1）先到官网下载压缩包`mongod-linux-x86_64-4.0.10.tgz`.

##### （2）上传压缩包到Linux中，解压到当前目录：

> **上传的方式有多种**：
>
> （1）`rz`连接本地直接找
>
> （2）Linux操作下：`scp 压缩包 主机名@服务IP:.`
>
> （3）本地cmd连接：`sftp root@ip:.`  然后put文件

```sh
tar -xvf mongodb-linux-x86_64-4.0.10.tgz
```

##### （3）移动解压后的文件夹到指定的目录中：

```sh
mv mongodb-linux-x86_64-4.0.10 /usr/local/mongodb
```

##### （4）新建几个目录，分别用来存储数据和日志：

```sh
#数据存储目录
mkdir -p /mongodb/single/data/db
#日志存储目录
mkdir -p /mongodb/single/log
```

##### （5）新建并修改配置文件:

```sh
vim /mongodb/single/mongod.conf
```

配置文件的内容如下：

```sh
systemLog:
#MongoDB发送所有日志输出的目标指定为文件
# #The path of the log file to which mongod or mongos should send all diagnostic logging information
destination: file
#mongod或mongos应向其发送所有诊断日志记录信息的日志文件的路径
path: "/mongodb/single/log/mongod.log"
#当mongos或mongod实例重新启动时，mongos或mongod会将新条目附加到现有日志文件的末尾。
logAppend: true
storage:
#mongod实例存储其数据的目录。storage.dbPath设置仅适用于mongod。
##The directory where the mongod instance stores its data.Default Value is "/data/db".
dbPath: "/mongodb/single/data/db"
journal:
#启用或禁用持久性日志以确保数据文件保持有效和可恢复。
enabled: true
processManagement:
#启用在后台运行mongos或mongod进程的守护进程模式。
fork: true
net:
#服务实例绑定的IP，默认是localhost
bindIp: localhost,192.168.0.2
#bindIp
#绑定的端口，默认是27017
port: 27017
```

##### （6）启动MongoDB服务

```sh
[root@bobohost single]# /usr/local/mongodb/bin/mongod -f /mongodb/single/mongod.conf
about to fork child process, waiting until server is ready for connections.
forked process: 90384
child process started successfully, parent exiting
```

**注意**：

如果启动后不是 `successfully` ，则是启动失败了。原因基本上就是配置文件有问题。

通过进程来查看服务是否启动了：

```sh
[root@bobohost single]# ps -ef |grep mongod
root 90384 1 0 8月26 ? 00:02:13 /usr/local/mongdb/bin/mongod -f /mongodb/single/mongod.conf
```

##### （7）分别使用mongo命令和compass工具来连接测试。

提示：如果远程连接不上，需要配置防火墙放行，或直接关闭linux防火墙

```sh
#查看防火墙状态
systemctl status firewalld
#临时关闭防火墙
systemctl stop firewalld
#开机禁止启动防火墙
systemctl disable firewalld
```

##### （8）停止关闭服务:

停止服务的方式有两种：**快速关闭**和**标准关闭**，下面依次说明：

**（一）快速关闭方法（快速，简单，数据可能会出错）**

目标：通过系统的kill命令直接杀死进程：

杀完要检查一下，避免有的没有杀掉。

```sh
#通过进程编号关闭节点
kill -2 54410
```

**（二）标准的关闭方法（数据不容易出错，但麻烦）**

目标：通过mongo客户端中的shutdownServer命令来关闭服务

主要的操作步骤参考如下：

```sh
//客户端登录服务，注意，这里通过localhost登录，如果需要远程登录，必须先登录认证才行。
mongo --port 27017
//#切换到admin库
use admin
//关闭服务
db.shutdownServer()
```

##### 【补充】:

如果一旦是因为数据损坏，则需要进行如下操作（了解）：

**1）删除lock文件**：

```sh
rm -f /mongodb/single/data/db/*.lock
```

**2）修复数据**：

```sh
/usr/local/mongdb/bin/mongod --repair --dbpath=/mongodb/single/data/db
```

