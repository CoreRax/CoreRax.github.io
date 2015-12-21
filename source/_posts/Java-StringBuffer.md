###Class StringBuffer
继承于java.lang.Object  
已实现的接口：Serializable,Appendable,CharSequence  
是一个线程安全，可变的字符数组。StringBuffer就像一个String，但是可以被修改。  
该类的方法在必要的情况下是同步的，因此，对于任何特殊实例的操作都好像以调用顺序序列的方式发生的。  
StringBuffer中最主要的操作是添加和插入方法，许多方法都被重写以接收各种类型的数据。他们有效地转换给定的数据为String并且添加或者插入到StringBuffer中。  
当操作发生与源序列有关（比如增加或者插入），该类只在StringBuffer缓冲区中同步而不是在源序列中同步。  
每一个StringBuffer都有容量，只要StringBuffer中的字符序列的长度不超过容量，就没有必要分配新的内部buffer数组，如果内部buffer溢出了，将会自动的增长。  
从JDK5开始，StringBuffer被补充了一个单线程的等价类，StringBuilder，通常应该优先使用StringBuilder，因为不需要同步，所以速度更快。