---
title: angular
date: 2017-09-03 11:07:09
tags: JavaScript
---
## angular.module 
```
var module = angular.module('myApp', [依赖模块])
```
- 第一个参数: 模块名
- 第二个参数: 依赖模块

```
<html lang="en" ng-app="myApp">
<div ng-controller="test">
        <p>价格: <input type="text" ng-model="goods.price"></p>
        <p>数量: <input type="text" ng-model="goods.amount"></p>
        <p>费用: <span>{{getPrice() | currency:'¥'}}</span></p>
        <p>运费: <span>{{goods.fee | currency:'¥'}}</span></p>
        <p>总价: <span>{{getPrice() + goods.fee | currency:'¥'}}</span></p>
    </div>
</html>
<script type="text/javascript">
var module = angular.module('myApp', []);
module.controller('test', function($timeout, $scope, $rootScope) {
    $scope.goods = {
        price: 5000,
        amount: 5,
        fee: 10
    }
    $scope.getPrice = function() {
        return $scope.goods.price * $scope.goods.amount;
    }
    $scope.$watch('goods', function() {
        //监听到goods.pprice变化 做相应操作
        $scope.goods.fee = $scope.getPrice() >= 100 ? 0 : 10;
    }, true)
})
</script>
```
- 为了避免压缩导致参数变化, controller的第二个参数可以变化为数组形式

```
function($scope, $rootScope) {...}
====>
['$scope', '$rootScope', function(p1, p2) {...}]
```
- `module.controller`
- `module.run(['$rootScope', function(p){...}])`
    + run可以不用再dom上指定ng-controller 就可以控制全局变量
- `module.filter` 自定义过滤器

```
<p>{{name | myFilter}}</p>
var module = angular.module('myApp', []);
module.filter('myFilter', function() {
    return function(str) {
        return str.charAt(0).toUpperCase() + str.substring(1);
    }
})
```

- `module.directive` 自定义指令
    + restrict的四种定义方式 
        1. 区分大小写;
        2. 可以组合使用
        3. 注意驼峰转换 
        - `E`: 标签指令
        - `A`: 属性指令
        - `C`: class属性
        - `M`: 注释替换
        ```
        <my-hello></my-hello>
        <div my-hello></div>
        <div class="my-hello"></div>
        <!-- directive:my-hello -->
        module.directive('myHello', function() {
            return {
                restrict: 'EACM',
                replace: true,
                transclude: true,
                controller: ['$scope', function($scope) {
                    this.tips = 'hello world'
                }],
                template: '<div>Hello world <h1 ng-transclude></h1></div>'
            }
        })
        ```
    + place
    + template: 模板标签
    + templateUrl: 模板文件 会用ajax 去请求
    + scope
        - 为true时 指令被服用时, 作用域会独立
        - 为`{}`时, 作用域会和全局作用域隔离开, 会使用自己的controller作用域及变量
            + `@`: 替换为变量
            + `=`: 替换为作用域变量(非自身controller作用域)
            + `&`: 替换为函数(非自身controller作用域)
    ```
    <my-tab aaa="div1" my-name="name" my-fn="show(num)"></my-tab>
        <my-tab aaa="div2" my-name="name" my-fn="show(num)"></my-tab>
    module.directive('myTab', function() {
        return {
            restrict: 'E',
            replace: true,
            templateUrl: './tmp1.html',
            scope: {
                myId: '@aaa',
                myName: '=',
                myFn: '&'
            },
            controller: ['$scope', function($scope) {
                $scope.name = 'mytab'
            }],
            link: function(scope, element, attr) {
                element.delegate('input', 'click', function() {
                    $(this).attr('class', 'active').siblings().attr('class', '')
                    $(this).siblings('div').css({'display': 'none'}).siblings('div').eq($(this).index()).css({'display': 'block'})
                })
            }
        }
    })
    //./tmp1.html
    <input class="active" type="button" value="1" ng-click="myFn({num:123})">
    ```
    - link: 对自身的操作
    - controller: 当对`this`挂载数据时, 就实现了与其他自定义指令, 进行数据共享
    - transclude: 当为true时保留子模板
    - require: 引入其他自定义指令
        + `?`防止报错
        + `^`向上级查找
    ```
    <my-hello><my-hi></my-hi></my-hello>
    module.directive('myHello', function() {
        return {
            restrict: 'EACM',
            replace: true,
            transclude: true,
            controller: ['$scope', function($scope) {
                this.tips = 'hello world'
            }],
            template: '<div>Hello world <h1 ng-transclude></h1></div>'
        }
    })
    module.directive('myHi', function() {
        return {
            restrict: 'EACM',
            replace: true,
            require: '?^myHello',
            template: '<span>hi world</span>',
            link: function(scope1, element1, attr, reController) {
                console.log(reController.tips)
            }
        }
    })
    ```

- module.config 服务商 根据配置信息用于初始化操作 
    + 参数+Provider

```
module.config(['$interpolateProvider', function($interpolateProvider) {
    //替换标识符
    $interpolateProvider.startSymbol('@@');
    $interpolateProvider.endSymbol('@@');
])
```

- module.factory('服务名', fn) 自定义服务

```
module.factory('myService', function() {
    return {
        myServiceFn: function() {
            console.log('hello myService')
        }
    }
})
module.controller('test', ['$scope', 'myService', function ($scope, myService){
    console.log(myService.myServiceFn())
}]);
```

- module.service 支持面向对象的自定义服务

```
module.service('myServiceObj', myServiceObjFn);
function myServiceObjFn() {
    this.name = 'myServiceObjFn'
}
myServiceObjFn.prototype.age = 18
```
- module.constant 自定义常量的服务

```
module.constant('myServiceConst', 'hello myServiceConst')
module.config(['myServiceConst', function(myServiceConst) {
    console.log(myServiceConst)
}])
```
- module.value 自定义常量的服务 但是不能config配置

- module.provider('服务名', fn)
    + 功能与factory类似, 因为factory通过这个方法封装
    + 与factory的区别是: 这个方法可以进行config服务商 进行初始化操作

```
module.provider('myService2', function() {
    return {
        flag: false,
        $get: function() {
            var _this = this;
            return function(num1, num2) {
                var tmp = Math.random() * (num2 - num1) + num1
                return _this.flag ? Math.round(tmp) : tmp;
            }
        }
    }
})
module.config(['myService2Provider', function(myService2Provider) {
    myService2Provider.flag = false;
}])

```

## 指令
- ng-app
    + 初始化作用, 初始化模块
    + 只有在声明了ng-app区域, angular才生效 
- ng-controller

```
//常规模式
module.controller('test', ['$scope', '$rootScope', '$filter', '$interval', function($scope, $rootScope, $filter, $interval) {}];
<div ng-controller="test"></div>
//面向对象形式
module.controller('test2', ['$scope', fn]);
function fn() {}
fn.prototype.name = 'fnObj'
<div ng-controller="fn as fnObj">{{fnObj.name}}</div>
```
- ng-click 点击事件
    + 可以直接修改$scope上的变量 ==> `ng-click="name='aa'"`
    + 也可以在$scope上绑定一个方法再调用
    ```
    ng-click="changeName()"
    $scope.changeName = function() {
            $scope.name = 'name'
        }
    ```
- ng-model 双向绑定
    + `ng-model="name"` 与name变量绑定在一起, name变量的变化会影响绑定对象
    
```
<body >
    <div ng-controller="test">
        <p>价格: <input type="text" ng-model="goods.price"></p>
        <p>数量: <input type="text" ng-model="goods.amount"></p>
        <p>费用: <span>{{getPrice() | currency:'¥'}}</span></p>
        <p>运费: <span>{{goods.fee | currency:'¥'}}</span></p>
        <p>总价: <span>{{getPrice() + goods.fee | currency:'¥'}}</span></p>
    </div>
</body>
<script type="text/javascript">
    function test($timeout, $scope, $rootScope) {
        $scope.goods = {
            price: 5000,
            amount: 5,
            fee: 10
        }
        $scope.getPrice = function() {
            return $scope.goods.price * $scope.goods.amount;
        }
        $scope.$watch('goods', function() {
            //监听到goods.pprice变化 做相应操作
            $scope.goods.fee = $scope.getPrice() >= 100 ? 0 : 10;
        }, true)
    }
</script>
```
- ng-model-option
    + ng-model的一些特殊处理

```
//丢失光标才触发model
<input ng-model="data" ng-model-option="{updateOn: 'blur'}" />
```
- ng-repeat
    + $index 显示索引
    + $first 是否第一个
    + $last 是否最后一个
    + middle 是否非首非尾
    + even 是否偶数
    + odd 是否奇数
    + ng-repeat-start ng-ng-repat-end 两者要配合使用 数据将会在这区间有效输出
```
<ul>
    <li ng-repeat="item in items">{{item.data}}</li>
</ul>
$scope.items = [
        {
            data: 'angular'
        },
        {
            data: 'vue'
        },
        {
            data: 'react'
        }
    ]
```
- ng-select select框选中事件
- ng-change 改变事件 需要model配合
- ng-copy 复制事件
- ng-cut 剪切事件
- ng-paste 粘贴事件
- ng-disabled
- ng-readonly
- ng-checkd
- ng-value
    + 使用原生value时, 当js还没执行时, 在页面上就直接显示`{{变量名}}`, 用户体验不好, 使用`ng-value`就可以避免这些问题

```
value="{{name}}" ===> ng-value="name"
```
- ng-bind
    + ng-bind 与ng-value类似 都是为了提高体验, 在js还没执行时, 模板tag不会直接显示在页面上 

```
<div>{{name}}>>
//===>等价于
<div gn-bind="name"></div>
```
- ng-bind-templete
    + 功能与ng-bind类似 只不过可以直接用模板

```
<div>{{name}},{{name}}>>
//===>等价于
<div gn-bind-templete="{{name}},{{name}}"></div>
```
- ng-bind-html
    + 前面2个是无法解析html便签的, 但这个指令是可以解析的
    + 但是需要引入`ngSanitize`模块

```
var module = angular.module('myApp', ['ngSanitize']);
$scope.test = "<h1>hello</h1>"
<div ng-bind-html="test"></div> 
```
- ng-clock
    + 也是js未渲染完毕, 不显示便签
    + 其原理是先给样式套上`display:none`, 渲染完毕后, 再设置为`display:block`
- ng-non-bindable
    + 不解析便签 
- ng-class
    + 添加class
    + 也可以使用$scope变量, 但变量要用`{{}}`包裹起来
```
<div ng-class="{class名: true, class名2: true}"><div>
```
- ng-style
    + 可以直接设置样式
    + 使用的对象方式 {color: 'red'}
    + 也可以使用$scope变量, 但变量要用`{{}}`包裹起来

```
$scope.style = "{color: 'red', background: 'blue'}"
<div ng-style="{{style}}"><div>
```
- ng-href
- ng-src
- ng-attr-属性名
    + 例如: ng-attr-id
- ng-show 显示 ng-show="true"
- ng-hide 隐藏
- ng-if
    + 功能和`ng-show`类似
    + 但是`ng-if`是对dom进行增删操作 
- ng-switch

```
<input type="checkbox" name="" ng-model="checkbox">
<div ng-switch on="checkbox">
    <p ng-switch-default>ng-switch-default</p>
    <p ng-switch-when="false">ng-switch-when</p>
</div>
```
- ng-open 与detail关联使用
- ng-init 初始化变量, 不用先在$scope上挂载
```
<div ng-init="init='hello'">{{init}}</div>
```
- ng-include 引入其他 模块

### angular重装标签
```
var module = angular.module('myApp', ['ngSanitize']);
module.controller('test2', ['$scope', fn]);
function fn() {}
fn.prototype.colors = [
    {name: 'red'},
    {name: 'yellow'},
    {name: 'blue'},
]
<div ng-controller="fn as fnObj">
    <a href="">{{myColor.name}}</a>
    <select ng-options="color.name for color in fnObj.colors" ng-model="myColor"></select>
    <form name="myForm">
        <input type="email" ng-model="fnObj.name" name="myInput">
        <p>验证通过: {{myForm.myInput.$valid}}</p>
        <p>验证不通过: {{myForm.myInput.$invalid}}</p>
        <p>初始值: {{myForm.myInput.$pristine}}</p>
        <p>修改过: {{myForm.myInput.$dirty}}</p>
        <p>有错误: {{myForm.myInput.$error}}</p>
    </form>
</div>
```
## 参数
- angluar是依赖注入, 所有它传递的形参名称不能修改
- 参数顺序可以改变

### $scope 
- 局部作用域
- 只有在绑定的ng-controller区域才可以使用
- 挂载这个对象上的变量, 才能在模板里显示

```
<body>
    <div ng-controller="test">
        {{name}}
    </div>    
    {{age}}
</body>
<script type="text/javascript">
    function test($scope, $rootScope) {
        $scope.name = 'lc'
        $rootScope.age = 18
    }
</script>
```
#### $scope.$watch
- 监听器
- 第一个参数 监听对象
- 第二个参数 监听触发的回调函数
    + 回调函数带有3个参数
    + 第一个参数, 新的变量值
    + 第二个参数, 旧的变量值 
- 第三个参数 boolen 是否深度监听, 也就是说对象里面的子变量或子对象

```
        $scope.$watch('goods.price', function() {
            //监听到对象变化 做相应操作
        }, true)
```
#### $scope.$apply
- 一般情况下, setTimeout是没法生效的, 必须使用$timeout, 但是使用$scope.$apply, setTimeout就可以生效

```
setTimeout(function() {
        $scope.$apply(function() {
            $scope.goods.amount = 100
        })
    }, 1000)
```

### $rootScope
- 全局作用域
- 挂载在这个对象上的变量, 在全局可生效

### $timeout
- 功能和setTimeout相似, 但setTimeout无法修改angular变量现实双向绑定

```
$timeout(function() {
            $scope.name = 'll'
        }, 1000)
```

### $interval
- $interval.cancel(timer)

```
var timer = $interval(function() {
        $scope.text = iNow + '秒';
        if (! iNow --) {
            $scope.text = '搜索';
            $scope.isDisabled = false
            $interval.cancel(timer)
        }
    }, 1000);
```

### $filter 过滤器
- `$filter('过滤器名').(操作对象, 过滤器参数...)`

```
alert($filter('uppercase')('hello')
alert($filter('limitTo')('hello', 2))
```

### $http 类似ajax的http请求
- 当`method: 'jsonp'`时, `JSON_CALLBACK`为回调函数名
```
$scope.searchByBaidu = function() {
    $timeout.cancel(timer);
    $timeout(function () {
        $http({
            method: 'jsonp',
            url: 'https://sp0.baidu.com/5a1Fazu8AA54nxGko9WTAnF6hhy/su?wd=' + $scope.searchData + '&json=1&cb=JSON_CALLBACK'
        }).success(function(data, state, headers, config) {
            console.log(data)
            $scope.searchResult = data.g
        }).error(function(err) {
            console.log(err)
        })
        
    }, 500);
}
```
### $location 对location的封装
- `$location.path('aaa/bbb')` 在hash后面设置路径

### $anchorScroller 锚点跳转

### cacheFactory 缓存工具
- info() 缓存信息
- get(key) 获取缓存
- put(key, val) 存储缓存
- remove(key)

```
var cache = $cacheFactory('myCache', {capacity: 1}) // capacity 缓存最大空间
cache.put('name', 'lc')
cache.put('age', '18')
console.log(cache.info, cache.get('name'), cache.get('age'))
```
### $log 功能与console类似
### $interpolate 插值替换
```
<input type="text"  ng-model="name">
<textarea ng-model="body"></textarea>
<p>{{showBodyContent}}</p>
$scope.$watch('body', function(newBody) {
    if (newBody) {
        //解析新内容
        var template = $interpolate(newBody);
        //将内容中的name替换为$scope中的name
        $scope.showBodyContent = template({name: $scope.name})
    }
})
```
### $q 跟jquery的primise类似
```
var dfd = $q.defer();
function getPromise() {
    setTimeout(function() {
        if (Math.random() > 0.5) {
            dfd.reject()
        } else {
            dfd.resolve();
        }
    }, 2000);
    return dfd.promise;
}
getPromise().then(function() {
    console.log('success')
}, function() {
    console.log('error')
});
```

## angular 工具方法

### angular.bind
- 功能与$.proxy类似

### angular.extend
- 功能与$.extend 类似

### angular.copy
- 复制功能 

### angular.equals(a, b)
- 这个比较与原生的有所不同
- 这个方法只要比较的两者值一样就相同, 比如NaN

### angular.forEach()
- 接受3个参数, 前2个参数和$一样
- 第三个参数, 可以接受新值

```
var arr = ['a', 'b']
var result = []
angular.forEach(arr, function(val, key){
    this.push(key + val)
}, result)
console.log(result)// ["0a", "1b"]
```

### angular.fromJson
- 将字符串转换为json

### angular.toJson
- 将json转换为字符串
- 第二个参数为true的话, json对象会被格式化

### angular.element 
- 小型jquery

### angular.bootstarp
- 手动进行初始化, 与ng-app功能类似
- 但ng-app一个页面只能有一次初始化; 而bootstrap 可以进行多次初始化操作
- `angular.bootstarp(dom对象, module模块名)`
    + 第一个参数, 在哪个dom对象上初始化
    + 第二个参数, module的模块名


## 过滤器 
> 可以使用管道方式调用多个过滤器 如: num | limitTo: 2 | uppercase

- `currency:'¥'` 货币过滤器
    + `{{goods.price * goods.amount | currency}}` `| currency`是过滤器, 转换为货币格式
- number 每3位一个逗号
- uppercase/lowercase
- json 将json格式化
- limitTo:1 截取数组或者字符串
- date: 'yyyy-mm-dd' 转换为时间
- orderBy:排序属性:是否逆序  排序对象
- filter:值 过滤数据, 仅剩下含有此值的数据 

## 事件
- $emit 冒泡传播
- $broadcast 捕捉传播
- event.stopPropagation 阻止事件传播

```
<div ng-controller="testPropagation">
    {{count}}
    <div ng-controller="testPropagation" ng-click="$emit('myEventInc');$broadcast('myEventDes')">
        {{count}}
        <div ng-controller="testPropagation">
            {{count}}
        </div>
    </div>
</div>
module.controller('testPropagation', function($scope) {
    $scope.count = 0;
    $scope.$on('myEventInc', function(event) {
        $scope.count ++;
        // event.stopPropagation() //阻止事件传播
    })
    $scope.$on('myEventDes', function(event) {
        $scope.count --
    })
})
```
## 插件
### ng-route
```
<div ng-controller="myController">
        <a href="" ng-click="$location.path('/aaa/123')">首页123</a>
        <a href="" ng-click="$location.path('/aaa/345')">首页345</a>
        <a href="#bbb">内容</a>
        <a href="#ccc">关于</a>
        <div ng-view></div>
    </div>
var module = angular.module('myApp', ['ngRoute']);
module.config(['$routeProvider', function($routeProvider) {
    $routeProvider
    .when('/aaa/:num', {
        template: '<div>首页内容</div>{{name}}',
        controller: 'aaa'
    }).when('/bbb', {
        template: '<div>内容部分</div>{{name}}',
        controller: 'bbb'
    }).when('/ccc', {
        template: '<div>关于内容</div>{{name}}',
        controller: 'ccc'
    }).otherwise({
        redirectTo: '/aaa/123'
    });
}])
module.run(['$rootScope', function($rootScope) {
    $rootScope.$on('$routeChangeStart', function(event, current, prev) {
        console.log(event, current, prev)
    })
}])
module.controller('myController', function($scope, $location, $routeParams) {
    $scope.$location = $location;
    console.log($routeParams)
})
module.controller('aaa', function($scope) {
    $scope.name = 'aaa'
})
module.controller('bbb', function($scope) {
    $scope.name = 'bbb'
})
module.controller('ccc', function($scope) {
    $scope.name = 'ccc'
})
```

### ngAnimate
+ 对dom增删动画
    - ng-enter
    - ng-enter-active
    - ng-leave
    - ng-leave-active
+ 对dom隐藏或显示
    - ng-hide-remove
    - ng-hide-remove-active
    - ng-hide-add
    - ng-hide-add-active
```
.box {
    width: 200px;
    height: 200px;
    background-color: red;
    transition: 1s all;
}
.box.ng-enter {
    opacity: 0;
}
.box.ng-enter-active {
    opacity: 1;
}
.box.ng-leave {
    opacity: 1;
}
.box.ng-leave-active {
    opacity: 0;
}
```

- enter
- leave
- addClass
- removeClass

```
<div class="box" ng-if="checkbox"></div>
module.animation('.box', function() {
    return {
        enter: function(element, callback) {
            console.log(element, callback)
        },
        leave: function(element, callback) {
            console.log(element, callback)
        }
    }
})
<div class="box" ng-show="checkbox"></div>
module.animation('.box', function() {
    return {
        addClass: function(element, aclass, callback) {
            console.log(element, aclass, callback)
        },
        removeClass: function(element, aclass, callback) {
            console.log(element, aclass, callback)
        },
    }
})
```

### ngResource 大型资源请求 跟$http类似
- get() get请求
- query() 数组类似数据
- save() post请求
- delete()

```
// 初始化对象, 第一个参数是请求地址 第二个是字符替换, 第三个是初始化配置
var obj = $resource(':name.:format', {format: 'json'}, {});
obj.get({name: 'lisi'}, function(){alert('success')}, function(){alert('error');})
```