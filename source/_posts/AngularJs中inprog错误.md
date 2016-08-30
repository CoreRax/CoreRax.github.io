title: AngularJs中的inprog错误
---

### 背景 
前几天在修改项目页面的时候遇到了这个问题。在改变页面图标顺序，也就是改变$scope中的一个数组以后，控制台会报这个inprog错误。
       
    Error: $apply already in progress

具体的报错位置是在一个手动调用$apply()函数的地方。

### 相关资料 
首先我自己对于这个问题感性的认识是：“apply 正在进程中”，我们的进程中已经有了一个apply。