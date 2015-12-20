title: AngularJs单页面路由
---

## Modules
对于大型应用，建议分为多个模块：

- 服务模块
- 指令模块
- 过滤器模块
- 一个应用模块依赖于上述三个模块，而且包含初始化及启动代码

模块是配置代码块和运行代码块的集合：

1.**配置代码块** 在provider注册和配置阶段执行。只有provider和constant可以被注入配置代码块，这是为了防止服务在完全配置好之前被意外初始化。

2.**执行代码块** 在injector被创建后执行，用来启动整个应用。只有服务的实例对象和constant可以被注入到执行代码块，这是防止在应用执行期间系统的更进一步的配置。

`angular.module('myModule', []).config(function(injectables) { // provider型注入器
    // 这是配置(config)代码块的范例，你可以有任意多个配置代码块
    // 配置块中你只能注入Provider类（注意：不是由Provider类生成的实例）以及`constant`
  }).`
  `run(function(injectables) { // instance型注入器
    // 这是运行(run)代码块的范例，你可以有任意个运行代码块
    // 运行块中你只能注入Provider实例（注意：不是Provider类）
  });`

## 路由机制

首先了解单页面应用，就是singlepage app。为了实现无刷新的试图切换，我们会使用ajax请求从后台获取数据，然后套上HTML模板，渲染在页面上，但是ajax的一个致命缺点就是到时浏览器后退按钮失效。尽管我页面上防止了一个大大的返回按钮，让用户点击来导航，但是总是无法避免用户习惯点击后退。解决此问题的一个方法是使用hash，监听hashchange事件来进行视图切换。另一个方法是用HTML5的history API，通过oushState（）记录操作历史，建通popstate时间来进行视图切换。也有人把这个叫做pjax技术。
![](http://i.imgur.com/V9f1y8u.png)
如此一来便形成了通过地址栏进行导航的深度链接，也就是我们需要的路由机制。通过路由机制，一个单应用的各个视图就可以很好的组织起来了。

ngRoute的内容

ng的路由机制是靠ngRoute提供的，通过hash和history两种方式实现路由，可以检测浏览器是否支持history来灵活调用相应的方式。ng的路由是一个单独的模块，包含以下内容：  
服务$routeProvider用来定义一个路由表，也就是地址栏与视图模板的映射  
服务$routeParams保存了地址栏中的参数，例如{id：1，name：'tom'}  
服务$route完成路由匹配，并且提供路由相关的属性访问事件，如访问当前路由对应的controller  
指令ngView用来在主视图中指定加载子视图的区域  
以上的内容加上$location服务，我们就可以实现一个单页面应用了。

### 使用
##### 1.引入文件和依赖
ngRoute模块包含在一个单独的文件中，所以需要在页面上引入这个文件，
`<script src="http://code.angularjs.org/1.2.5/angular.min.js"></script>`
`<script src="http://code.angularjs.org/1.2.5/angular-route.min.js"></script>`

引入之后还需要在模板声明中注入对ngRoute的依赖：
`var app = angular.module('MyApp', ['ngRoute'])`

完成了这些，我们就可以在模板或者是controller中使用上面的服务和指令了。
##### 2.定义路由表
$routeProvider提供了定义路由表的服务，它有两个核心方法，when(path,route)和otherwise(params)。  
　　when(path, route)方法接收两个参数，path是一个String类型，表示该条路由规则所匹配的路径，它将与地址栏的内容($location.path)的值进行匹配。如果需要匹配参数，可以在path中使用冒号加名称的方式。如：
path为`/show/:name`，如果地址栏中是：`/show/tom`,那么参数name所对应的值tom便会被保存在$routeParams中，像这样：{name:tom}。我们也可以使用*进行模糊匹配，如：`/show*/:name`，将匹配
/showInfo/tom。  
　　route参数是一个object，用来指定当path匹配以后所需要的一系列配置项，包括以下内容：  
controller //function 或者string类型。在当前模板上执行的controller函数，生成新的scope。  
controllerAs //string类型，为controller指定别名  
template //string或function类型，视图所用的模板，这部分内容将被ngView引用  
templateUrl //string或function类型，当视图模板为单独的html或者是使用了<scripttype="text/ng-template">定义模板时使用  
resolve //指定当前controller所依赖的其他模块  
redirectTo //重定向的地址  

最简单的情况，我们定义一个html文件为模板，并初始化一个指定的controller：  

    function emailRouteConfig($routeProvider){
    $routeProvider.
    when('/show', {
        controller: ShowController,
        templateUrl: 'show.html'
    }).
    when('/put/:name',{
       controller: PutController,
       templateUrl: 'put.html'
    });
	};

otherwise(params)方法对应路径匹配不到时的情况，这时候我们可以配置一个redirectTo参数，让它重定向到404页面或者是首页。
##### 3.在主视图模板中指定加载子视图的位置
单页面应用是局部刷新的，ngView用在哪里哪里就是刷新：  
`<div ng-view></div>`  

我们的子视图将会在此处被引用进来。完成这三步后我们的成的路由就配置好了。