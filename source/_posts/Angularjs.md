---
title: Angularjs(基础语法)
date: 2018-01-10 23:30:03
tags: js
categories: js
---

![C0YOPg.png](https://s1.ax1x.com/2018/05/10/C0YOPg.png)

#### Angular.js 四大特征

1. ##### MVC模式

   Angular遵循软件工程的MVC模式,并鼓励展现，数据，和逻辑组件之间的松耦合.通过依赖注入（dependency injection），Angular为客户端的Web应用带来了传统服务端的服务，例如独立于视图的控制。 因此，后端减少了许多负担，产生了更轻的Web应用。

   Model:数据,其实就是angular变量($scope.XX);

   View: 数据的呈现,Html+Directive(指令);

   Controller:操作数据,就是function,数据的增删改查;

   <!--more --> 

   ![C0Yvxs.png](https://s1.ax1x.com/2018/05/10/C0Yvxs.png)

2. ##### 双向绑定

   AngularJS是建立在这样的信念上的：即声明式编程应该用于构建用户界面以及编写软件构建，而指令式编程非常适合来表示业务逻辑。框架采用并扩展了传统HTML，通过双向的数据绑定来适应动态内容，双向的数据绑定允许模型和视图之间的自动同步。因此，AngularJS使得对DOM的操作不再重要并提升了可测试性

3. ##### 依赖注入

   依赖注入(Dependency Injection,简称DI)是一种设计模式, 指某个对象依赖的其他对象无需手工创建，只需要“吼一嗓子”，则此对象在创建时，其依赖的对象由框架来自动创建并注入进来,其实就是最少知识法则;模块中所有的service和provider两类对象，都可以根据形参名称实现DI.

4. ##### 模块化设计

   高内聚低耦合法则：

   1)官方提供的模块  ng、ngRoute、ngAnimate

     2)用户自定义的模块 angular.module('模块名',[ ])

### 基础指令

##### 表达式:

==语法格式：{{变量名}}|{{对象.变量名}}==

```javascript
<!DOCTYPE html>
<html>
	<head>
		<title>AngularJS表达式</title>
		<meta charset="UTF-8"/>
		<meta http-equiv="pragma" content="no-cache"/>
		<script src="js/angular.min.js"></script>
	</head>
	<body ng-app>
		{{100 + 100 - 100}}<br/>
		{{100 * 100}}<br/>
		{{100 > 100}}<br/>
	</body>
</html>

```

ng-app 指令定义了 AngularJS 应用的根元素，在根元素的所有子元素中用到指令，angularJs会自动识别。

也可以放在<div ng-app></div>里面.但作用范围只在div标签里.

==ng-app 指令在网页加载完毕时会自动初始化应用中的angularJS的指令。==

##### 双向绑定

==语法格式：ng-model=”变量名”|ng-model=”对象.变量名”==

```javascript
<!DOCTYPE html>
<html>
	<head>
		<title>AngularJS双向绑定</title>
		<meta charset="UTF-8"/>
		<meta http-equiv="pragma" content="no-cache"/>
		<script type="text/javascript" src="js/angular.min.js"></script>
	</head>
	<body ng-app>
		姓名：<input type="text" ng-model="name"/><br/>
		性别：<input type="text" ng-model="user.sex"/><br/>
		年龄：<input type="text" ng-model="user.age"/><br/>
		{{name}}，您好！，性别：{{user.sex}}，年龄：{{user.age}}
	</body>
</html>

```

==ng-model  用于绑定input文本框中的内容.并放置到$scope中.==

##### 初始化指令

==语法格式：ng-init=”变量名=’变量值’;变量名=’变量值’”==

```javascript
<!DOCTYPE html>
<html>
	<head>
		<title>AngularJS变量初始化</title>
		<meta charset="UTF-8"/>
		<meta http-equiv="pragma" content="no-cache"/>
		<script type="text/javascript" src="js/angular.min.js"></script>
	</head>
	<body ng-app ng-init="name='小明';user.sex='男';user.age=18">
		姓名：<input type="text" ng-model="name"/><br/>
		性别：<input type="text" ng-model="user.sex"/><br/>
		年龄：<input type="text" ng-model="user.age"/><br/>
		{{name}}，您好！，性别：{{user.sex}}，年龄：{{user.age}}
	</body>
</html>

```

##### 控制器

#####  ==定义模块语法格式：var 变量名 = angular.module(“模块名”, []);==

​    定义控制器语法格式：
==模块变量名.controller(“控制器名”, function($scope){});==

```javascript
<!DOCTYPE html>
<html>
	<head>
		<title>AngularJS控制器</title>
		<meta charset="UTF-8"/>
		<meta http-equiv="pragma" content="no-cache"/>
		<script type="text/javascript" src="js/angular.min.js"></script>
		<script type="text/javascript">
			/** 定义myApp模块 */
			var myApp = angular.module("myApp",[]);

			/** 为myApp模块添加控制器 */
			myApp.controller("myController", function($scope){
				/** 定义add方法 */
				$scope.add = function(){
					return parseInt($scope.x) + parseInt($scope.y);
				};
			});
		</script>
	</head>
	<body ng-app="myApp" ng-controller="myController">
		x：<input type="text" ng-model="x"/><br/>
		y：<input type="text" ng-model="y"/><br/>
		运算结果：{{add()}}
	</body>
</html>
```

$scope 贯穿整个 AngularJS App应用,它与数据模型相关联,同时也是表达式执行的上下文.有了$scope 就在视图和控制器之间建立了一个通道,基于作用域视图在修改数据时会立刻更新 $scope,同样的$scope 发生改变时也会立刻重新渲染视图.

##### 事件指令

- ng-click：单击事件
- ng-dblclick：双击事件
- ng-blur：失去焦点事件
- ng-focus：获取焦点事件
- ng-change：对应onchange改变事件
- ng-keydown：键盘按键按下事件
- ng-keyup：键盘按键按下并松开
- ng-keypress：同上
- ng-mousedown：鼠标按下事件
- ng-mouseup：鼠标按下弹起
- ng-mouseenter：鼠标进入事件
- ng-mouseleave：鼠标离开事件
- ng-mousemove：鼠标移动事件
- ng-mouseover：鼠标进入事件

==语法格式：ng-xxx=”控制器中定义的方法名();”;==

```javascript
<!DOCTYPE html>
<html>
	<head>
		<title>AngularJS事件指令</title>
		<meta charset="UTF-8"/>
		<meta http-equiv="pragma" content="no-cache"/>
		<script type="text/javascript" src="js/angular.min.js"></script>
		<script type="text/javascript">
			/** 定义模块 */
			var myApp = angular.module("myApp",[]);
			/** 模块中添加控制器 */
			myApp.controller("myController", function($scope){
				/** 定义方法 */
				$scope.add = function(){
					$scope.count = parseInt($scope.x) + 
parseInt($scope.y);
				};
				/** 定义方法 */
				$scope.blur = function(){
					alert($scope.x);
				};
				/** 定义方法 */
				$scope.keyup = function(){
					alert($scope.y);
				};
			});
		</script>
	</head>
	<body ng-app="myApp" ng-controller="myController">
		x：<input type="text" ng-model="x" ng-blur="blur();"/><br/>
		y：<input type="text" ng-model="y" ng-keyup="keyup()"/><br/>
		<input type="button" value="计算" ng-click="add();"/>
		运算结果：{{count}}
	</body>
</html>
```

说明：ng-xxx事件指令，绑定控制器的某个方法。

##### 循环数组

==语法格式：ng-repeat=”变量名 in 集合或数组”;==

```javascript
<!DOCTYPE html>
<html>
	<head>
		<title>AngularJS循环数组</title>
		<meta charset="UTF-8"/>
		<meta http-equiv="pragma" content="no-cache"/>
		<script type="text/javascript" src="js/angular.min.js"></script>
		<script type="text/javascript">
			/** 定义模块 */
			var myApp = angular.module("myApp",[]);
			/** 模块中添加控制器 */
			myApp.controller("myController", function($scope){
				/** 定义数组 */
				$scope.list = [100,200,300,400,500];
			});
		</script>
	</head>
	<body ng-app="myApp" ng-controller="myController">
		<ul>
			<li ng-repeat="item in list">
				{{item}}
			</li>
		</ul>
	</body>
</html>
```

==说明：这里的ng-repeat指令用于循环数组变量。==

##### 循环对象数组

$index：获取迭代时的索引号。

```javascript
<!DOCTYPE html>
<html>
	<head>
		<title>AngularJS循环对象数组</title>
		<meta charset="UTF-8"/>
		<meta http-equiv="pragma" content="no-cache"/>
		<script type="text/javascript" src="js/angular.min.js"></script>
		<script type="text/javascript">
			/** 定义模块 */
			var myApp = angular.module("myApp",[]);
			/** 模块中添加控制器 */
			myApp.controller("myController", function($scope){
				/** 定义对象数组 */
				$scope.list =[
		  			{name : "小明", sex : "男", age : 30},
					{name : "中明", sex : "女", age : 26},
					{name : "大明", sex : "男", age : 20}];
			});
		</script>
	</head>
	<body ng-app="myApp" ng-controller="myController">
		<table border="1">
			<tr>
				<th>编号</th>
				<th>姓名</th>
				<th>性别</th>
				<th>年龄</th>
			</tr>
			<tr ng-repeat="u in list">
				<td>{{$index + 1}}</td>
				<td>{{u.name}}</td>
				<td>{{u.sex}}</td>
				<td>{{u.age}}</td>
			</tr>
		</table>
	</body>
</html>

```

##### 条件指令

语法格式：ng-if=”条件表达式”;

```javascript
<!DOCTYPE html>
<html>
	<head>
		<title>AngularJS条件指令</title>
		<meta charset="UTF-8"/>
		<meta http-equiv="pragma" content="no-cache"/>
		<script type="text/javascript" src="js/angular.min.js"></script>
		<script type="text/javascript">
			/** 定义模块 */
			var myApp = angular.module("myApp",[]);
			/** 模块中添加控制器 */
			myApp.controller("myController", function($scope){
				/** 定义对象 */
				$scope.user = {age : 20};
			});
		</script>
	</head>
	<body ng-app="myApp" ng-controller="myController">
		<h2 ng-if="user.age <= 20">{{user.age}}</h2>
	</body>
</html>

```

##### 复选框

- ng-true-value="true": 选中值

- ng-false-value="false": 未选中值

- ng-checked=”true|false”: 是否选中复选框 

  通过在scope中的定义的值.再在复选框中定义一个选中事件来控制复选框的状态.

  ```javascript
  <!DOCTYPE html>
  <html>
  	<head>
  		<title>checkbox</title>
  		<meta charset="UTF-8"/>
  		<meta http-equiv="pragma" content="no-cache"/>
  		<script type="text/javascript" src="js/angular.min.js"></script>
  		<script type="text/javascript">
  			/** 定义模块 */
  			var myApp = angular.module("myApp",[]);
  			/** 模块中添加控制器 */
  			myApp.controller("myController", function($scope){
  				// 第一个复选框的值
  				$scope.value1 = false;
  				// 第二个复选框的值
  				$scope.value2 = "2";
  				// 全选复选框的值
  				$scope.ck = false;
  			});
  		</script>
  	</head>
  	<body ng-app="myApp" ng-controller="myController">
  		<input type="checkbox" 
  				ng-model="value1"/>1<br/>
  		<input type="checkbox" 
  				ng-true-value="1"
  				ng-false-value="2"
  				ng-model="value2"/>2<br/>
  		{{value1}} ---> {{value2}}
  		<hr/>
  		<input type="checkbox" ng-model="ck"/>全选<br/>
  		<input type="checkbox" ng-checked="ck"/>广州<br/>
  		<input type="checkbox" ng-checked="ck"/>深圳<br/>
  		<input type="checkbox" ng-checked="ck"/>东莞<br/>
  	</body>
  </html>
  ```

  ​

##### 下拉列表框

- ng-options="元素变量.键 as 元素变量.键 for 元素变量in 数组"：选项值表达式绑定
- ng-selected=”true|false”: 是否选中下拉列表框指定的选项

```javascript
<!DOCTYPE html>
<html>
	<head>
		<title>select</title>
		<meta charset="UTF-8"/>
		<meta http-equiv="pragma" content="no-cache"/>
		<script type="text/javascript" src="js/angular.min.js"></script>
		<script type="text/javascript">
			/** 定义模块 */
			var myApp = angular.module("myApp",[]);
			/** 模块中添加控制器 */
			myApp.controller("myController", function($scope){
				// 定义下拉列表框需要的数据变量
				$scope.cities = [{id:1, name:'广州市'},
				                 {id:2, name:'深圳市'},
				                 {id:3, name:'珠海市'},
				                 {id:4, name:'东莞市'}];
				
			});
		</script>
	</head>
	<body ng-app="myApp" ng-controller="myController">
		<select ng-model="code" 
ng-options="city.id as city.name for city in cities">
			<option value="">==请选择城市==</option>
		</select>
		{{code}}
		<hr/>
		
		<select>
			<option value="1">中国</option>
			<option value="2" ng-selected="true">美国</option>
		</select>
	</body>
</html>
```

