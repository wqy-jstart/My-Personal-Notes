### 溢出设置overflow

- visible 显示(默认)
- hidden 隐藏   
- scroll 滚动显示

### 行内元素垂直对其方式vertical-align

- top上对齐
- middle中间对齐
- bottom下对齐
- baseline 基线对齐(默认)

### 显示层级z-index

- 当元素脱离文档流后可能出现元素层叠显示的问题, 通过z-index控制元素的显示层级

值越大显示越靠前,  静态定位设置无效



### JavaScript

- 作用: 负责给页面添加动态的效果或动态内容   
- 语言特点:
  - 属于弱类型语言
  - 属于脚本语言,不需要编译由浏览器解析执行
  - 基于面向对象的语言
  - 安全性强: JS语言只能访问浏览器内部的数据,浏览器以外用户电脑中磁盘上面的私人数据是访问不到的. 
  - 交互性强:  因为JS语言是嵌入在html页面中运行在客户端电脑中的, 和用户进行交互更加便捷, Java语言是运行在服务器的语言, 用户进行交互的话每次都需要网络传输.

变量 ,数据类型,运算符,各种语句,方法,面向对象

### 变量

- JS语言属于弱类型语言,声明变量时不需要指定变量的类型

- 通过let或var声明变量

  - let声明的变量作用域和Java语言类似

  - var声明的变量作用域相当于是全局的,可以在任何地方访问

  - 举例:

    - Java: 

      for(int i=0;i<10;i++){

      ​	int j = i+1;

      }

      int x = i+j;   //i和j超出了作用域    编译报错

    - JS:

      for(let i=0;i<10;i++){

      ​	let j = i+1;

      }

      let x = i+j;     //不报错 但是访问不到i和j  因为超出了作用域

      for(var i=0;i<10;i++){

      ​	var j = i+1;

      }

      var  x = i+j;  //不报错并且可以访问到i和j的值

### 数据类型

- JavaScript语言中只有对象类型
- 常见的对象类型:

  - number数值:  相当于Java中所有数值类型的总和 
  - string 字符串:   可以用单引号或双引号修饰       
  - boolean布尔值:   true和false 
  - undefined未定义:   当变量只声明不赋值的时候变量的类型和值都为undefined  
- 获取变量类型:      typeof 变量

### 运算符

- 算术运算符: + - * / %
  - 除法和Java不一样, 会自动根据结果转换整数或小数
    - Java:  int x=5;  int y=2; int z = x/y;  z=2  
    - JS:  let x=5; let y=2;  let z = x/y;   z=2.5    x=6    z=3
- 关系运算符: > < >= <= !=  ==和===
  - ==: 先统一两个变量的类型再比较值    "666"==666     true
  - ===:先比较类型,类型一样后再比较值    "666"==666    false
- 逻辑运算符: && || !     只支持短路与和短路或
- 赋值运算符: =   +=  -=  *=   /=  %=
- 三目运算符:   条件?值1:值2

### 各种语句

- 各种语句和Java语言基本一致

- if else
- while
- for
- switch case

### 如何在html页面中添加JavaScript

- 三种添加方式

  - 内联: 在标签的事件属性中添加JS代码, 当事件触发时执行 
    - 事件: 指系统提供的一些特定时间点
    - 点击事件: 当用户点击元素时触发的时间点

  - 内部: 在html页面中的任意位置添加script标签, 在标签体内写js 代码
  - 外部: 在单独的js文件中写js代码, 在html页面中通过 script标签的src属性引入到页面中.    

### 方法

- Java:   public 返回值类型 方法名(参数列表){方法体}
- JS:       function 方法名(参数列表){方法体}  
- JavaScript中三种声明方法的方式:
  -  function 方法名(参数列表){方法体}  
  - let 方法名 = function(参数列表){方法体}
  - let 方法名 = new Function("参数1","参数2","方法体")

### 和页面相关的方法

1. 通过选择器找到页面中的元素对象  

   let 元素对象 = document.querySelector("选择器");

2. 获取和修改元素的文本内容

   元素对象.innerText       获取

   元素对象.innerText = "xxx";  修改

3. 获取和修改控件的值 

   控件对象.value    获取

   控件对象.value = "xxx";   修改

### NaN

- Not a Number 不是一个数 
- isNaN(变量)   返回值true代表是NaN  false代表不是NaN  



### JavaScript对象分类

- 内置对象:  string , number, boolean等
- BOM: 浏览器对象模型  包含和浏览器相关的内容
- DOM: 文档对象模型, 包含和页面相关的内容

### BOM浏览器对象模型

- Browser Object Model 浏览器对象模型
- window对象:  此对象中的属性和方法称为全局属性和全局方法, 访问的时候可以省略掉window.     
- window中常用的方法:
  - window.isNaN(变量)  判断变量是否是NaN 
  - parseInt()/parseFloat() 将字符串转成整数或小数, 还可以将小数转成整数
  - alert("xxx"); 弹出提示框
  - confirm("xxx"); 弹出确认框 
  - prompt("xxx"); 弹出文本框
  - let timer = setInterval(方法,时间间隔);  开启定时器
  - clearInterval(timer); 停止定时器
  - setTimeout(方法,时间间隔); 开启只执行一次的定时器
- window中常用的属性
  - location位置
    - location.href  获取或修改浏览器的请求地址
    - location.reload()    刷新页面
  - history历史
    - history.length 获取历史页面的数量
    - history.back()   返回上一页面
    - history.forward() 前往下一页面

### DOM文档对象模型

- Document Object Model  包含和页面相关的内容

1. 通过选择器找到页面中的元素对象  

   let 元素对象 = document.querySelector("选择器");

2. 获取和修改元素的文本内容

   元素对象.innerText       获取

   元素对象.innerText = "xxx";  修改

3. 获取和修改控件的值 

   控件对象.value    获取

   控件对象.value = "xxx";   修改

4. 创建元素对象

   let 元素对象 = document.createElement("标签名");

5. 添加元素对象到某个元素里面

   元素对象.append(新的元素对象);