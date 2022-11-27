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

