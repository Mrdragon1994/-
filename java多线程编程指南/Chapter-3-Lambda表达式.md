---
title: Chapter 3 Lambda表达式
date: 2020-04-04 22:37:59
tags:
- Java8
- Java
categories:
- Java8
- Java
---

# Chapter 3 Lambda表达式

## 3.1 Lambda管中窥豹

1. 可以把Lambda表达式理解为简洁地表示可传递的匿名函数的一种方式：它没有名称，但是有参数列表，函数主体，返回类型，甚至还有一个可抛出的异常列表。

2. **匿名**—它不像普通的方法那样有一个明确的名称;

3. **函数**—我们说它是函数，是因为Lambda函数不像方法那样属于某个特定的类，但和方法一样，Lambda有参数列表，函数主体，返回类型，还有可抛出的异常列表；

4. **传递**—Lambda表达式可以作为参数传递给方法或存储在变量中；

5. **简洁**—无需像匿名类那样写很多模板代码；

   ![image-20200404224741275](Chapter-3-Lambda表达式.assets/image-20200404224741275.png)

```java
//使用Lambda之前
Comparator<Apple> byWeight = new Comparator<Apple>() {
    public int compare(Apple a1, Apple a2) {
        return a1.getWeight() - a2.getWeight()
    }
};
//使用Lambda之后
Comparator<Apple> byWeight = (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight());
```

6. Java8中有效的几个Lambda表达式

   ```java
   //该表达式具有一个String类型的参数并返回一个int
   (String s) -> s.length()
   //该表达式具有一个Apple类型的参数并返回一个boolean的值
   (Apple a) -> a.getWeight() > 150
   //该表达式具有两个int类型的参数然而没有返回值,注意Lambda表达式是可以包含多行语句的
   (int x, int y) -> {
       System.out.println("Result:");
       System.out.println(x + y);    
   }   
   //该表达式没有输入参数，返回一个int
   () -> 42
   //该表达式具有俩个Apple类型的参数,返回一个int
   (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight())    
   ```

7. Lambda常见的表达式语法是：

   ```java
   (parameters) -> expression
   (parameters) -> {
   	statements;
   }
   ```

   如果不放在{}中，那么expression是没有分号的，如果放在了{}中，那么表达式必须具有分号！！！

## 3.2 在哪里以及如何使用Lambda

### 3.2.1 函数式接口

1. **函数式接口**就是只定义了一个**抽象方法**的接口，当接口中有**默认方法**时，不论它有多少，只要该接口只定义了一个抽象方法，那么他就是函数式接口。

   常见的有：

   ```java
   //java.util.Comparator
   public interface Comparator<T> {
   	int compare(T o1, T o2);
   }
   //java.lang.Runable
   public interface Runnable {
       void run();
   }
   //java.util,concurrent.Callabel
   public interface Callable<V> {
       V call();
   }
   ```


### 3.2.2 函数描述符

1. 写函数描述符时，我们只需要关注函数式接口中的抽象方法即可，(parameters)是抽象方法的方法签名，expression是抽象方法的方法体，且要体现出来返回值类型;

   ```java
   Thread t = new Thread(() -> System.out.println("aaa"))
   ```

2. 函数式接口的标注性注解是：@FunctionalInterface

## 3.4 使用函数式接口

​	在java.util.function包中引入了常见的几个函数式接口。

原始类型特化：

1. 就如java中的基本类型和包装类型，java可以帮我们自动实现装箱和拆箱，在函数式接口的应用过程中，java8为我们提供了一个专门的版本，以便在输入和输出都是原始类型时避免自动装箱操作，这些可以改变输入和输出类型的函数式接口是在基础的函数式接口之上封装的，因此，我们可以称之为原始类型特化；
2. 一般来说，针对专门的输入参数类型的函数式接口的名称都要加上对饮的原始类型前缀，比如：DoublePredicate, IntConsumer, LongBinaryOperator, IntFunction等；Function接口还有针对输出参数类型的变种，ToIntFunction\<T\>，IntToDoubleFunction。

### 3.4.1 java.util.function.Predicate\<T\>接口

1. java.util.function.Predicate\<T\>接口定义了一个名为test的抽象方法，它接收泛型T的对象，返回一个boolean值

   ```java
   @FunctionalInterface
   public interface Predicate<T> {
   	boolean test(T t);
   }
   ```

   函数描述符：T -> boolean

   原始类型特化：IntPredicate, LongPredicate, DoublePredicate

### 3.4.2 java.util.function.Consumer\<T\>接口

1. java.util.function.Consumer\<T\>定义了一个名为accept()的抽象方法，

   ```java
   @FunctionalInterface
   public interface Consumer<T> {
       void accept(T t);
   }
   ```

   函数描述符：T -> void

   原始类型特化：IntConsumer, LongComsumer, DoubleConsumer

### 3.4.3 java.util.function.Function\<T, R\>接口

1. java.util.function.Function\<T, R\>接口定义了一个叫做apply的方法

   ```java
   @FunctionalInterface
   public interface Function<T, R> {
       R apply(T t);
   }
   ```

   函数描述符：T -> R

   原始类型特化：IntFunction\<R>,  IntToDoubleFunction,IntToLongFunction,  LongFunction\<R>,
   LongToDoubleFunction,LongToIntFunction,DoubleFunction\<R>,ToIntFunction\<T>,ToDoubleFunction\<T>,ToLongFunction\<T>  

### 3.4.4 java.util.function.Supplier\<T\>接口

1. 

   ```
   
   ```

   函数描述符：() -> T

   原始类型特化：BooleanSupplier,IntSupplier, LongSupplier, DoubleSupplier

### 3.4.5 java.util.function.UnaryOperator\<T\>接口

1. 

   ```
   
   ```

   函数描述符：T -> T

   原始类型特化：IntUnaryOperator, LongUnaryOperator, DoubleUnaryOperator

### 3.4.6 java.util.function.BinaryOperator\<T\>接口

1. 

   ```
   
   ```

   函数描述符：(T, T) -> T

   原始类型特化：IntBinaryOperator, LongBinaryOperator, DoubleBinaryOperator

### 3.4.7 java.util.function.BiPredicate\<L, R\>接口

1. 

   ```
   
   ```

   函数描述符：(L, R) -> boolean

   原始类型特化：无

### 3.4.8 java.util.function.BiConsumer\<T, U\>接口

1. 

   ```
   
   ```

   函数描述符：(T, U) -> void

   原始类型特化：ObjIntConsumer\<T>,ObjLongConsumer\<T>,ObjDoubleConsumer\<T>

### 3.4.9 java.util.function.BiFunction\<T, U, R\>接口

1. 

   ```
   
   ```

   函数描述符：(T, U) -> R

   原始类型特化：ToIntBiFunction\<T,U>,ToLongBiFunction\<T,U>,ToDoubleBiFunction\<T,U>  

总结描述如下：

| name           | type               | description                   |
| -------------- | ------------------ | ----------------------------- |
| Consumer       | Consumer\<T>       | 接收T对象，不返回值           |
| Predicate      | Predicate\<T>      | 接收T对象并返回boolean        |
| Function       | Function\<T, R>    | 接收T对象，返回R对象          |
| Supplier       | Supplier\<T>       | 提供T对象，不接收值           |
| UnaryOperator  | UnaryOperator\<T>  | 接收T对象，返回T对象          |
| BiConsumer     | BiConsumer\<T, U>  | 接收T对象和U对象，不返回值    |
| BiPredicate    | BiPredicate\<T,U>  | 接收T对象和U对象，返回boolean |
| BiFunction     | BiFunction\<T,U,R> | 接收T对象和U对象，返回R对象   |
| BinaryOperator | BinaryOperator\<T> | 接收两个T，返回T对象          |
|                |                    |                               |

| 使用案例              | Lambda的例子                                                 | 对应的函数式接口                                             |
| --------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 布尔表达式            | (List\<String> list) -> list.isEmpty()                       | Predicate\<List\<String>>                                    |
| 创建对象              | () -> new Apple(10)                                          | Supplier\<Apple>                                             |
| 消费一个对象          | (Apple a) -> System.out.println(a.getWeight())               | Consumer\<Apple>                                             |
| 从一个对象中选择/提取 | (String s) -> s.length()                                     | Function\<String, Integer>或ToIntFunction\<String>           |
| 合并两个值            | (int a, int b) -> a * b                                      | IntBinaryOperator                                            |
| 比较两个对象          | (Apple a1, Apple a2) -> a1.getWeight().compareTo(a2.getWeight()) | Comparator\<Apple>或BiFunction\<Apple, Apple, Integer>或ToIntBiFunction\<Apple, Apple> |
|                       |                                                              |                                                              |

https://www.cnblogs.com/linzhanfly/p/9686941.html

https://www.cnblogs.com/theRhyme/p/10774720.html

---

具体实现在看

## 3.5 类型检查、类型推断以及限制









