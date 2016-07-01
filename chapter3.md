# spark streaming 

## receiver

receiver长时间（可能7*24小时）运行在executor。 每个receiver负责一个 input DStream (例如 一个 读取Kafka消息的input stream)。 每个receiver， 加上input DStream会占用一个core/slot.

注：每个节点的资源会被抽象成若干个slot，由于一个Task占用一个slot，因此slot数目可看成是最多同时运行的Task数目。如果一个Job的Task数目非常多，限于slot数目有限，可能需要运行若干轮。

## input DStream

一个input DStream是一个特殊的DStream， 将Spark Streaming连接到一个外部数据源来读取数据。


## kafka topic

topic是发布消息发布的category 或者 feed名. 对于每个topic, Kafka管理一个分区的log，分区内的消息都是有序不可变的。

## kafka partition

partitions的设计目的有多个.最根本原因是kafka基于文件存储.通过分区,可以将日志内容分散到多个server上,来避免文件尺寸达到单机磁盘的上限,每个partiton都会被当前server(kafka实例)保存;可以将一个topic切分多任意多个partitions(备注:基于sharding),来消息保存/消费的效率.此外越多的partitions意味着可以容纳更多的consumer,有效提升并发消费的能力.


## kafka consumer group

在kafka中，每个消费者要标记自己在那个组中。
如果所有的消费者都在同一个组中，则类似传统的queue消息模式，消息只发给一个消费者。
如果消费者都在不同的组中， 则类似发布-订阅消息模式。 每个消费者都会得到所有的消息。
最通用的模式是混用这两种模式，
关于kafka和消费者线程， 遵循下面的约束：
如果你的消费者读取包含10个分区的 test的topic,

如果你配置你的消费者只使用1个线程， 则它负责读取十个分区
如果你配置你的消费者只使用5个线程， 则每个线程负责读取2个分区
如果你配置你的消费者只使用10个线程， 则每个线程负责读取1个分区
如果你配置你的消费者只使用14个线程， 则10个线程各负责读取1个分区,4个空闲
如果你配置你的消费者只使用8个线程， 则6个线程个负责一个分区，2个线程各负责2个分区


## kafka 并行读取

<!--language:python-->

    numStreams = 5
    kafkaStreams = [KafkaUtils.createStream(...) for _ in range (numStreams)]
    unifiedStream = streamingContext.union(*kafkaStreams)
    unifiedStream.pprint()
