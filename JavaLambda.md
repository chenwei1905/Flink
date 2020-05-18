## Java Lambda 表达式

**Flink支持对Java API的所有 算子使用lambda表达式，但是，每当lambda表达式使用Java泛型时，您需要显式声明类型信息。本文档介绍如何使用lambda表达式并描述当前的限制。**


### 一个简单的内联map()函数
函数的输入i和输出参数的类型map()不需要声明，因为它们是由Java编译器推断的。
```java

  final ExecutionEnvironment env = ExecutionEnvironment.getExecutionEnvironment();
  List<Integer> list = env.fromElements(1, 2, 3).map(item-> item*item).collect();
  System.out.println(list);
  //[1,4,9]

```

### 需要编写带有类型签名的函数
```java

DataSet<String> text = env.readTextFile("D:///个人文档整理/测试数据/flink.txt");
List<Person> list = text.map(item -> new Person(item.split("\\s+")[0], item.split("\\s+")[1])).returns(Person.class).collect();

System.out.println(list);


```
在这种情况下需要显式指定类型信息，否则输出将被视为Object导致无效序列化的类型。
