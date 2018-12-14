---
title: Angularjs(内置服务)
date: 2018-01-15 21:17:06
tags: js
categories: js
---

#### AngularJs 内置服务

Angular.js 为我们提供了一些内置的方法,而且使用起来很方便.

##### $timeout服务

语法格式:$timeout(函数,间隔时间(单位为毫秒));但该方法只会调用一次.

```javascript
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>AngularJs $timeout</title>
    <meta name="Copyright" content="All Rights Reserved"/>
    <script src="js/angular.min.js"></script>
    <script type="text/javascript">
        // 定义一个模块
        var app = angular.module('myApp',[]);
        // 模块添加控制器层
        app.controller('myController', function($scope, $timeout){
            $scope.count = 1;
            // 定义方法
            var calc = function(){
                $scope.count += 1;
                // $timeout服务，延迟1000毫秒调用一次
                var timer = $timeout(function(){
                   calc();
                }, 1000);
                if ($scope.count == 5){
                    // 取消调度
                    $timeout.cancel(timer);
                }
            };
            calc();
        });
    </script>
</head>
<body ng-app="myApp" ng-controller="myController">
   {{count}}
</body>
</html>

```

##### $interval服务

$interval服务就相当于js中的windows.setInterval 函数

语法格式:$interval(函数,间隔时间毫秒,总调用次数);

```javascript
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>AngularJs $interval</title>
    <meta name="Copyright" content="All Rights Reserved"/>
    <script src="js/angular.min.js"></script>
    <script type="text/javascript">
        // 定义一个模块
        var app = angular.module('myApp',[]);
        // 模块添加控制器层
        app.controller('myController', function($scope, $interval){
            $scope.count = 1;
            // $interval服务，每隔1000毫秒调用一次
            var timer = $interval(function(){
                $scope.count += 1;
                if ($scope.count == 5){
                    // 取消调度
                    $interval.cancel(timer);
                }
            }, 1000, 2);
        });
    </script>
</head>
<body ng-app="myApp" ng-controller="myController">
   {{count}}
</body>
</html>
```

#### $watch 服务

$watch 用于持续监听某变量的变化,更新界面.

```javascript
<!DOCTYPE html>
<html>
	<head>
		<title>$watch服务</title>
		<meta charset="UTF-8"/>
		<meta http-equiv="pragma" content="no-cache"/>
		<script type="text/javascript" src="js/angular.min.js"></script>
		<script type="text/javascript">
			/** 定义模块 */
			var myApp = angular.module("myApp",[]);
			/** 模块中添加控制器 */
			myApp.controller("myController", function($scope){
				// $watch持续监听变量变化
				$scope.$watch("name", function(newVal, oldVal){
					$scope.dd = "新值：" + newVal + " 旧值：" + oldVal;
				});
			});
		</script>
	</head>
	<body ng-app="myApp" ng-controller="myController">
		<input type="text" ng-model="name"/>
		{{dd}}
	</body>
</html>
```

#### $http 服务

一般从后端获取参数,都是通过该内置服务$http服务来实现的.

语法格式:(该种使用方法比较全面)

```javascript
$http({
	method : 'get|post', // 请求方式
	url : '', // 请求URL
	params : {'name':'admin'} // 请求参数
}).then(function(response){ // 请求成功
	// response: 响应对象封装了响应数据、状态码
},function(response){ // 请求失败
	// response:  响应对象封装了响应状态码
});
```

下面就使用一个跟Java后端服务器交互的异步请求的例子:

使用步骤:

1. 前端页面发送异步请求.
2. 后端服务器接收请求参数,并响应数据到前端页面.

前端页面:

```javascript
<!DOCTYPE html>
<html>
	<head>
		<title>$http</title>
		<meta charset="UTF-8"/>
		<meta http-equiv="pragma" content="no-cache"/>
		<script type="text/javascript" src="js/angular.min.js"></script>
		<script type="text/javascript">
			/** 定义模块 */
			var myApp = angular.module("myApp",[]);
			/** 模块中添加控制器 */
			myApp.controller("myController", function($scope, $http){
				// 发送异步请求
				$scope.httpGet = function(){
					$http({
						method : "GET", // 请求方式
						url : "/city", // 请求URL
						params : {"id":10} // 请求参数
					}).then(function(response){ // 请求成功
						// response响应对象
						$scope.cities = response.data;
					},function(response){ // 请求失败
						alert("数据加载失败！");
					});
				};
			});
		</script>
	</head>
	<body ng-app="myApp" ng-controller="myController" 
		  ng-init="httpGet();">
		<select ng-model="code" 
ng-options="city.id as city.name for city in cities">
			<option value="">==请选择城市==</option>
		</select>
		{{code}}
	</body>
</html>
```

后端服务器使用了SpringMVC

注意:get的请求不需要加@RequestBody.Post请求需要加上@RequestBody注解.

```java
@RestController
public class TestController {
	
	/** 处理get请求与post请求 */
	@RequestMapping("/city")
	public List<Map<String,String>> getCitys(@RequestParam("id")Long id){
		System.out.println("请求参数id: " + id);
		List<Map<String,String>> data = new ArrayList<>();
		Map<String,String> city1 = new HashMap<>();
		city1.put("id", "1");
		city1.put("name", "广州市");
		Map<String,String> city2 = new HashMap<>();
		city2.put("id", "2");
		city2.put("name", "深圳市");
		Map<String,String> city3 = new HashMap<>();
		city3.put("id", "3");
		city3.put("name", "惠州市");
		data.add(city1);
		data.add(city2);
		data.add(city3);
		return data;
	}
}
```

然而这样编写的代码好像有点多,我们可以试着简化一下.

##### $http.get()发送请求

```javascript
// 第一种格式
$http.get(URL,{  
    params: {  
       "id":id  
    }  
}).then(function(response){// 请求成功
	// response: 响应对象封装了响应数据、状态码
}, function(response){ // 请求失败
	// response: 响应对象封装了响应状态码
});

// 第二种格式
$http.get(URL).then(function(response){ // 请求成功
	// response: 响应对象封装了响应数据、状态码
},function(response){ // 请求失败
	// response: 响应对象封装了响应状态码
});
```

##### $http.post()发送请求

```javascript
// 第一种方式
$http.post(URL,{  
    "id" : id  
}).then(function(response){ // 请求成功
	// response: 响应对象封装了响应数据、状态码
},function(response){ // 请求失败
	// response: 响应对象封装了响应状态码
});
```

再举一个栗子吧:

```javascript
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>AngularJs $http.post()方法</title>
    <meta name="Copyright" content="All Rights Reserved"/>
    <script src="js/angular.min.js"></script>
    <script type="text/javascript">
        // 定义一个模块
        var app = angular.module('myApp',[]);
        // 模块添加控制器层
        app.controller('myController', function($scope, $http){
            $scope.sendPost = function(){
                // $http.get()发送异步请求
                $http.post("/user", {"name" : "张三"})
                    .then(function(response){ // 请求成功
                        if(response.status == 200){
                            $scope.res = response.data;
                        }
                    }, function(response){ // 请求失败
                        alert("数据加载失败！");
                    });
            };
        });
    </script>
</head>
<body ng-app="myApp" ng-controller="myController"
      ng-init="sendPost();">
   {{res.msg}}
</body>
</html>
```

Java后端服务器:

谨记POST请求,在Controller中接收参数,在形参上需要加上@RequestBody 注解.

```java
/** $http.post()请求 */
@PostMapping("/user")
public Map<String,String> user(@RequestBody Map<String,String> map){
    System.out.println("请求参数name: " + map.get("name"));
    Map<String,String> data = new HashMap<>();
    data.put("msg", map.get("name") + "，您好！");
    return data;
}
```

以上就是暂时我对Angularjs内置服务的理解,有什么说错的,请各位大神见谅.也很乐意接收各位大大的意见.