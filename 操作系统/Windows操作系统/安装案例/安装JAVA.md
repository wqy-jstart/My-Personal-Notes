# 安装JAVA

##### 1.下载JDK：

华为JDK地址：https://mirrors.huaweicloud.com/java/jdk/

##### 2.配置环境变量：

1. 打开资源管理器->我的电脑右键属性->高级系统设置->环境变量

2. 配置两个系统变量

   ```sh
   # 第一个
   变量名：JAVA_HOME
   变量值：JDK路径(bin前一级)
   # 第二个
   变量名：CLASSPATH
   变量值：.;%JAVA_HOME%\lib\dt.jar;%JAVA_HOME%\lib\tools.jar
   ```

3. 双击进入系统的path变量

   ```sh
   1.添加变量值：%JAVA_HOME%\bin
   2.添加变量值：%JAVA_HOME%\jre\bin
   3.添加变量值：JDK含bin的目录路径
   ```

##### 3.打开windows的Dos窗口

测试：

```sh
javac
java -version
```

出现版本号即成功！