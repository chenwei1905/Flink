## [数据集转换](https://github.com/chenwei1905/Flink)

## map
	
采用一个数据元并生成一个数据元。
```java
data.map(new MapFunction<String, Integer>() {
  public Integer map(String value) { return Integer.parseInt(value); }
});
```
## FlatMap
采用一个数据元并生成零个，一个或多个数据元。
```java
data.flatMap(new FlatMapFunction<String, String>() {
  public void flatMap(String value, Collector<String> out) {
    for (String s : value.split(" ")) {
      out.collect(s);
    }
  }
});

```

## MapPartition 

在单个函数调用中转换并行分区。该函数将分区作为Iterable流来获取，并且可以生成任意数量的结果值。每个分区中的数据元数量取决于并行度和先前的 算子操作。
```java
data.mapPartition(new MapPartitionFunction<String, Long>() {
  public void mapPartition(Iterable<String> values, Collector<Long> out) {
    long c = 0;
    for (String s : values) {
      c++;
    }
    out.collect(c);
  }
});
```

## Filter

	
计算每个数据元的布尔函数，并保存函数返回true的数据元。
重要信息：系统假定该函数不会修改应用谓词的数据元。违反此假设可能会导致错误的结果。

```java

data.filter(new FilterFunction<Integer>() {
  public boolean filter(Integer value) { return value > 1000; }
});


```

## Reduce
通过将两个数据元重复组合成一个数据元，将一组数据元组合成一个数据元。Reduce可以应用于完整数据集或分组数据集。
```java
data.reduce(new ReduceFunction<Integer> {
  public Integer reduce(Integer a, Integer b) { return a + b; }
});
```
如果将reduce应用于分组数据集，则可以通过提供CombineHintto 来指定运行时执行reduce的组合阶段的方式 setCombineHint。在大多数情况下，基于散列的策略应该更快，特别是如果不同键的数量与输入数据元的数量相比较小（例如1/10）。

## ReduceGroup
将一组数据元组合成一个或多个数据元。ReduceGroup可以应用于完整数据集或分组数据集。
```java
data.reduceGroup(new GroupReduceFunction<Integer, Integer> {
  public void reduce(Iterable<Integer> values, Collector<Integer> out) {
    int prefixSum = 0;
    for (Integer i : values) {
      prefixSum += i;
      out.collect(prefixSum);
    }
  }
});
```

## Aggregate
将一组值聚合为单个值。聚合函数可以被认为是内置的reduce函数。聚合可以应用于完整数据集或分组数据集。
```java
Dataset<Tuple3<Integer, String, Double>> input = // [...]
DataSet<Tuple3<Integer, String, Double>> output = input.aggregate(SUM, 0).and(MIN, 2);
```
您还可以使用简写语法进行最小，最大和总和聚合。
```java
	Dataset<Tuple3<Integer, String, Double>> input = // [...]
DataSet<Tuple3<Integer, String, Double>> output = input.sum(0).andMin(2);
```

## Distinct
返回数据集的不同数据元。它相对于数据元的所有字段或字段子集从输入DataSet中删除重复条目。
```java
data.distinct();
```
使用reduce函数实现Distinct。您可以通过提供CombineHintto 来指定运行时执行reduce的组合阶段的方式 setCombineHint。在大多数情况下，基于散列的策略应该更快，特别是如果不同键的数量与输入数据元的数量相比较小（例如1/10）。



## join

通过创建在其键上相等的所有数据元对来连接两个数据集。可选地使用JoinFunction将数据元对转换为单个数据元，或使用FlatJoinFunction将数据元对转换为任意多个（包括无）数据元。请参阅键部分以了解如何定义连接键。
```java
result = input1.join(input2)
               .where(0)       // key of the first input (tuple field 0)
               .equalTo(1);    // key of the second input (tuple field 1)
```
您可以通过Join Hints指定运行时执行连接的方式。提示描述了通过分区或广播进行连接，以及它是使用基于排序还是基于散列的算法。有关可能的提示和示例的列表，请参阅“ 转换指南”。
如果未指定提示，系统将尝试估算输入大小，并根据这些估计选择最佳策略。
```java
// This executes a join by broadcasting the first data set
// using a hash table for the broadcast data
result = input1.join(input2, JoinHint.BROADCAST_HASH_FIRST)
               .where(0).equalTo(1);
```
请注意，连接转换仅适用于等连接。其他连接类型需要使用OuterJoin或CoGroup表示。


## OuterJoin
在两个数据集上执行左，右或全外连接。外连接类似于常规（内部）连接，并创建在其键上相等的所有数据元对。此外，如果在另一侧没有找到匹配的Keys，则保存“外部”侧（左侧，右侧或两者都满）的记录。匹配数据元对（或一个数据元和null另一个输入的值）被赋予JoinFunction以将数据元对转换为单个数据元，或者转换为FlatJoinFunction以将数据元对转换为任意多个（包括无）数据元。请参阅键部分以了解如何定义连接键。
```java
input1.leftOuterJoin(input2) // rightOuterJoin or fullOuterJoin for right or full outer joins
      .where(0)              // key of the first input (tuple field 0)
      .equalTo(1)            // key of the second input (tuple field 1)
      .with(new JoinFunction<String, String, String>() {
          public String join(String v1, String v2) {
             // NOTE:
             // - v2 might be null for leftOuterJoin
             // - v1 might be null for rightOuterJoin
             // - v1 OR v2 might be null for fullOuterJoin
          }
      });
```


## CoGroup

reduce 算子操作的二维变体。将一个或多个字段上的每个输入分组，然后关联组。每对组调用转换函数。请参阅keys部分以了解如何定义coGroup键。
```java
data1.coGroup(data2)
     .where(0)
     .equalTo(1)
     .with(new CoGroupFunction<String, String, String>() {
         public void coGroup(Iterable<String> in1, Iterable<String> in2, Collector<String> out) {
           out.collect(...);
         }
      });
```

## Cross
	
构建两个输入的笛卡尔积（交叉乘积），创建所有数据元对。可选择使用CrossFunction将数据元对转换为单个数据元
```java
DataSet<Integer> data1 = // [...]
DataSet<String> data2 = // [...]
DataSet<Tuple2<Integer, String>> result = data1.cross(data2);
```
注：交叉是一个潜在的非常计算密集型 算子操作它甚至可以挑战大的计算集群！建议使用crossWithTiny（）和crossWithHuge（）来提示系统的DataSet大小。


## Union

生成两个数据集的并集。
```java
DataSet<String> data1 = // [...]
DataSet<String> data2 = // [...]
DataSet<String> result = data1.union(data2);
```

## Rebalance

均匀地Rebalance 数据集的并行分区以消除数据偏差。只有类似Map的转换可能会遵循Rebalance 转换。
```java
DataSet<String> in = // [...]
DataSet<String> result = in.rebalance()
                           .map(new Mapper());
```

## Hash-Partition
散列分区给定键上的数据集。键可以指定为位置键，表达键和键选择器函数。
```java
DataSet<Tuple2<String,Integer>> in = // [...]
DataSet<Integer> result = in.partitionByHash(0)
                            .mapPartition(new PartitionMapper());
```


## Range-Partition

	
Range-Partition给定键上的数据集。键可以指定为位置键，表达键和键选择器函数。
```java
DataSet<Tuple2<String,Integer>> in = // [...]
DataSet<Integer> result = in.partitionByRange(0)
                            .mapPartition(new PartitionMapper());
```

## Custom Partitioning

手动指定数据分区。
注意：此方法仅适用于单个字段键。
```java
DataSet<Tuple2<String,Integer>> in = // [...]
DataSet<Integer> result = in.partitionCustom(Partitioner<K> partitioner, key)
```

## Sort Partition

本地按指定顺序对指定字段上的数据集的所有分区进行排序。可以将字段指定为元组位置或字段表达式。通过链接sortPartition（）调用来完成对多个字段的排序。
```java
DataSet<Tuple2<String,Integer>> in = // [...]
DataSet<Integer> result = in.sortPartition(1, Order.ASCENDING)
                            .mapPartition(new PartitionMapper());
```

## First-n
返回数据集的前n个（任意）数据元。First-n可以应用于常规数据集，分组数据集或分组排序数据集。分组键可以指定为键选择器函数或字段位置键。
```java
DataSet<Tuple2<String,Integer>> in = // [...]
// regular data set
DataSet<Tuple2<String,Integer>> result1 = in.first(3);
// grouped data set
DataSet<Tuple2<String,Integer>> result2 = in.groupBy(0)
                                            .first(3);
// grouped-sorted data set
DataSet<Tuple2<String,Integer>> result3 = in.groupBy(0)
                                            .sortGroup(1, Order.ASCENDING)
                                            .first(3);
```

### 以下转换可以用于元组的数据集
## project
	
从元组中选择字段的子集
```java
DataSet<Tuple3<Integer, Double, String>> in = // [...]
DataSet<Tuple2<String, Integer>> out = in.project(2,0);
```


## MinBy / MaxBy

从一组元组中选择一个元组，其元组的一个或多个字段的值最小（最大）。用于比较的字段必须是有效的关键字段，即可比较。如果多个元组具有最小（最大）字段值，则返回这些元组的任意元组。MinBy（MaxBy）可以应用于完整数据集或分组数据集。
```java
DataSet<Tuple3<Integer, Double, String>> in = // [...]
// a DataSet with a single tuple with minimum values for the Integer and String fields.
DataSet<Tuple3<Integer, Double, String>> out = in.minBy(0, 2);
// a DataSet with one tuple for each group with the minimum value for the Double field.
DataSet<Tuple3<Integer, Double, String>> out2 = in.groupBy(2)
                                                  .minBy(1);

```


