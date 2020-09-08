---
layout:     post
title:      "JAVA8 Lambda表达式"
subtitle:   "JAVA8，Lambda"
date:       2019-04-14
author:     "Paul"
header-img: "img/12.jpg"
tags:
    - JAVA8
    - Lambda
---



> JAVA8 新特性

**概述**
Lambda表达式可以说是JAVA8最为核心的东西了，它将函数式编程的概念引入了Java。Lambda表达式允许把函数作为一个方法的参数传入到方法中，或者把代码看作数据。

**函数是编程的好处**
* 代码简洁直观，比如使用stream API就可以让你告别for循环。
* 多核友好，Java函数式编程使得编写并行程序非常简单，只需要调用一下parallel()方法即可。

**面向对象与函数是编程的比较**
启动一个线程并输出一句话
```java
        new Thread(new Runnable(){
            @Override
            public void run() {
                System.out.println("使用传统方式开启线程");
            }
        }).start();
```
这里使用Runnable接口的匿名内部类来指定任务内容，其实这个new Runnable对象并不是我们想要的，下面来看Lambda表达式怎么实现。
```java
    new Thread(()->System.out.println("使用Lambda方式开启线程")).start();
```

再来看一个使用Lambda表达式优化编程的例子。
需求：根据不同的条件筛选出不同的商品。

* 传统方式
  针对不同的需求写不同的实现。
   ```java
   //需求：筛选出所有名称包含手机的商品
  public List<Product> filterProductByName(List<Product> products){
    List<Product> list = new ArrayList<>();
    for (Product product : products) {
        if (product.getName().contains("手机")) {
            list.add(product);
        }
    }
    return list;
  }
  //需求：筛选出所有价格大于1000的商品
  public List<Product> filterProductByPrice(List<Product> products){
    List<Product> list = new ArrayList<>();
    for (Product product : products) {
        if (product.getPrice() > 1000) {
            list.add(product);
        }
    }
    return list;
  }
   ```
* 使用策略设计模式优化
  策略接口
  ```java
  public interface MyPredicate<T> {
    boolean test(T t);
  }
  ```
  策略实现类
  ```java
  	public class FilterProductByName implements MyPredicate<Product> {
    @Override
    public boolean test(Product t) {
        return t.getName().contains("手机");
    }
  } 
  public class FilterProductByPrice implements MyPredicate<Product> {
    @Override
    public boolean test(Product t) {
        return t.getPrice() > 1000;
    }
  }
  ```
  这时还是要针对不同的接口写不同的策略类。
* 使用匿名内部类优化
  ```java
   @Test
  public void test3() throws Exception {
    List<Product> list = filterProduct(products, new MyPredicate<Product>() {
        @Override
        public boolean test(Product t) {
            return t.getName().contains("手机");
        }
    });
   
    for (Product product : list) {
        System.out.println(product);
    }
  }
  ```
  虽然使用匿名内部类后我们的代码已经做到了很少的冗余了，但是每次都要new一个对象，而且要实现里面的抽象方法也很麻烦。
* Lambda表达式优化
  ```java
    @Test
  public void test4() throws Exception {
    filterProduct(products, (p) -> p.getPrice() > 1000).forEach(System.out::println);
  }
  ```
* Stream API优化
  ```java
  //获取价格从高到低排行前三的商品信息
   @Test
  public void test5() throws Exception {
    //获取价格从高到低排行前三的商品信息
    products.stream().sorted((x,y) -> y.getPrice().compareTo(x.getPrice()))
    .limit(3).forEach(System.out::println);
  }
  ```

**Lambda表达式语法**
```java
@Test
public void testLambda() throws Exception {
    new Thread(()->System.out.println("使用Lambda方式开启线程")).start();;
}
```
原来Thread类的构造器中需要传入一个Runnable接口的实现类，实现类里面只有一个run方法来决定具体做什么。

所以Lambda表达式在这里就是将要完成的任务代码传递给Thread。
```java
()->System.out.println("使用Lambda方式开启线程");
```
-> 被称为Lambda操作符或箭头操作符。它将Lambda分为了两个部分：
左侧：参数，run方法不需要参数，所以可以直接写()。
右侧：函数体，就是Lambda表达式要执行的任务，功能或者行为。

那么是不是所有的匿名内部类都可以使用Lambda表达式替换那？
不可以，接口（可以看成Lambda表达式的容器）中只有一个抽象方法时，Lambda表达式才能正确的匹配。


**函数式接口**
我们把只有一个抽象方法的接口称作为“函数式接口”，只有函数是接口才可以使用Lambda表达式进行函数式编程。

```java
修饰符 interface 接口名称 {
 
[public abstract] 返回值类型 方法名称(可选参数信息);
 
// 其他
 
}
```
JAVA8中引入了一个新的注解@FunctionalInterface，这个注解可以作用到接口上，强制该接口只能由一个抽象方法，没有该注解，满足函数式接口的定义也可以。

查看Runnable的源码，发现它就是一个函数式接口。
![lambda](/imgblog/lambda.png)

**语法总结**

* 可选的类型声明：不需要声明参数类型。
* 可选的参数圆括号（）：一个参数无需定义圆括号，多个参数需要。
* 可选的大括号{}：如果主题包含了一个语句，就不需要使用{}。
* 可选的return，如果主体只有一个表达式返回值则可以省略return和{}。


**内置函数式接口**
JDK提供了大量常用的函数式接口以丰富Lambda的典型使用场景，主要在java.util.function包中提供。

| 函数式接口    | 参数类型 | 返回类型 | 说明                     |
| ------------- | -------- | -------- | ------------------------ |
| Consumer<T>   | T        | void     | 方法：void accept（T t） |
| Supplier<T>   | 无       | T        | 方法：T get()            |
| Function(T,R) | T        | R        | 方法：R apply(T t)       |
| Predicate<T>  | T        | boolean  | 方法：boolean test(T t)  |

代码使用演示：
```java
//函数式接口类型，参数类型，返回类型，说明
//Consumer<T>    T       void   void accept(T t)

@Test
public void test1() throws Exception{
    shop(10000.0,new Consumer<Double>(){
	@Override
	public void accept(Double moeny){
		System.out.println("买电脑花了"+money+"元");
}
})
shop(5000.0, money-> System.out.println("购买手机花费:" + money + "元"));
}

public static void shop(Double money,Consumer<Double> con){
	con.accpet(money);
}

//函数式接口类型，参数类型，返回类型，说明
//Supplier<T>    无       T     T get();可用作工厂
 @Test
    public void test2() throws Exception {
        String code = getCode(4, ()-> new Random().nextInt(10));
        System.out.println(code);
    }
    //获取指定位数的验证码
    public String getCode(int num, Supplier<Integer> sup){
        String str = "";
        for (int i = 0; i <num ; i++) {
            str += sup.get();
        }
        return str;
    }

//函数式接口类型，参数类型，返回类型，说明
//Function<T,R>    T       R    返回结果是R类型的对象，方法：R apply(T t)
@Test
public void test3() throws Exception{
	int length = getStringRealLength("str",str->str.trim().length);
	System.out.println(length);
}
public int getStringRealLength(String str,Function<String,Integer> fun){
	return fun.apply(str);
}
```

在使用Lambda表达式的时候，我们实际传递进去的代码就是一种解决方案：拿什么参数做什么操作。

**日志性能优化**
记录日志的典型场景就是对日志消息进行拼接后，在满足条件的情况下进行打印输出：
```java
public class LambdaLazyDemo {
    @Test
    public void testLambdaLazy() throws Exception {
        String msgA = "Hello";
        String msgB = "World";
        String msgC = "Java";
 
        log(1, msgA + msgB + msgC);
 
 
    }
    private static void log(int level, String msg) {
        if (level == 1) {//不论条件是否满足,msg都是进行拼接然后传递过来的
            System.out.println(msg);
        }
    }
}
```
这段代码的问题是无论级别是否满足要求，作为log方法的第二个参数，三个字符串一定会被首先拼接并传入方法内，然后才会进行级别判断。

SLF4J为了解决这种性能浪费的问题，并不推荐首先进行字符串的拼接，而是将字符串的若干部分作为可变参数传入方法内，仅在日志级别满足要求的情况下才会进行拼接。
```java
 LOGGER.debug("变量{}的取值为{}。", "os", "windows") ;
```
Lambda可以做到更好。