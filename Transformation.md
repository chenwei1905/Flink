## map
这是最简单的转换之一，其中输入是一个数据流，输出的也是一个数据流
## FlatMap

## Filter


## KeyBy

KeyBy 在逻辑上是基于 key 对流进行分区。在内部，它使用 hash 函数对流进行分区。它返回 KeyedDataStream 数据流。

```java 
KeyedStream<Student, Integer> keyBy = student.keyBy(new KeySelector<Student, Integer>() {
    @Override
    public Integer getKey(Student value) throws Exception {
        return value.age;
    }
});
keyBy.print();
```

## Reduce

## Fold
Fold 通过将最后一个文件夹流与当前记录组合来推出 KeyedStream。 它会发回数据流。

```java
KeyedStream.fold("1", new FoldFunction<Integer, String>() {
    @Override
    public String fold(String accumulator, Integer value) throws Exception {
        return accumulator + "=" + value;
    }
})


```

## Aggregations


## Window
Window 函数允许按时间或其他条件对现有 KeyedStream 进行分组。 以下是以 10 秒的时间窗口聚合：

```java

inputStream.keyBy(0).window(Time.seconds(10));
```

Flink 定义数据片段以便（可能）处理无限数据流。 这些切片称为窗口。 此切片有助于通过应用转换处理数据块。 要对流进行窗口化，我们需要分配一个可以进行分发的键和一个描述要对窗口化流执行哪些转换的函数

要将流切片到窗口，我们可以使用 Flink 自带的窗口分配器。 我们有选项，如 tumbling windows, sliding windows, global 和 session windows。 Flink 还允许您通过扩展 WindowAssginer 类来编写自定义窗口分配器。 这里先预留下篇文章来讲解这些不同的 windows 是如何工作的。
## WindowAll

## Union

## Project




