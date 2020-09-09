#### java.util和java.lang包下常用API

#### [java8 stream 常用API](https://blog.csdn.net/qq_37781649/article/details/103258397?utm_source=app)

 ![img](https://img-blog.csdnimg.cn/20191202130704652.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM3NzgxNjQ5,size_16,color_FFFFFF,t_70) 

##### 流的创建

```java
List<String> strList = Arrays.asList("龙台", "百凯","英雄");
strList.stream().forEach(System.out::println);
// 顺序输出  龙台  百凯  英雄
```

 `Stream.iterate` 该方式是在 `Stream` 接口下的静态方法，见名识义可以看出是以迭代器的方式创建一个流 

```java
Stream.iterate(1, each -> each + 1)
        .limit(3)
        .collect(Collectors.toList())
        .forEach(System.out::println);
// 顺序输出 1 2 3
```

 `Stream.generate` 同样是 `Stream` 接口下的静态方法，参数就是一个 `Supplier` 的供给型的参数 

```java
Stream.generate(Math::random)
        .limit(5)
        .forEach(System.out::println);
// 依次打印5个随机数
```

#####  **`flatmap`**

 可以将一个二维的集合映射成一个一维，相当于他映射的深度比 **`map`** 深了一层 

```java
Stream<List<Integer>> inputStream = Stream.of(
        Arrays.asList(1),
        Arrays.asList(2, 3),
        Arrays.asList(4, 5, 6)
);
List<Integer> collect = inputStream
        .flatMap(each -> each.stream())
        .collect(Collectors.toList());
collect.forEach(System.out::println);
```

##### 流的过滤

```java
 List<Student> students = studentList.stream()
                .sorted(((s1, s2) -> s1.getNum().compareTo(s2.getNum())))
                .collect(Collectors.toList());
```

#####  **`limit`**  skip

和 数据库的分页道理是一致的，而 **`skip`** 则是反其道而行。通过代码来进行查看 

```java
List<Integer> integerList = Arrays.asList(1,2,3,4,5,6);
integerList.stream().limit(3).forEach(System.out::println);
// 打印 1 2 3 
integerList.stream().skip(3).forEach(System.out::println);
// 打印 4 5 6
```

##### groupingBy 

 流的聚合 **`groupingBy`** 和 **`mysql`** 数据库中的 **`Group By`** 原理基本一致 

```java
List<Student> studentList = Arrays.asList(
        new Student("003", "英雄","山东"),
        new Student("002", "偏执","北京"),
        new Student("001", "狂妄","山东"));

Map<String, List<Student>> stuGroup = studentList.stream()
        .collect(Collectors.groupingBy(each -> each.getArea()));

stuGroup.forEach((key, val) -> System.out.println(key + ":" + val.toString()));

山东:[Student(num=003, name=英雄, area=山东), Student(num=001, name=狂妄, area=山东)]
北京:[Student(num=002, name=偏执, area=北京)]
```

#####  **`partitioningBy`** 

 **`partitioningBy`**  返回参数的 **`key`** 为 **`Boolean`** 类型，在执行 **`partitioningBy`** 时，如果地区为 “山东”则为 **`True`** 

```java
List<Student> studentList = Arrays.asList(
        new Student("003", "英雄","山东"),
        new Student("002", "偏执","北京"),
        new Student("001", "狂妄","山东"));

Map<Boolean, List<Student>> stuGroup = studentList.stream()
        .collect(Collectors.partitioningBy(each -> Objects.equals(each.getArea(), "山东")));

System.out.println("结果为 true :" + stuGroup.get(true));
System.out.println("结果为 false :" + stuGroup.get(false));

结果为 true :[Student(num=003, name=英雄, area=山东), Student(num=001, name=狂妄, area=山东)]
结果为 false :[Student(num=002, name=偏执, area=北京)]
```

##### 流的获取findFirst

```java
// 如果流获取第一个元素为空，那么就会返回默认值
Optional<String> optional = Arrays.asList("偏执","狂妄","英雄").stream().findFirst();
System.out.println(optional.orElse("默认返回"));
```

##### anyMatch **`allMatch`** `noneMatch`

任何一个元素匹配，返回 true

```java
// 毫无疑问 当执行到元素 4 时，返回 **`true`**
boolean anyMatchResult = Arrays.asList(1,2,3,4,5).stream().anyMatch(each -> each > 3);
```

**`allMatch`** 所有元素匹配，返回 **`true`**

**`noneMatch`** 没有一个元素匹配，返回  **`true`**

##### **`average、count、max、min、sum`**

 **`average、count、max、min、sum`** 其实就相当与 **`summaryStatistics`** 拆出的个体 一笔带过了 

```java
Stream<Integer> stream = Stream.of(1,2,3,4,5);
OptionalInt max = stream.mapToInt(each -> each).max();
OptionalLong min = stream.mapToLong(each -> each).min();
OptionalDouble average = stream.mapToInt(each -> each).average();
long count = stream.mapToInt(each -> each).count();
int sum = stream.mapToInt(each -> each).sum();
```

stream().collect()

```java
//转换为set集合
userList.stream().map(User::getName).collect(Collectors.toSet()).forEach(System.out::println);

//转换为HashSet集合     
userList.stream().map(User::getName).collect(Collectors.toCollection(LinkedHashSet::new)).forEach(System.out::println);

//通过.collect()计算平均值
Double avgAge = userList.stream().collect(Collectors.averagingDouble(User::getAge));

//一级分组
Map<String, List<User>> map1 = userList.stream().collect(Collectors.groupingBy(item -> item.getSex()));

//多级分组
Map<String, Map<String, List<User>>> map2 = userList.stream().collect(Collectors.groupingBy(User::getSex, Collectors.groupingBy(item -> {
    if (item.getAge() >= 40) {
        return "中年";
    } else if (item.getAge() <= 20) {
        return "小伙子";
    } else {
        return "青年";
    }
})));

//分区，满足条件个一个区 不满足条件的一个区
Map<Boolean, List<User>> partitionBySex = userList.stream().collect(Collectors.partitioningBy(item -> item.getSex().equals("男")));

//连接字符串
String join = userList.stream().map(User::getName).collect(Collectors.joining(","));

```



#### 反射

#### SQL函数复杂查询