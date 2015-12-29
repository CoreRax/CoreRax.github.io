title：Java面试题总结  
---

### Final,finally,Finalize
Final是一个修饰符，被他修饰变量将不能再被改变或者指向另一个队对象；被他修饰的对象将不能被继承；被他修饰方法不能被重写。

Finally是用在try...catch块之后，用于执行一些不管发生什么情况都要运行的代码，以安全运行程序。

Finalize是一个继承与java.lang.Object的方法。一旦垃圾回收器准备好释放对象占用的内存，将首先调用对象的Finalize方法，并且在下一次垃圾回收时将对象回收，finalize能让你在对象被回收时做一些重要的清理工作。
一般情况下不需要重新覆盖这个方法，因为Object已经有一个默认的实现。但是用Java以外的代码编写的Class（如JNI，C++的new方法分配的内存），垃圾回收机制并不能对这部分对象进行正确的回收，这时就需要覆盖默认的方法来实现对这部分内存的正确释放和回收。比如C++的delete。

### 并发-如何创建一个线程  
有两种方法，1.是实现Runnable接口，当成一个参数传给Thread构造函数；2.是直接继承Thread类。  

### 并发-线程的生命周期  
新建一个线程时它的状态是new，调用start()方法以后由new变为runnable，线程调度器会为Runnable线程池中的线程分配CPU时间，并且将状态改为running，调用wait()方法可以编程waiting状态，blocked和Dead。

### 并发-什么是上下文切换
上下文切换就是线程的切换，包括存储和回复CPU状态的过程。  

### 并发-如何确保main()方法在所在的线程最后结束  
我们可是使用Thread类的join()方法来确保所有的程序创建的线程在main()方法结束退出前结束。  

### 并发-为什么线程通信方法wait(),notify(),notifyall()被定义在Object类里？  
Java的每一个对象中都有一个锁（moniter将，也可以成为监视器）并且wait(),notify()等方法用于等待对象锁，或者通知其他线程对象的监视器可用。在Java的线程中并没有可供任何对象使用的锁和同步器。这就是为什么这些方法是Object类的一部分，这样Java的每一个类都有用于线程间通信的基本方法。  

### 并发- 为什么wait(), notify()和notifyAll ()必须在同步方法或者同步块中被调用？  
当一个线程调用对象的wait()方法时，这个线程必须拥有该对象的锁，接着它就会释放这个对象锁并且进入等待状态直到其他线程调用这个对象的notify()方法。同样的，当一个线程线程调用对象的notify()方法时他会释放这个对象的锁，以便其他等待的线程可以得到这个对象的锁。由于所有这些通信方法都需要线程持有对象的锁，这样就只能通过同步来实现，他们只能在同步方法或者痛不酷爱中被调用。

### 为什么Thread类中的sleep()和yied方法是静态的  
Thread类的sleep()和yield()方法将在当前正在执行的线程上运行。所以在其他处于等待状态的线程调用这些方法是没有意义的。他们可以在当前正在执行的线程中工作，并避免程序员错误的认为可以在其他非运行线程调用这些方法。

### volatile关键字有什么作用  
确保不同线程中读取到的变量是一样的。

### 同步块和同步方法哪一个是更好的选择  
同步块是更好的选择，因为它不会锁住整个对象。 

### 什么是ThreadLocal  
ThreadLocal用于创建线程的本地变量，我们知道一个对象的所有线程会共享他的全局变量，所以这些变量不是线程安全的，我们可以使用同步技术。但是当我们不想使用同步技术的时候，我们可以选择ThreadLocal变量。每个线程都会拥有他们自己的Thread变量，他们可以使用get/set方法来获取他们的默认值或者在线程内部改变他们的值。  

### 什么是Java Timer类？如何创建一个特定时间间隔的任务？
java.util.Timer是一个工具类，可以用于安排一个线程在未来的某个特定时间执行。Timer类可以用安排一次性任务或者周期任务。
java.util.TimerTask是一个实现了Runnable接口的抽象类，我们需要去继承这个类来创建我们自己的定时任务并使用Timer去安排它的执行。  

### 什么是线程池？如何创建线程池？
一个线程池管理了一组工作线程，同时它还包括一个用于放置等待执行的任务的队列。
java.util.concurrent.Executors提供了一个 java.util.concurrent.Executor接口的实现用于创建线程池。  

### 接口和抽象类的区别  


- 接口中所有的方法都是抽象的，类可以同时包括抽象和非抽象方法
- 类可以实现很多个接口，但是只能继承一个抽象类
- 类如果要实现一个接口，它必须要实现接口声明的所有方法。但是，类可以不实现抽象类声明的所有方法，当然，在这种情况下，类也必须得声明成是抽象的
- Java接口中声明的变量默认都是final的。抽象类可以包含非final的变量。Java接口中的成员函数默认是public的。抽象类的成员函数可以是private，protected或者是public。
- 接口是绝对抽象的，不可以被实例化。抽象类也不可以被实例化，但是，如果它包含main方法的话是可以被调用的。  

###什么是值传递，什么是引用传递  
对象被值传递，意味着传递了对象的一个副本。因此，就算是改变了对象副本，也不会影响源对象的值。
对象呗引用传递，意味着传递的并不是实际对象，而是对象的引用。因为，外部对引用对象的改变都会反映到源对象上。  

### Java集合类框架的基本接口有哪些
- Collection：代表一组对象，每一个对象都是他的子元素。
- Set：不包含重复元素的Collection。
- List：可以有重复元素的的有序Collection。
- Map：可以把Key映射到value的对象，key不可重复。 

### 为什么集合类没有实现Cloneable和Serializable接口
集合类接口定义了一组叫做元素的对象，集合类接口的每一种具体的实现类都可以选择它自己的方式对元素进行保存和排序，有的集合类允许重复的键，有的不允许。

### 什么是迭代器
Iterator接口提供了很多对于集合元素进行迭代的方法，每个集合类都包含了可以返回迭代器实例的迭代方法，迭代器可以在迭代的过程中删除底层集合的元素。

### Itertor和ListIterator的区别  
- Iterator可以遍历Set和List集合，但是ListIterator只能用来遍历List。
- Iterator只能向后遍历，ListIterator既可以前向也可以后向遍历。
- LstIterator实现了Iterator接口，并包含其他的功能，比如：增加元素，替换元素，获取前一个和后一个元素的索引，等等。  

### Java中的HashMap的工作原理是什么 
Java中HashMap以键值对的形式来存储元素，HashMap需要一个hash函数，它使用hashcode()和equals()方法来向集合/从集合添加和检索元素。当调用put()方法时HashMap会计算key的hash值然后把键值对存储在集合中的合适的索引上。如果key已经存在，value的值会被更新。

### HashMap和HashTable有什么区别
Java中HashTable是jdk 1的时代留下来的产物。  
- HashMap允许键和值是null，而hashTable不允许。
- HashTable是同步的，而HashMap不是，所以HashMap更加适合单线程环境。

### 数组(Arrays)和列表(ArrayList)有什么区别？什么时候应该使用Arrays而不是ArrayList？
- 数组的大小是固定的，ArrayList的大小是不固定的。
- ArrayList提供了更多的方法和特性，比如：addAll()，removeAll()，iterator()等。
- 对于基本类型数据，集合使用自动装箱来减少编码工作量。但是，当处理固定大小的基本数据类型的时候，这种方式相对比较慢。

### ArrayList和LinkedList的区别  
ArrayList和LinkedList都实现了List接口，但是不同点在于：   

- ArrayList的底层实现是数组，LinkedList的底层实现是链表，因此在对于元素的随机访问时ArrayList的效率更高。  
- LinkedList的插入添加删除操作更快。
- LinkedList更加占内存，因为每个节点都有两个引用。  

### Comparable和Comparator接口是干什么的？列出它们的区别。  
Java提供了只包含一个compareTo()方法的Comparable接口，这个方法可以给两个对象排序。具体来说，它返回负数，0，正数，表示输入对象小于，等于，大于已经存在的对象。

Java提供了包含compare()和equals()两个方法的Comparator接口，compare()方法用来给两个输入参数排序，返回负数，0，正数表明第一个参数是小于，等于，大于第二个参数。equals()方法需要一个对象作参数，它用来决定输入参数是否和comparator相同。只有当输入参数也是一个comparator并且输入参数和当前comparator的排序结果是相同的时候，这个方法才返回true。

### HashSet和TreeSet的区别
HashSet是用Hash表来实现的，因此他的元素是无序的。add()，remove()，contains()方法的时间复杂度是O(1)。  
另一方面，TreeSet是由一个树形的结构来实现的，它里面的元素是有序的。因此，add()，remove()，contains()方法的时间复杂度是O(logn)。