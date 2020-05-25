## Flink Data Sink

Data Sink 是把数据存储下来的意思


### Data Sink的存储方式

主要有Kafka、ElasticSearch、Socket、RabbitMQ、JDBC、Cassandra POJO、File、Print 等 Sink 的方式。


### 自定义Sink的方式
自带的Sink可以看到都是继承了RichSinkFunction抽象类，实现了其中的方法，那么如果要定义自己的sink的话其实也是需要继承这个类，需要实现invoke方法;


