---
title: 了解lambda表达式
date: 2017-11-23 14:30:36
tags: Java8
icon: fa-battery-quarter
---
# 了解lambda表达式

## 什么是lambda

lambda表达式是一段可以传递的代码，它的核心思想是将面向对象中的传递数据变成传递行为。
我们回顾下jdk8之前编写线程时的代码段：

```java
Runnable r = new Runnable() {
    @Override
    public void run() {
        System.out.println("do something.");
    }   
}
```

把**Runnable**对象传给**Thread**对象作为构造参数时创建一个线程，运行后将输出do something.。
我们使用匿名内部类的方式实现了该方法。设计匿名内部类的目的，就是为了方便Java程序员将代码作为数据传递。不多匿名内部类还是不够简便。为了实现一个简单的逻辑，不得不加上6行冗繁的样板代码。如果是lambda会怎么做？

```java
Runnable r = () -> System.out.println("do something.");
```
这样的代码很酷，使用**->**将参数和实现逻辑分离，当运行这个线程的时候执行的是**->**之后的代码段，而且编译器帮助我们做了类型推倒；
下面我们一起来学习lambda的语法。

## 基本语法

在lambda中我们遵循如下的表达式来编写：

```java
expression = (variable) -> action
```

* **variable**：这是一个变量，一个占位符。像x，y，z可以是多个变量。
* **action**：这里我们称之为action，这是我们实现代码逻辑的部分。当一个动作实现无法用一行代码完成，可以用**{}**包裹起来。

lambda表达式可以包含多个参数：

```java
int sum = (x, y) -> x + y;
```

这个时候我们思考这段代码不是之前的x和y数字相加，而是创建了一个函数，用来计算两个数的和。
后面用int类型进行接收。在lambda表达式中我们省略了return。

## 函数式接口

函数式接口是只有一个方法的接口，用作lambda表达式的类型。来看看jdk中的**Runnable**源码

```java
@FunctionalInterface
public interface Runnable {
    public abstract void run();
}   
```

### 尝试函数式接口

我们来编写一个函数式接口，输入一个年龄，判断这个人是否是成年人。

```java
public class FunctionalInterfaceDemo {
    @FunctionalInterface
    interface Predicate<T> {
        boolean test(T t);
    }
    
    public static boolean doPredicate(int age, Predicate<Integer> predicate) {
        return predicate.test(age);
    }
    
    public static void main(String[] args) {
        boolean isAdult = doPredicate(20, x -> x >= 18);
        System.out.println(isAdult);
    }
}
```

这个列子我们很轻松的完成是否是承认的动作，在此之前我们的做法是编写一个判断是否是成人的方法，是无法将判断公用的。而在本例中你只需要将**行为**传递进去。

jdk中为我们提供了**java.util.function**包
![Markdown](http://i1.bvimg.com/619704/6ea5c79f6aa4a444t.jpg)

我们前面写的Predicate接口函数也是jdk中的一个实现，他们大致分为以下几类：

|    接口  |    类型  |   参数   |   返回值   |   事例  |
| :----: | :------:| :------: | :------: | :-----: |
| consumer | 消费型接口 | T    |    void |   输出一个值 |
| supplier | 供给型接口 | None    |    T |   工厂方法 |
| function | 函数型接口 | T    |    R   |   获得Animal对象的名字 |
| predicate | 断言型接口 | T    |    boolean |   这个用户已经成年了吗 |

**消费型接口**

```java
public static void donation(Integer money, Consumer<Integer> consumer) {
    consumer.accept(money);
}
public static void main(String[] args) {
    donation(1000, money -> System.out.println("捐赠了" + money + "元"));
}
```

**供给型接口**

```java
public static List<Integer> supply(Integer num, Supplier<Integer> supplier) {
    List<Integer> resultList = new ArrayList<Integer>();
    for(int i = 0; i < num; i++) {
        resultList.add(supplier.get());
    }
    return resultList;
}
public static void main(String[] args) {
    List<Integer> list = supply(10, () -> (int)(Math.random() * 100));
    list.forEach(item -> System.out.println(item));
}
```

**函数型接口**

```java
public static Integer convert(String str, Function<String, Integer> function) {
    return function.apply(str);
}

public static void main(String[] args) {
    Integer value = convert("28", str -> Integer.parseInt(str));
}
```

**断言型接口**

```java
public static List<String> filter(List<String> fruit, Predicate<String> predicate) {
    List<String> f = new ArrayList<String>();
    for (String s : fruit) {
        if(predicate.test(s)) {
            f.add(s);
        }
    }
    return f;
}
public static void main(String[] args) {
    List<String> fruit = Arrays.asList("香蕉", "哈密瓜", "火龙果");
    List<String> newFruit = filter(fruit, (f) -> f.length() == 2);
    System.out.println(newFruit);
}
```




