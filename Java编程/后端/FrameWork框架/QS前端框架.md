# QS前端框架

## `qs`是一款可以将对象格式的数据转换成FormData格式的框架。

## 使用原因:

- 在前后端分离的项目中,如果在后端项目中添加了@RequestBody注解,那么就要求我们客户端提交的请求参数**必须**是JSON格式的，例如:

  - ```java
    {
    	name: "华为",
    	pinyin: "HUAWEI"
    }
    ```

- 如果控制器中处理请求的方法的参数**没有添加**此注解，则客户端提交的请求参数**必须**是FormData格式的,例如:

  - ```java
    name=华为&pinyin=HUAWEI
    ```

- 在前端技术中，可以使用`qs`[框架](https://so.csdn.net/so/search?q=框架&spm=1001.2101.3001.7020)可以轻松的将JavaScript对象转换为FormData格式！

  在前端项目中，打开终端，输入命令,安装`qs`：

  ```js
  npm i qs -S
  ```

- 然后,在`main.js`中添加配置:

  ```js
  import qs from 'qs';
  Vue.prototype.qs = qs;
  ```

- 在项目中，当需要将对象转换为FormData格式时，例如：

  ```js
  let formData = this.qs.stringify(this.ruleForm);//ruleForm是JavaScript中的对象
  ```

  

