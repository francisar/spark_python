# Spark cluster

## worker node

一个工作节点可以执行一个或者多个executor.


## executor

executor就是一个进程， 负责启在一个worker节点上启动应用，运行task执行计算，存储数据到内存或者磁盘上。 每个Spark应用都有自己的executor。一个executor拥有一定数量的cores, 也被叫做“slots”， 可以执行指派给它的task。

## job

一个并行的计算单元，包含多个task。 在执行Spark action (比如 save, collect)产生; 在log中可以看到这个词。

## task

一个task就是一个工作单元， 可以发送给一个executor执行。 它执行你的应用的实际计算的部分工作。 每个task占用父executor的一个slot (core)。

## stage

每个job都被分隔成多个彼此依赖称之为stage的task(类似MapReduce中的map 和 reduce stage);

## 共享变量

普通可序列化的变量复制到远程各个节点。在远程节点上的更新并不会返回到原始节点。因为我们需要共享变量。 Spark提供了两种类型的共享变量。
*  Broadcast 变量。  SparkContext.broadcast(v)通过创建， **只读**。
* Accumulator: 累加器，通过SparkContext.accumulator(v)创建，在任务中只能调用add或者+操作，不能读取值。只有驱动程序才可以读取值。
