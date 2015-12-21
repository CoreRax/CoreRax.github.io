title: AngularMVC基础
---

### Controller使用过程中的注意点  
- 不要试图复用Controller，一个控制器一般只负责一小块视图  
- 不要再Controller中操作DOM，对与DOM的操作应该封装给指令
- 不要再Controller中做数据格式化，有很好用的表单控件
- 不要再controller中做数据过滤，angular中有$filter服务
- 一般来说Controller中为了减少耦合，Controller是不会相互调用的，控制器之间的交互会通过事件进行

### Model  
ng-model生成数据模型，使用ng-model  

### View

**AngularJs的MVC是借助于$scope实现的**  

### $scope  
- $scope是一个POJO
- $scope提供了一些工具和方法$watch/$apply(用来实时监测对象的变化)
- $scope是一个表达式执行的环境（作用域）
- $scope是一个树形结构，与DOM标签平行
- 子$scope对象会继承父$scope上的属性和方法
- 每一个angular应用只用一个根$scope，一般位于ng-app上
- $scope可以传播事件，类似DOM事件，可以向上也可以向下
- $scope不仅是MVC的基础，也是实现数据双向绑定的基础，所以基本就是AngularJs的基础
- 可以使用angular.element($0).scope()进行调试

**$scope的生命周期**  
创建 -> 注册监控 -> 检测模型变化 -> 观察模型是否脏 -> 销毁  

