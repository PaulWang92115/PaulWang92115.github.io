---
layout:     post
title:      "JAVA8 Stream API"
subtitle:   "JAVA8，Stream，流"
date:       2019-04-14
author:     "Paul"
header-img: "img/12.jpg"
tags:
    - JAVA8
    - Stream
    - 流
---



> JAVA8 新特性

Stream API 提供了对集合类数据处理的一种更加高效的方式。可以把他理解为数据的管道，可以通过这条管道提供的API很方便的对于里面的数据进行复杂操作，比如查找，过滤和映射。

![streamapi](/imgblog/streamapi.png)

四种stream接口继承自BaseStream，其中IntStream，LongStream，DoubleStream三种基本类型。为不同数据类型设置不同stream接口，可以提高性能。

**传统集合操作 VS Stream API**
便利集合，找出符合某些条件的成员
```java
@Test
public void testWithFor() throws Exception {
    List<String> list = new ArrayList<>();
    list.add("马云");
    list.add("马化腾");
    list.add("李彦宏");
    list.add("雷军");
    list.add("刘强东");
    List<String> maList = new ArrayList<>();
    for (String name : list) {
        if (name.startsWith("马")) {
            maList.add(name);
        }
    }
    List<String> length3List = new ArrayList<>();
    for (String name : maList) {
        if (name.length() == 3) {
            length3List.add(name);
        }
    }
    for (String name : length3List) {
        System.out.println(name);
    }
}
```

Stream的优雅写法
```java
@Test
public void testWithStream() throws Exception {
    List<String> list = new ArrayList<>();
    list.add("马云");
    list.add("马化腾");
    list.add("李彦宏");
    list.add("雷军");
    list.add("刘强东");
    list.stream()
        .filter(s -> s.startsWith("马"))
        .filter(s -> s.length() == 3)
        .forEach(System.out::println);

}
//首先获取流，过滤姓马的，过滤出名字长度为3的，逐一打印。
```

图中展示了过滤、映射、跳过、计数等多步操作，每一个竖着的方框都是一个“流”，调用指定的方法，可以从一个“流”转换为另一个“流”。
![flow](/imgblog/flow.png)

**内部迭代于外部迭代**
使用Iterator迭代器迭代集合的方式称之为外部迭代，就是需要手动对集合进行种种操作才能得到结果的方式。
与之相反的称之为内部迭代，是集合本身内部通过流进行处理后直接获取结果就可以了。
![inout](/imgblog/inout.png)

```java
@Test
public void test2() throws Exception {
    System.out.println("=======================外部迭代=========================");
    List<Integer> list = Arrays.asList(1, 2, 1, 3, 3, 2, 4);
    for (Integer integer : list) {
        System.out.println(integer);
    }
    Iterator<Integer> iterator = list.iterator();
    while (iterator.hasNext()) {
        System.out.println(iterator.next());
    }
    System.out.println("=======================内部迭代=========================");
    Stream<Integer> stream = list.stream().filter(i -> {
        System.out.println("内部迭代惰性求值");
        return i % 2 == 0;
    });
    //只有执行return时才会执行对应的system.out
```

**操作步骤**
* 创建流：通过数据源（集合，数组）获取流
* 处理流：对流中的数据进行处理
* 收集流：通过调用收集方法，产生结果。

**创建流**
以下几种方式通常用来创建流：
1 Collection的默认方法stream()和parallelStream()
2 Arrays.stream()
3 Stream.of()
4 Stream.iterate()  //迭代无限流(1,n->n+1)
5 Stream.generate() //生成无限流(Math::random)
```java
@Test
public void testCreateStream() throws Exception {
    // 1.Collection的默认方法stream()和parallelStream()
    List<String> list = Arrays.asList("a", "b", "c");
    Stream<String> stream = list.stream();// 获取顺序流
    Stream<String> parallelStream = list.parallelStream();
 
    // 2.Arrays.stream()
    IntStream intStream = Arrays.stream(new int[] { 1, 2, 3 });
    Stream<Integer> IntegerStream = Arrays.stream(new Integer[] { 1, 2, 3 });
 
    // 3.Stream.of()
    Stream<Integer> IntegerStream2 = Stream.of(1, 2, 3, 4);
    IntStream intStream2 = IntStream.of(1, 2, 3);
 
    // 4.Stream.iterate()//迭代无限流
    // Stream.iterate(1, n->n +1).forEach(System.out::println);
    Stream.iterate(1, n -> n + 1).limit(100).forEach(System.out::println);
 
    // 5.Stream.generate()//生成无限流
    // Stream.generate(Math::random).forEach(System.out::println);
    Stream.generate(Math::random).limit(2).forEach(System.out::println);

}
```

**处理流**
* 筛选和切片
  filter(Predicate<T> p)：过滤（根据传入的Lambda返回的true/false从流中过滤掉某些数据或筛选出某些数据）。
  distinct()：去重（根据流中数据的hashcode和equals去除重复元素）。
  limit(long n)：限定保留n个数据。
  skip(long n)：跳过n个数据。
  ![filter1](/imgblog/filter1.png)

  ![filter2](/imgblog/filter2.png)

```java
@Test
public void test1() throws Exception {
    Arrays.asList(1, 2, 1, 3, 3, 2, 4).stream().filter(i -> i % 2 == 0).forEach(System.out::println);
    System.out.println("================================================");
    Arrays.asList(1, 2, 1, 3, 3, 2, 4).stream().filter(i -> i % 2 == 0).distinct().forEach(System.out::println);
    System.out.println("================================================");
    Arrays.asList(1, 2, 1, 3, 3, 2, 4).stream().distinct().limit(2).forEach(System.out::println);
    System.out.println("================================================");
    Arrays.asList(1, 2, 1, 3, 3, 2, 4).stream().distinct().skip(2).forEach(System.out::println);
}
```
* 映射
  映射map(Function<T，R> f)：接收一个函数作为参数，该函数会被应用到流中的每个元素上，并将其映射成一个新的元素。
  flatMap(Function<T,Strean<R>> mapper)：接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流链接成一个流。
  ![map](/imgblog/map-5503590.png)

  ![map2](/imgblog/map2.png)

```java
@Test
public void test3() throws Exception {
    System.out.println("=======================map=========================");
    Stream<String> stream = Stream.of("i","love","java");
    stream.map(s -> s.toUpperCase()).forEach(System.out::println);
    System.out.println("=======================flatMap========================");
    Stream<List<String>> stream2 = Stream.of(Arrays.asList("H","E"), Arrays.asList("L", "L", "O"));
    stream2.flatMap(list -> list.stream()).forEach(System.out::println);
}
```
* 排序
  sorted()：自然排序使用Comparable<T> 的int compareTo(T o)方法。
  sorted(Comparator<T> com)：定制排序使用Comparator的int compare(T o1,T o2)方法。

```java
@Test
public void test4() throws Exception {
    System.out.println("====================自然排序============================");
    Arrays.asList(3, 2, 1, 4, 5, 8, 6).stream().sorted().forEach(System.out::println);
    System.out.println("====================定制排序============================");
    Arrays.asList(3, 2, 1, 4, 5, 8, 6).stream().sorted((x,y) -> y.compareTo(x)).forEach(System.out::println);
}
```

**收集流**
* 查找匹配
  allMatch：检查是否匹配所有元素。
  anyMatch：检查是否至少匹配一个元素。
  noneMatch：检查是否没有匹配元素。
  findFirst：返回第一个元素。
  findAny：返回当前流中的任意元素（一般用于并行流）。
```java
Optional<T> 时java8新加入的一个容器，这个容器只存1个或0个元素，用于防止出现NullPointerException
*isPresent() //判断容器中是否有值
*ifPresent(Consume lanbda) //容器不为空则执行括号中的lambda表达式
*T get() //获取容器中的元素，若容器为空则抛出NoSuchElement异常。
*T orElse(T other) //获取容器中的元素，若容器为空则返回括号中的默认值。
```
匹配代码实现
```java
@Test
public void test5() throws Exception {
    System.out.println("======================检查是否匹配所有==========================");
    boolean allMatch = Arrays.asList(3, 2, 1, 4, 5, 8, 6).stream().allMatch(x-> x>0);
    System.out.println(allMatch);
    System.out.println("======================检查是否至少匹配一个元素====================");
    boolean anyMatch = Arrays.asList(3, 2, 1, 4, 5, 8, 6).stream().anyMatch(x -> x>7);
    System.out.println(anyMatch);
    System.out.println("======================检查是否没有匹配的元素======================");
    boolean noneMatch = Arrays.asList(3, 2, 1, 4, 5, 8, 6).stream().noneMatch(x -> x >10);
    System.out.println(noneMatch);
    System.out.println("======================返回第一个元素==========================");
    Optional<Integer> first = Arrays.asList(3, 2, 1, 4, 5, 8, 6).stream().findFirst();
    System.out.println(first.get()); 
    System.out.println("======================返回当前流中的任意元素=======================");
    Optional<Integer> any = Arrays.asList(3, 2, 1, 4, 5, 8, 6).stream().findAny();
    System.out.println(any.get());
}
```
* 统计
  count()：返回流中元素的总个数。
  max(Comparator<T>)：返回流中最大值。
  min(Comparator<T>)：返回流中最小值。
```java
@Test
public void test6() throws Exception {
    long count = Arrays.asList(3, 2, 1, 4, 5, 8, 6).stream().count();
    System.out.println(count);
    Optional<Integer> max = Arrays.asList(3, 2, 1, 4, 5, 8, 6).stream().max((x,y) -> x.compareTo(y));
    System.out.println(max.get());
    Optional<Integer> min = Arrays.asList(3, 2, 1, 4, 5, 8, 6).stream().min((x,y) -> x.compareTo(y));
    System.out.println(min.get());
}
```
* 归约
  reduct(T identity,BinaryOperator)/reduce(BinaryOperator)：将流中的元素挨个整合起来，得到一个值。
  ![reduce](/imgblog/reduce.png)
```java
@Test
public void test7() throws Exception {
    System.out.println("=====reduce:将流中元素反复结合起来，得到一个值==========");
    Stream<Integer> stream = Stream.iterate(1, x -> x+1).limit(100);
    //stream.forEach(System.out::println);
    Integer sum = stream.reduce(0,(x,y)-> x+y);
    System.out.println(sum);
}
```
* 汇总
  reduce擅长的时生成一个值，如果想要从Stream生成一个集合或者Map等复杂对象时可以使用Collect()。
  collect(Collector<R,A,R>)：将流转换为其他形式。可以转换为list，set，TreeSet，map，sum，avg，max，min等等。
```java
@Test
public void test8() throws Exception {
    System.out.println("=====collect:将流转换为其他形式:list");
    List<Integer> list = Stream.iterate(1, x -> x+1).limit(100).collect(Collectors.toList());
    System.out.println(list);
    System.out.println("=====collect:将流转换为其他形式:set");
    Set<Integer> set = Arrays.asList(1, 1, 2, 2, 3, 3, 3).stream().collect(Collectors.toSet());
    System.out.println(set);
    System.out.println("=====collect:将流转换为其他形式:TreeSet");
    TreeSet<Integer> treeSet = Arrays.asList(1, 1, 2, 2, 3, 3, 3).stream().collect(Collectors.toCollection(TreeSet::new));
    System.out.println(treeSet);
    System.out.println("=====collect:将流转换为其他形式:map");
    Map<Integer, Integer> map = Stream.iterate(1, x -> x+1).limit(100).collect(Collectors.toMap(Integer::intValue, Integer::intValue));
    System.out.println(map);
    System.out.println("=====collect:将流转换为其他形式:sum");
    Integer sum = Stream.iterate(1, x -> x+1).limit(100).collect(Collectors.summingInt(Integer::intValue));
    System.out.println(sum);
    System.out.println("=====collect:将流转换为其他形式:avg");
    Double avg = Stream.iterate(1, x -> x+1).limit(100).collect(Collectors.averagingInt(Integer::intValue));
    System.out.println(avg);
    System.out.println("=====collect:将流转换为其他形式:max");
    Optional<Integer> max = Stream.iterate(1, x -> x+1).limit(100).collect(Collectors.maxBy(Integer::compareTo));
    System.out.println(max.get());
    System.out.println("=====collect:将流转换为其他形式:min");
    Optional<Integer> min = Stream.iterate(1, x -> x+1).limit(100).collect(Collectors.minBy((x,y) -> x-y));
    System.out.println(min.get());
}
```
* 分组和分区
  Collectors.groupingBy()对元素做group操作。
  Collectors.partitioningBy()对元素进行二分区操作。
  ![groupby](/imgblog/groupby.png)

  ![groupby2](/imgblog/groupby2.png)

```java
@Test
public void test9() throws Exception {
    System.out.println("=======根据商品分类名称进行分组==========================");
    Map<String, List<Product>> map = products.stream().collect(Collectors.groupingBy(Product::getDirName));
    System.out.println(map);
    System.out.println("=======根据商品价格范围多级分组==========================");
    Map<Double, Map<String, List<Product>>> map2 = products.stream().collect(Collectors.groupingBy(
            Product::getPrice, Collectors.groupingBy((p) -> {
                if (p.getPrice() > 1000) {
                    return "高级货";
                    } else {
                        return "便宜货";
                        }
                })));
    System.out.println(map2);
 
}
@Test
public void test10() throws Exception {
    System.out.println("========根据商品价格是否大于1000进行分区========================");
    Map<Boolean, List<Product>> map = products.stream().collect(Collectors.partitioningBy(p -> p.getPrice() > 1000));
    System.out.println(map);
}
```
