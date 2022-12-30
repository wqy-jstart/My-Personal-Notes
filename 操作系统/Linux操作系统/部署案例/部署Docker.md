# 部署Docker

Docker是一种容器化技术，它的核心思想是通过对应用的**封装**、**分发**、**部署**、**运行**生命周期进行管理达到应用组件级别的“一次性封装，到处运行”。这里的应用组件，可以是一个web应用，也可以是一个环境，更可以是一个数据库等等。

部署背景：Linux-CentOS7+云服务器

### 1.更新软件源

第一个命令

```sh
yum update
```

第二个命令

```sh
yum install -y yum-utils device-mapper-persistent-data lvm2
```

### 2.添加docker稳定版本的yum软件源

```sh
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

### 3.再次更新yum源，并安装docker

```sh
yum update
```

下载docker

```sh
yum install -y docker-ce
```

### 4.安装完成后启动Docker

启动：

```sh
systemctl start docker
```

重启：

```sh
systemctl restart docker
```

停止：

```sh
systemctl stop docker
```

开机自启动：

```sh
systemctl enable docker
```

查看docker的状态：

```sh
systemctl status docker
```

