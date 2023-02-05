# Windows命令

##### 1.查看电脑地址状态

```sh
netstat -an
```

查看TCP端口占用情况

```sh
netstat -ano
```

##### 2.查看本机信息

```sh
systeminfo
```

##### 3.查看本机应用服务状态

```sh
services.msc
```

##### 4.打开资源管理器

```sh
explorer
```

##### 5.注销命令

```sh
logoff
```

##### 6.60秒倒计时关机命令

```sh
shutdown
```

##### 7.查看·本机用户和组

```sh
lusrmgr.msc
```

##### 8.打开记事本

```sh
notepad
```

##### 9.垃圾整理

```sh
cleanmgr
```

##### 10.开始信使服务

```sh
net start messenger # 开启
net start messenger # 停止
```

##### 11.计算机管理

```sh
compmgmt.msc
```

##### 12.DVD播放器

```sh
dvdplay
```

##### 13.启动字符映射表

```sh
charmap
```

##### 14.磁盘管理实用程序

```sh
diskmgmt.msc
```

##### 15.启动计算器

```sh
calc
```

##### 16.磁盘碎片整理程序

```sh
dfrg.msc
```

##### 17.设备管理器

```sh
devmgmt.msc
```

##### 18.系统医生

```sh
drwtsn32
```

##### 19.15秒关机

```sh
rononce -p
```

##### 20.注册表编辑器

```sh
regedt32
```

##### 21.系统配置实用程序

```sh
Msconfig.exe
```

##### 22.显示内存使用情况

```sh
mem.exe
```

##### 23.程序管理器

```sh
progman
```

##### 24.系统信息

```sh
winmsd
```

##### 25.计算机性能监测程序

```sh
perfmon.msc
```

##### 26.检查Windows版本

```sh
winver
```

##### 27.任务管理器

```sh
taskmgr
```

##### 28.打开windows管理体系结构

```sh
wmimgmt.msc
```

##### 29.windows更新程序

```sh
wupdmgr
```

##### 30.写字板

```sh
write
```

##### 31.扫描仪和照相机向导

```sh
wiaacmgr
```

##### 32.图画板

```sh
mspaint
```

##### 33.远程桌面连接

```sh
mstsc
```

##### 34.打开控制台

```sh
mmc
```

##### 35.同步命令

```sh
mobsync
```

##### 36.木马捆绑工具，系统自带

```sh
iexpress
```

##### 37.共享文件夹管理器

```sh
fsmgmt.msc
```

##### 38.辅助工具管理器

```sh
utilman
```

##### 39.打开系统组件服务

```sh
dcomcnfg
```

##### 40.打开屏幕键盘

```sh
osk
```

##### 41.清理命令行内容

```sh
cls
```

##### 42.切换目录

1. 切换盘`D:`：

   ```sh
   C:\Users\21681>D:
   
   D:\>
   ```

2. 切换当前盘的目录`cd`：

   ```sh
   D:\>cd devtools
   
   D:\devtools>
   ```

3. 查看当前目录的文件`dir`：

   ```sh
   D:\devtools\java_install>dir
    驱动器 D 中的卷是 Data
    卷的序列号是 245B-DA8D
   
    D:\devtools\java_install 的目录
   
   2023/01/12  15:25    <DIR>          .
   2023/01/12  15:25    <DIR>          ..
   2023/01/12  15:25    <DIR>          apache-tomcat-8.5.84
   2023/01/12  15:24        11,860,862 apache-tomcat-8.5.84-windows-x64.zip
                  1 个文件     11,860,862 字节
                  3 个目录 224,151,785,472 可用字节
   
   D:\devtools\java_install>
   ```

##### 43.连接远程服务器

1.`sftp`

```sh
sftp root@ip地址
```

连接远程后打印命令帮助：

```sh
bye                                Quit sftp # 退出sftp
chmod [-h] mode path               Change permissions of file 'path' to 'mode' # # 将文件“path”的权限更改为“mode”
chown [-h] own path                Change owner of file 'path' to 'own' # 将文件“path”的所有者更改为“own”
df [-hi] [path]                    Display statistics for current directory or
                                   filesystem containing 'path' # 显示当前目录或包含“路径”的文件系统
exit                               Quit sftp # 退出sftp
get [-afpR] remote [local]         Download file # 下载文件
help                               Display this help text # 查看帮助
lls [ls-options [path]]            Display local directory listing # 显示本地目录列表
lmkdir path                        Create local directory # 创建本地目录
ln [-s] oldpath newpath            Link remote file (-s for symlink) # 链接远程文件（-s表示符号链接）
lpwd                               Print local working directory # 打印本地工作目录
ls [-1afhlnrSt] [path]             Display remote directory listing # 显示远程目录列表
mkdir path                         Create remote directory # 创建远程目录
progress                           Toggle display of progress meter # 进度表的显示
put [-afpR] local [remote]         Upload file # 上载文件
pwd                                Display remote working directory # 显示远程工作目录
reget [-fpR] remote [local]        Resume download file # 恢复下载文件
rename oldpath newpath             Rename remote file # 重命名远程文件
reput [-fpR] local [remote]        Resume upload file # 恢复上载文件
rm path                            Delete remote file # 删除远程文件
rmdir path                         Remove remote directory # 删除远程目录
version                            Show SFTP version # 显示SFTP版本
!command                           Execute 'command' in local shell # 在本地shell中执行“command”
!                                  Escape to local shell # 逃到本地外壳
?                                  Synonym for help # 帮助
```

