**什么是反射**  

Java反射机制允许程序在运行状态对自身进行检查，或者自审，并且能直接操作程序的内部属性和方法。反射机制是Java被视为动态语言的关键。允许程序执行期通过Reflection APIs取得任何一支名称的类的内部信息，包括package，type parameters，superclass，implemented interfaces，inner classes，outer class,fields,constructors,methods,modifiers,并可以在执行期生成实例，变更field，或者调用方法。  

**Java反射中需要的类**
Java的类反射所需要的类不多，分别是Field，Constructor，Method，Class，Object。