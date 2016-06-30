# spark 基本概念


## RDD

英文名：Resilient Distributed Dataset
中文名：弹性分布式数据集

spark 中数据计算的基本单位是RDD

RDD作为数据结构，本质上是一个只读的分区记录集合。一个RDD可以包含多个分区，每个分区就是一个dataset片段。RDD可以相互依赖。如果RDD的每个分区最多只能被一个Child RDD的一个分区使用，则称之为narrow dependency；若多个Child RDD分区都可以依赖，则称之为wide dependency。不同的操作依据其特性，可能会产生不同的依赖。例如map操作会产生narrow dependency，而join操作则产生wide dependency。

针对RDD主要有两种类型的操作：Transformation 和Actions

## Transformation

RRD通过该操作产生的新的RDD时不会立即执行，只有等到Action操作才会真正执行。


map,filter,flatMap,sample,union,groupByKey,reduceByKey,join,groupWith,cartesian,sort等属于Transformation操作


## Actions

提交Spark作业，当Action时，Transformation类型的操作才会真正执行计算操作，然后产生最终结果输出。
count,collect,reduce,lookup,save等属于action操作。


## 缓存

Spark通过useDisk、useMemory、deserialized、replication4个参数组成11种缓存策略。

<!--language:scala-->

    class StorageLevel private(
    　　private var useDisk_ : Boolean,
    　　private var useMemory_ : Boolean,
    　　private var useOffHeap_ : Boolean,
    　　private var deserialized_ : Boolean,
    　　private var replication_ : Int = 1
    )


    object StorageLevel {
    　　val NONE = new StorageLevel(false, false, false, false)
    　　val DISK_ONLY = new StorageLevel(true, false, false, false)
    　　val DISK_ONLY_2 = new StorageLevel(true, false, false, false, 2)
    　　val MEMORY_ONLY = new StorageLevel(false, true, false, true)
    　　val MEMORY_ONLY_2 = new StorageLevel(false, true, false, true, 2)
    　　val MEMORY_ONLY_SER = new StorageLevel(false, true, false, false)
    　　val MEMORY_ONLY_SER_2 = new StorageLevel(false, true, false, false, 2)
    　　val MEMORY_AND_DISK = new StorageLevel(true, true, false, true)
    　　val MEMORY_AND_DISK_2 = new StorageLevel(true, true, false, true, 2)
    　　val MEMORY_AND_DISK_SER = new StorageLevel(true, true, false, false)
    　　val MEMORY_AND_DISK_SER_2 = new StorageLevel(true, true, false, false, 2)
    　　val OFF_HEAP = new StorageLevel(false, false, true, false)

    }



可选用的存储级别有如下：

* MEMORY_ONLY:将RDD 作为反序列化的的对象存储JVM 中。如果RDD不能被内存装下，一些分区将不会被缓存，并且在需要的时候被重新计算。
这是是默认的级别
* MEMORY_AND_DISK:将RDD 作为反序列化的的对象存储在JVM 中。如果RDD不能被与内存装下，超出的分区将被保存在硬盘上，并且在需要时被读取
* MEMORY_ONLY_SER: 将RDD 作为序列化的的对象进行存储（每一分区占用一个字节数组）。
通常来说，这比将对象反序列化的空间利用率更高，尤其当使用fast serializer,但在读取时会比较占用CPU
* MEMORY_AND_DISK_SER: 与MEMORY_ONLY_SER 相似，但是把超出内存的分区将存储在硬盘上而不是在每次需要的时候重新计算
* DISK_ONLY:只将RDD 分区存储在硬盘上
* DISK_ONLY_2:与上述的存储级别一样，但是将每一个分区都复制到两个集群结点上

## Shuffle

Shuffle指的是从map阶段到reduce阶段转换的时候，即map的output向着reduce的input映射的时候，并非节点一一对应的，即干map工作的slave A，它的输出可能要分散跑到reduce节点A、B、C、D …… X、Y、Z去，就好像shuffle的字面意思“洗牌”一样，这些map的输出数据要打散然后根据新的路由算法(比如对key进行某种hash算法)，发送到不同的reduce节点上去。
