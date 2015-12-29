title：Java多线程之间的通信 
--- 

  
很多时候要让多个线程按照一定次序来访问共享资源需要考虑同步问，我们使用synchronized关键字，但是仅仅是synchronized关键字是不够的，虽然synchronized关键字可以阻止并发更新同一个共享资源，实现同步，但是却不能用来实现线程间的通信。  

Java提供了三个非常重要的方法：wait(),notify(),notifyall()。他们都继承于Object对象。

### wait()
调用wait()方法可以使调用该方法的线程释放共享资源的锁，然后从线程退出，进入等待队列，直到被再次唤醒。

### notify()
调用notify()方法可以唤醒等待队列中的第一个等待同一共享资源的的线程，使得线程退出等待队列，进入可运行状态。  

### notify()  
调用notifyall()方法可以唤醒在等待队列中等待同一共享资源的的线程从等待队列中退出，进入可运行状态。


**通过共享对象通信**  
**管道流**  
线程通信使用管道流，管道流有3种形式：        PipedInputStream、PipedOutputStream、PipedReader和PipedWriter以及Pipe.SinkChannel和Pipe.SourceChannel，

它们分别是管道流的字节流、管道字符流和新IO的管道Channel。

管道流通信基本步骤：

        A、使用new操作法来创建管道输入、输出流

        B、使用管道输入流、输出流的connect方法把2个输入、输出流连接起来

        C、将管道输入、输出流分别传入2个线程

        D、2个线程可以分别依赖各自的管道输入流、管道输出流进行通

为什么线程通信的方法wait(), notify()和notifyAll()被定义在Object 类里？

Java的每个对象中都有一个锁(monitor，也可以成为监视器) 并且wait()，notify()等方法用于等待对象的锁或者通知其他线程对象的监视器可用。在Java的线程中并没有可供任何对象使用的锁和同步器。这就是为什么这些方法是Object类的一部分，这样Java的每一个类都有用于线程间通信的基本方法。