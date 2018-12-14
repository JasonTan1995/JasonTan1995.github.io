---
title: Thymeleaf(1)入门
date: 2018-09-01 22:45:18
tags: Thymeleaf
categories: Spring
---

![](https://ws1.sinaimg.cn/large/6b297ce5gy1fuuflelmx6j20k80503ys.jpg)

​       因为最近工作中使用到了Springboot，而Thymeleaf是Springboot推荐使用的模板引擎（Springboot默认已经不支持JSP），所以记录一些工作中使用到Thymeleaf的地方。

官方网站：https://www.thymeleaf.org/

官网会有一些基本的使用教程，并且有PDF版的教程下载。

​       首先Thymeleaf是一个支持html原型的自然引擎，它在html标签增加额外的属性来达到模板+数据的展示方式，由于浏览器解释html时，忽略未定义的标签属性，因此Thymeleaf的模板可以静态运行。接下来就每个标签的作用来详细说一下。由于原本的html中并没有引入Thymeleaf的命名空间，因此要使用Thymeleaf的标签，需要引入命名空间。（NameSpace）

<!-- more-->

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title></title>
    <meta charset="UTF-8">
</head>
</html>    
```



#### th:fragment

　　该标签应用于布局的情况下，在日常开发中会经常把导航栏，页尾，菜单等部分提取成模板给其他页面使用．那么在Thymeleaf中，就可以使用th：fragment属性来定义一个模板。

 ```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title></title>
    <meta charset="UTF-8">
</head>
  <body>
       <div th:fragment="foot">
       @2018 jasonTan
    </div>
  </body>
</html>    
 ```

​       在上面定义了一个叫foot的片段，接下来就可以使用th：include或者th：replace属性来使用它。

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <title></title>
    <meta charset="UTF-8">
</head>
  <body>
     <div th:include="footer::foot"/>
  </body>
</html> 
```

​     th:include中的参数格式为模板名：dom选择器（ID选择器）。如果只写模板名，不写选择器，会加载整个模板。

​     在Thymeleaf中可以使用两个属性来进行引入片段：th：include和th：replace,它们的区别是：th：include是加载模板内容，而th：replace则会替换当前标签为模板中的标签。下面通过一个栗子来看看。

```html
<body> 
  <div th:include="footer :: foot"></div>
  <div th:replace="footer :: foot"></div>
 </body>
```

​      返回的html：

```html
<body> 
   <div> &copy; 2016 </div> 
  <footer>&copy; 2016 </footer> 
</body>
```

​        在这次项目中,使用到了th：fragment来抽取了一些公共的Js，Css等。（定义一个head.html）引入JS使用th：src，引入Css文件就使用th：href。

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head th:fragment="common_header(title,links,scripts)">
    <title th:replace="${title}"></title>
    <meta charset="UTF-8">

    <link rel="stylesheet" type="text/css" th:href="@{/vendor/bootstrap/css/bootstrap.min.css}">
    <link rel="stylesheet" type="text/css" th:href="@{/css/bootstrap-select.min.css}">
    <script th:src="@{/js/jquery/jquery.min.js}"></script>
    <script type="text/javascript"  th:src="@{/js/jquery/jquery-ui.min.js}"></script>
    <script type="text/javascript"   th:src="@{/js/bootstrap.min.js}"></script>
    <script type="text/javascript"  th:src="@{/js/bootstrap-select.min.js}"></script>

    <th:block th:replace="${links}"/>
    <th:block th:replace="${scripts}"/>
</head>
</html>
```

​         在其他页面引用：（注意这里调用是link，script并不是links，scripts）

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head th:replace="fragment/head :: common_header(~{::title},~{::link},~{::script})">
    <title>RMS</title>
    <link rel="stylesheet" href="">
    <script th:src="@{/js/login.js}"></script>
</head>
  <body>
  </body>
</html>
```



#### th:if

   该标签的作用：如果判断为true才在页面中显示出来，为false则不显示。

   有时候一个用户的权限就可以使用到该标签来控制比如说按钮这些组件。

```html
<input th:if="${session.user.allowAction == 'Add'}" name="Create" type="button" value="New" onClick="goTo('new')" class="btn btn-primary" />
```

​    这里的session为Thymeleaf的内置对象，用于获取session中的值。param则为获取请求的参数的属性，application是获取应用程序上下文的属性。

#### th:each

​    该标签用于循环遍历集合。

​    格式：变量名:${在后台传过来的值的名称}

```html
<select>
    <option th:each="c:${menuList}" th:value="${c.menuId}" th:text="${c.menuName}"></option>
</select>		
```



####  取值符号${},#{},*{},@{}

         ##### ${}

​      变量表达式(美元表达式) 用于访问容器上下文环境中的变量，功能同jsp中${}

        ```java
  @RequestMapping("login")
    public String login(Model model) {
        model.addAttribute("hello","Hello World!");
        return "index";
    }

 //在页面中取值
<h1 th:text="${hello}"></h1>
        ```

##### #{}

​     消息表达式。通常与th:text属性一起使用,th:text的标签文本是#{}中的key所对应的value,而标签内的文本将不会显示。

 ```html
<p th: text=" #{home. welcome}" >This text will not be show! </p>

//新建 templates/home.properties
home.welcome=this messages is from home.properties!
 ```

​       可以看出消息表达式通常用于显示页面静态文本，将静态文本维护在properties文件中也方面维护，做国际化等。

##### *{}

​     选择表达式(星号表达式)。选择表达式与变量表达式有一个重要的区别：选择表达式计算的是选定的对象，而不是整个环境变量映射。也就是：只要是没有选择的对象，选择表达式与变量表达式的语法是完全一样的。我们可以通过*{}，直接获取对象的属性,可以通过th:object对象属性绑定的对象。

```html
<div th: obj ect=" ${session. user}" >
<p>Name: <span th: text=" *{firstName}" >Sebastian</span>.</p>
<p>Surname: <span th: text=" *{lastName}" >Pepper</span>.</p>
<p>Nationality: <span th: text=" *{nationality}" >Saturn</span>.</p>
</div>
```

 ##### @{}

​       链接url表达式。

```html
<link rel="stylesheet" type="text/css" th:href="@{/css/bootstrap-select.min.css}">
<script th:src="@{/js/jquery/jquery.min.js}"></script>		
```



以上就是我最近使用到Thymeleaf的地方，表达式以及一些标签的用法。

参考文章：

表达式详解：http://www.cnblogs.com/hjwublog/p/5051632.html　

Thymeleaf的模板使用：https://www.cnblogs.com/lazio10000/p/5603955.html





​         





