title: Java-StringBuffer
---
### Class StringBuffer
继承于java.lang.Object  
已实现的接口：Serializable,Appendable,CharSequence  
是一个线程安全，可变的字符数组。StringBuffer就像一个String，但是可以被修改。  
该类的方法在必要的情况下是同步的，因此，对于任何特殊实例的操作都好像以调用顺序序列的方式发生的。  
StringBuffer中最主要的操作是添加和插入方法，许多方法都被重写以接收各种类型的数据。他们有效地转换给定的数据为String并且添加或者插入到StringBuffer中。  
当操作发生与源序列有关（比如增加或者插入），该类只在StringBuffer缓冲区中同步而不是在源序列中同步。  
每一个StringBuffer都有容量，只要StringBuffer中的字符序列的长度不超过容量，就没有必要分配新的内部buffer数组，如果内部buffer溢出了，将会自动的增长。  
从JDK5开始，StringBuffer被补充了一个单线程的等价类，StringBuilder，通常应该优先使用StringBuilder，因为不需要同步，所以速度更快。

### Interface Collection<E>  
父接口：Iterable<E>  
子接口：BeanContext, BeanContextServices, BlockingDeque<E>, BlockingQueue<E>, Deque<E>, List<E>, NavigableSet<E>, Queue<E>, Set<E>, SortedSet<E>, TransferQueue<E>  
所有已知实现类：AbstractCollection, AbstractList, AbstractQueue, AbstractSequentialList, AbstractSet, ArrayBlockingQueue, ArrayDeque, ArrayList, AttributeList, BeanContextServicesSupport, BeanContextSupport, ConcurrentHashMap.KeySetView, ConcurrentLinkedDeque, ConcurrentLinkedQueue, ConcurrentSkipListSet, CopyOnWriteArrayList, CopyOnWriteArraySet, DelayQueue, EnumSet, HashSet, JobStateReasons, LinkedBlockingDeque, LinkedBlockingQueue, LinkedHashSet, LinkedList, LinkedTransferQueue, PriorityBlockingQueue, PriorityQueue, RoleList, RoleUnresolvedList, Stack, SynchronousQueue, TreeSet, Vector  

该类是Collection层次结构中的根接口。一个collection代表一组对象，这些对象也成为collection的元素。一些collection允许重复的元素，而另外一些不允许。一些是有序的一些是无序的。JDK不提供这个任何这个接口的直接实现：它提供更多子接口比如List，Set之类的实现。此接口用来通常用来传递collection，并且在需要最大普遍性的地方操作这些collection。  
所有通用的collection实现类（通常是通过它的一个子接口间接实现collection）应该提供两个标准构造方法：一个是void构造方法，用于创建空collection；另一个是带有collection类型单参数的构造方法，用于创建一个具有与其参数相同元素的新的collection。实际上，后者允许用户复制任何collection，以生成所需实现类型的一个等效collection。尽管无法强制执行此约定，但是Java平台库中所有通用的collection都遵循它。  
破坏性方法：是指可修改其所操作的collection的那些方法，如果此collection不支持该操作，则指定这些方法抛出UnsupportedOperationException。如果是这样，那么在调用对该collection无效时，这些方法可能，但并不一定抛出UnsupportedOperationException。  

###class Collections  
继承自java.util.Collection  
此类完全由在collection上进行操作或返回collection的静态方法组成。它包含在collection上操作的多态算法，即“包装器”，包装器返回由指定collection支持的新collection以及少数其他内容。  
如果为此类的方法提供的collection或类对象为null，则这些方法都将抛出NullPointerException。  
