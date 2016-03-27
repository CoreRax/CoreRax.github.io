考虑如下问题：   
如果一个intent请求在一片数据上执行一个动作，Android如何知道哪一个应用程序或者组件能来响应这个请求？   


intent-filter就是用来注册Activity，service和Broadcast Receiver具有能在某一数据上执行某种操作的能力。使用intent Filter,应用程序组件告诉Android，它们能为其他程序组件的动作请求提供服务，包括同一程序的组件，本地的或者第三方的应用程序。  

为了注册一个应用程序组件为intent处理者，在组件的manifest节点添加一个intent-filter标签。  

在intent Filter节点里使用下面的标签（关联属性），你能指定组件支持的动作(action),种类(category),数据(data):  
1.动作测试：  
<intent-filter>中可以包括子元素<action>。至少包括一个<action>元素。  
2.类别测试：  
<intent-filter>元素可以包含<category>子元素。只有当Intent请求中所有的Category与组件中某一个IntentFilter的<category>完全匹配时，才会让该 Intent请求通过测试
，IntentFilter中多余的<category>声明并不会导致匹配失败。一个没有指定任何类别测试的 IntentFilter仅仅只会匹配没
有设置类别的Intent请求。   
3.数据测试：  
<data>元素指定了希望接受的Intent请求的数据URI和数据类型，URI被分成三部分来进行匹配：scheme、 authority和path
。其中，用setData()设定的Inteat请求的URI数据类型和scheme必须与IntentFilter中所指定的一致。若IntentFilter中还指定了authority或path，它们也需要相匹配才会通过测试。