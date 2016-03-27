**什么是spark**  
是一个大数据处理框架，以速度，易用性，复杂性分析为目的构建的。是Apache的开源项目之一。  
**优势**  
相比其他大数据框架相比优势如下  


**外部数据集**  
Spark可以在任何Hadoop支持的存储源里创建分布式数据集，这些存储源包括本地文件系统，HDFS，Cassandra, HBase, Amazon S3等。Spark支持文本文件，序列文件，和其他Hadoop支持的输入格式。

文本类型弹性分布式数据集可以使用SparkContext中的textFile方法创建。这个方法把一个URI当成一个文件，并且当成一个行的集合来读取。  
    
    JavaRDD<String> distFile = sc.textFile("data.txt");  
一旦创建分布式数据集，distFile对象就可以被用来做一些数据集操作。比如我们可以使用map和reduce来计算所有行的长度之和：
    
    distFile.map(s->s.length()).reduce((a,b) ->a+b)  
 Spark读取文件注意事项：  

- 如果你使用的是在本地文件系统上的路径，文件必须是在工作节点上相同的路径并且可以访问的。不管是复制到所有工作节点还是使用网络共享文件系统。  
- Spark的所有基于文件的输入方法，包括textFile，支持运行在目录上，压缩文件上，还有通配符。比如可以使用textFile("/my/directory"), textFile("/my/directory/*.txt"), and textFile("/my/directory/*.gz")   
- textFile方法也有一个可选的第二参数老控制文件被分为多少部分。默认的Spark为每一块文件创建一个分区（块在HDFS中默认是64MB）,可以请求更大数目的分区，但不能少于块数。  

除了文本文件格式，Spark的Java API也支持其他几个数据格式：   

- JavaSparkContext.wholeTextFiles允许你读取一个包含多种文本格式的目录，然后将这些文件按（文件名，内容）对的形式返回。这和textFile在每一个文件中返回每一行的一个记录相反。  
- 对于流式文件，使用SparkContext的sequenceFile[k,v]方法。其中k和v分别是类型的键和文件的值。这些应该继承Hadoop的Writable接口，比如intWritable和Text。除此之外，Spark允许你为几个常用的Writable接口本地指定类型，比如sequenceFile[int, String]将自动的读取IntWritables 和 Texts。
- 对于其他的Hadoop输入类型，你可以使用SparkContext.HadoopRDD方法，它可以获取任意的JobConf，输入类型类，键类和值类。你也可以使用SparkContext.newAPIHadoopRDD来支持基于新的MapReduce API的输入格式。
- RDD.saveAsObjectFile和SparkContext.objectFile支持将一个RDD存储为一个简单的包含序列化的Java对象的格式。虽然这对于一些特别的格式比如Avro效率不高，但是提供了一个简单的方法来存储RDD。


**RDD操作**  
RDD支持两种类型的操作:transformations（变换）它从已经存在的RDD创建一个新的数据集；action（活动/行动）它在数据集上做好运算之后返回一个值给驱动程序。比如，map是一个transformation，它通过一个函数传递每个数据集元素，并且返回一个新的RDD代表结果。另一方面，reduce是一个action它使用某种方法聚集所有的RDD的元素并且返回一个最后的结果给驱动程序（虽然也有一个并行的ruduceByKey它返回分布式数据集）。  
Spark中所有的transformations是懒惰的/消极的,他们不会立即计算他们的结果。相反他们仅仅记录transformations应用到一些基本的数据集。transformations仅仅在action请求一个结果返回到驱动程序的时候才会计算。这个设计使得Spark运行更加高效，比如我们意识到一个被map创建的数据集将会被用于reduce并且仅仅返回reduce的结果给驱动程序。而不是一个更大的映射数据数据集。//不太明白   
默认的，每一个变换过后的RDD在每一次你运行action的时候可能被重新计算。然而你可能也存留着一个RDD在内存中通过使用persist或者cache方法，在这种情况下Spark将会保留这些元素在节点中以便下一次你查询的时候更加快速的连接。这也支持在硬盘中的持久化的RDD，或者在多个节点中重复的。  

**基础**  
为了举例说明RDD的基本知识，考虑下列简单的程序：  
    
        JavaRDD<String> lines = sc.textFile("data.txt");
        JavaRdd<String> lineLengths =  lines.map(s->s.length());
        int totalLength = lineLengths.reduce((a,b) -> a+b);
第一行从一个外部文件中定义了一个基本的RDD(弹性分布式数据集)。这个数据集现在还没有加载到内存，或者开始处理：lines仅仅是一个指向文件的指针。第二行定义了lineLengths作为一个map transformation的结果。同样的，lineLengths并没有被立即运算，因为它是懒惰性质的。最后运行reduce，它是一个action。这时候Spark将计算任务打散成task来在不同的机器上运行。每个机器运行它那一部分map和本地reduction，返回自己的结果到驱动程序。  
如果之后还需要使用lineLengths，我们可以添加：  

    lineLengths.persist(StorageLevel.MEMORY_ONLY()); 
在reduce之前，这可以使得lineLength在第一次计算之前就存储到内存中。


**传递Functions给Spark**  
Spark的API很大程度依赖于传递Functions到驱动程序来运行集群。在Java中，Functions由实现org.apache.spark.api.java.function 包中接口的类代表。有两种方法来创建这些functions：   

- 在你的类中实现Function接口，或者作为一个匿名内部类或者有名字的类传递一个实例到Spark。
- 在Java8 中使用lambda表达式来简洁地定义一个实现。
当本教程为了简洁大部分使用lambda语法，使用相同的API就很简单。比如我们可以把上面的代码写成：

         JavaRDD<String> lines = sc.textFile("data.txt");
         JavaRDD<String> lineLengths = lines.map(new Function<Stirng, Integer>(){
         public Integer call(String s){ return s.length();}
         });
         int totalLength = lineLength.reduce(new Function2<Integer, Integer, Integer>() {
         public Integer call(Integer a, Integer b){return a+b;}
        });

或者，如果觉得吧function写为内联是笨拙的做法：
    
    Class GetLength implements Function<String, Integer> {
         public Integer call(String s) { return s.length(); }
    }
    class Sum implements Function2<Integer, Integer, Integer> {
    public Integer call(Integer a, Integer b) { return a + b; }
    }

    JavaRDD<String> lines = sc.textFile("data.txt");
    JavaRDD<Integer> lineLengths = lines.map(new GetLength());
    int totalLength = lineLengths.reduce(new Sum());

记住匿名内部类在Java中也可以访问闭合区间的变量，只要这些变量是final的。Spark传递这些变量的拷贝给每一个节点。  

**理解闭包**  
Spark中比较困难的事是当在集群之间运行代码时理解变量和方法的作用域和生命期。RDD操作很多是在变量的作用域之外改变变量的值，这是一个造成混淆的主要原因。在下面的例子中我们将看到代码使用foreach()来增加计数器，但是相同的问题发生在其他操作中。  
####### 举例 

假设一个RDD元素加和，结果将表现得很不同，取决于运行的在相同的JVM上。一个普通的例子是当在local模式运行Spark和将Spark应用发布到集群上运行。  


     int counter = 0;
     JavaRDD<Integer> rdd = sc.parallelize(data);
     
     //Wrong:Don't do this!
     rdd.foreach(x->counter +=x);
     println("Counter value: " + counter);

####### Local vs. cluster modes 
上面代码的表现是无法定义的，可能无法按照预期的运行。为了运行任务，将RDD操作的进程打散成任务，每一个任务都被一个执行者执行。在执行之前，Spark计算任务的**闭包**。闭包是可以被执行RDD上计算任务的执行者可见的变量和方法。这个闭包是序列化的兵器可以送到每一个执行者。  

在闭包中被送到每一个执行者的变量现在是拷贝，因此，当counter被foreach()引用的时候，已经不再只是驱动节点上的counter了。这里任然有一个counter在驱动节点的内存中，但是已经不再对执行者可见了。执行者只能看到序列化闭包中的拷贝。因此，counter最后的值将仍旧是0因为所有的对counter的操作只能影响到序列化闭包中。  

在本地模式中，在一些情况下foreach() function会实际上执行在相同的JVM上并且会引用相同的原始counter，可能会更新它。  

为了确保这些一系列行为被完好定义，应该使用一个Accumulator(累加器)。累加器在Spark中被明确地用来提供一种安全的机制来更新一个变量当执行过程被划分到一个集群中不同的工作节点时。在累加器部分会详细讨论。

一般来说