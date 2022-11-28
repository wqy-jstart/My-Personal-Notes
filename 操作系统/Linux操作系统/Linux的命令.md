# Linux系统的命令

#### 1.显示日期的命令: `date`

```shell
root@1-2:~# date
Sun Nov 27 01:33:47 PM UTC 2022
```

- 时间格式化:

  ```shell
  root@1-2:~# date +%Y/%m/%d
  2022/11/27
  ```

  ```shell
  root@1-2:~# date +%H:%M
  13:36
  ```

#### 2.显示日历的命令: `cal`

#### 3.显示计算器: `bc`

```shell

[vbird@www ~]$ bc 
bc 1.06 
Copyright 1991-1994, 1997, 1998, 2000 Free Software Foundation, Inc. 
This is free software with ABSOLUTELY NO WARRANTY. 
For details type `warranty'. 
1+2+3+4  <==只有加法时 
10 
7-8+3 
2 
10*52 
520 
10%3     <==计算『余数』 
1 
10^2 
100 
10/100   <==这个最奇怪！不是应该是 0.1 吗？ 
0 
quit     <==离开 bc 这个计算器 
#输入信息+加、-减、*乘、/除、^指数、%余数
```

因为bc默认仅输出整数，如果要输出小数点下位数，那么就必须要运行 scale=number ，那个number就是小数点位数

```shell
[vbird@www ~]$ bc 
bc 1.06 
Copyright 1991-1994, 1997, 1998, 2000 Free Software Foundation, Inc. 
This is free software with ABSOLUTELY NO WARRANTY. 
For details type `warranty'. 
scale=3     <==没错！就是这里！！ 
1/3 
.333 
340/2349 
.144 
quit 
```

#### 4.`Tab`按键

他具有『命令补全』与『文件补齐』的功能喔！ 重点是，可以避免我们打错命令或文件名呢！

```shell

[vbird@www ~]$ ca[tab][tab]    <==[tab]按键是紧接在 a 字母后面！ 
cadaver             callgrind_control   capifax             card 
cal                 cameratopam         capifaxrcvd         case 
caller              cancel              capiinfo            cat 
callgrind_annotate  cancel.cups         captoinfo           catchsegv 
# 上面的 [tab] 指的是『按下那个tab键』，不是要你输入中括号内的tab啦！
```

#### 5.`Ctrl+c`按键

如果你在Linux底下输入了错误的命令或参数，有的时候这个命令或程序会在系统底下『跑不停』这个时候怎么办？别担心， 如果你想让当前的程序『停掉』的话，可以输入：[Ctrl]与c按键(先按着[Ctrl]不放，且再按下c按键，是组合按键)， 那就是中断目前程序的按键啦！

#### 6.`Ctrl+d`按键

[Ctrl]与d按键的组合！这个组合按键通常代表着： 『键盘输入结束(End Of File, EOF 或 End Of Input)』的意思！ 另外，他也可以用来取代exit的输入呢！例如你想要直接离开文字接口，可以直接按下[Ctrl]-d就能够直接离开了(相当于输入exit啊！)。

```shell
root@1-2:~# 
logout
```

### 7.`who`、`netstat -a`、`ps-aux`命令

`who`：查看目前有谁在线?

`netstat -a` ：查看网络联机状态

`ps -aux`查看背景运行的程序

### 8.`sync`、`shutdown`、`reboot`

将数据同步写入硬盘中的命令:`sync`

惯用的关机命令: `shutdown`

重新启动,关机: `reboot`、`halt`、`poweroff`

>由于Linux系统的关机/重新启动是很重大的系统运行，因此只有root才能够进行例如shutdown, reboot等命令。

### 9.`ls`命令

```shell
[root@www ~]# ls -al
total 156
drwxr-x---   4    root   root     4096   Sep  8 14:06 .
drwxr-xr-x  23    root   root     4096   Sep  8 14:21 ..
-rw-------   1    root   root     1474   Sep  4 18:27 anaconda-ks.cfg
-rw-------   1    root   root      199   Sep  8 17:14 .bash_history
-rw-r--r--   1    root   root       24   Jan  6  2007 .bash_logout
-rw-r--r--   1    root   root      191   Jan  6  2007 .bash_profile
-rw-r--r--   1    root   root      176   Jan  6  2007 .bashrc
-rw-r--r--   1    root   root      100   Jan  6  2007 .cshrc
drwx------   3    root   root     4096   Sep  5 10:37 .gconf      <=范例说明处
drwx------   2    root   root     4096   Sep  5 14:09 .gconfd
-rw-r--r--   1    root   root    42304   Sep  4 18:26 install.log <=范例说明处
-rw-r--r--   1    root   root     5661   Sep  4 18:25 install.log.syslog
[    1   ][  2 ][   3  ][  4 ][    5   ][     6     ][       7          ]
[  权限  ][连结][拥有者][群组][文件容量][  修改日期 ][      檔名        ]
```

ls是『list』的意思，重点在显示文件的文件名与相关属性。而选项『-al』则表示列出所有的文件详细的权限与属性 (包含隐藏档，就是文件名第一个字符为『 . 』的文件)。如上所示，在你第一次以root身份登入Linux时， 如果你输入上述指令后，应该有上列的几个东西，先解释一下上面七个字段个别的意思：

![文件属性的示意图](http://cn.linux.vbird.org/linux_basic/0210filepermission_files/0210filepermission_2.gif)

**以档案类型权限为例:**

这个地方最需要注意了！仔细看的话，你应该可以发现这一栏其实共有十个字符：

![文件的类型与权限之内容](http://cn.linux.vbird.org/linux_basic/0210filepermission_files/0210filepermission_3.gif)

- 第一个字符代表这个文件是『

  目录、文件或链接文件

  等等』：

  - 当为[ **d** ]则是目录，例如[上表](http://cn.linux.vbird.org/linux_basic/0210filepermission_2.php#table2.1.1)档名为『.gconf』的那一行；
  - 当为[ **-** ]则是文件，例如[上表](http://cn.linux.vbird.org/linux_basic/0210filepermission_2.php#table2.1.1)档名为『install.log』那一行；
  - 若是[ **l** ]则表示为连结档(link file)；
  - 若是[ **b** ]则表示为装置文件里面的可供储存的接口设备(可随机存取装置)；
  - 若是[ **c** ]则表示为装置文件里面的串行端口设备，例如键盘、鼠标(一次性读取装置)。

- 接下来的字符中，以三个为一组，且均为『rwx』 的三个参数的组合。其中，[ r ]代表可读(read)、[ w ]代表可写(write)、[ x ]代表可执行(execute)。 要注意的是，这三个权限的位置不会改变，如果没有权限，就会出现减号[ - ]而已。

  - 第一组为『文件拥有者的权限』，以『install.log』那个文件为例， 该文件的拥有者可以读写，但不可执行；
  - 第二组为『同群组的权限』；
  - 第三组为『其他非本群组的权限』。
