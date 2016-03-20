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
RDD支持两种类型的操作:transformations（变换）它从已经存在的RDD创建一个新的数据集；action（活动/行动）它在数据集上做好运算之后返回一个值给驱动程序。比如，map是一个transformation它通过一个函数传递每个数据集元素，并且返回一个新的RDD代表结果。另一方面，reduce是一个action它使用某种方法聚集所有的RDD的元素并且返回一个最后的结果给驱动程序（虽然也有一个并行的ruduceByKey它返回分布式数据集）。  
Spark中所有的transformations是懒惰的/消极的,他们不会立即计算他们的结果。相反他们仅仅记录transformations应用到一些基本的数据集。transformations仅仅在action请求一个结果返回到驱动程序的时候才会计算。这个设计使得Spark运行更加高效，比如我们意识到一个被map创建的数据集将会被用于reduce并且仅仅返回reduce的结果给驱动程序。而不是一个更大的映射数据数据集。//不太明白   
默认的，每一个变换过后的RDD在每一次你运行action的时候可能被重新计算。然而你可能也存留着一个RDD在内存中通过使用persist或者cache方法，在这种情况下Spark将会保留这些元素在节点中以便下一次你查询的时候更加快速的连接。这也支持在硬盘中的持久化的RDD，或者在多个节点中重复的。