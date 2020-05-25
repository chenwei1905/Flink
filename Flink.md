## 数据集类型
1. 无穷数据集：无穷的持续集成的数据集合
2. 有界数据集：有限不会改变的数据集合


## 数据运算模型
1. 流式：只要数据一直在产生，计算就持续地进行
2. 批处理：在预先定义的时间内运行计算，当完成时释放计算机资源
   
## Flink 主要的基石是什么
1. checkPoint: 基于Chandy-Lamport算法，实现分布式一致性快照，提供了一致性的语义
2. state：丰富的State API，valueState，ListState，MapState，BroadcastStatus
3. Time: 实现了Watermark机制，乱序数据处理，迟到数据容忍
4. window 开箱即用的滚动、滑动、会话窗口，以及灵活的自定义窗口

**Time**
1.  Processing Time: 是指事件被处理时机器的系统时间
2.  Event Time 是事件发生的时间，一般就是数据本身携带的时间。这个时间通常是在事件到达 Flink 之前就确定的，并且可以从每个事件中获取到事件时间戳。
3.  Ingestion Time: Ingestion Time 是事件进入 Flink 的时间。 在源操作处，每个事件将源的当前时间作为时间戳，并且基于时间的操作（如时间窗口）会利用这个时间戳。
**windows**
用来对一个无限的流设置一个有限的集合，在有界的数据集上进行操作的一种机制。window 又可以分为基于时间（Time-based）的 window 以及基于数量（Count-based）的 window。
1. `dataStream.keyBy(0).window(TumblingEventTimeWindows.of(Time.seconds(5))); // Last 5 seconds of data`
```java
data.keyBy(1)
    .timeWindow(Time.minutes(1)) //tumbling time window 每分钟统计一次数量和
    .sum(1);

data.keyBy(1)
    .timeWindow(Time.minutes(1), Time.seconds(30)) //sliding time window 每隔 30s 统计过去一分钟的数量和
    .sum(1);
```
## Flink APIS
SQL/Table API(dynamic tables)  
DataStream API(Stream,windows)  
ProcessFunction(events,state,time)

## Flink 数据流编程模型
抽象级别 | 说明
--------|-----
SQL | HIGH-LEVEL Languange
TABLE API| Declaretive DSL
Date stream/ DateSet API | Core APIs
Stateful Stream Processing | Low-level building block(Streams, state, [event] time)


1. 最底层提供了有状态流。它将通过 过程函数（Process Function）嵌入到 DataStream API 中。它允许用户可以自由地处理来自一个或多个流数据的事件，并使用一致、容错的状态。除此之外，用户可以注册事件时间和处理事件回调，从而使程序可以实现复杂的计算。
2. DataStream / DataSet API 是 Flink 提供的核心 API ，DataSet 处理有界的数据集，DataStream 处理有界或者无界的数据流。用户可以通过各种方法（map / flatmap / window / keyby / sum / max / min / avg / join 等）将数据进行转换 / 计算。
**可以在表与 DataStream/DataSet 之间无缝切换，也允许程序将 Table API 与 DataStream 以及 DataSet 混合使用。**
3. Table API 是以 表 为中心的声明式 DSL( Domain Specific Language:领域特定语言)，其中表可能会动态变化（在表达流数据时）。Table API 提供了例如 select、project、join、group-by、aggregate 等操作，使用起来却更加简洁（代码量更少）。
4. Flink 提供的最高层级的抽象是 SQL 。这一层抽象在语法与表达能力上与 Table API 类似，但是是以 SQL查询表达式的形式表现程序。SQL 抽象与 Table API 交互密切，同时 SQL 查询可以直接在 Table API 定义的表上执行。

Fink 程序与数据流结构

```java

DataStream<String> lines = env.addSource(new FlinkKafkaConsumer<>(...))  \\source
DataStream<Event> events = lines.map((line)-> parse(line)); \\Transformation

DataStream<Statistics> stats = events.keyBy("id").timeWindow(Time.seconds(10)).apply(new MyWindowAggregationFunction()); \\Transformation

stats.addSink(new RollingSink(path)); \\sink
```

1. Source: 数据源，Flink 在流处理和批处理上的 source 大概有 4 类：基于本地集合的 source、基于文件的 source、基于网络套接字的 source、自定义的 source。自定义的 source 常见的有 Apache kafka、Amazon Kinesis Streams、RabbitMQ、Twitter Streaming API、Apache NiFi 等，当然你也可以定义自己的 source。

2. Transformation：数据转换的各种操作，有 Map / FlatMap / Filter / KeyBy / Reduce / Fold / Aggregations / Window / WindowAll / Union / Window join / Split / Select / Project 等，操作很多，可以将数据转换计算成你想要的数据。

3. Sink：接收器，Flink 将转换计算后的数据发送的地点 ，你可能需要存储下来，Flink 常见的 Sink 大概有如下几类：写入文件、打印出来、写入 socket 、自定义的 sink 。自定义的 sink 常见的有 Apache kafka、RabbitMQ、MySQL、ElasticSearch、Apache Cassandra、Hadoop FileSystem 等，同理你也可以定义自己的 sink。






