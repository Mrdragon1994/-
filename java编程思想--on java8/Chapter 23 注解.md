[toc]
# 注解
## 标准注解
Java5引入了三种，Java7引入了一种，Java8引入了一种;
- @Override: 当前方法定义将覆盖基类方法
- @Deprecated: 表示当前方法过时
- @SuppressWarnings: 关闭不当的编译器警告信息
- @SafeVarargs: 用于禁止对具有泛型varargs参数的方法或构造函数的调用方发出警告
- @FunctionalInterface: 声明当前类型为函数式接口

## 元注解
元注解是用来注解其他注解的,目前Java提供了5种。

| 注解        | 解释                                                         |
| ----------- | ------------------------------------------------------------ |
| @Target     | 表示注解可以用在哪些地方,可能的ElementType参数包括:<br> CONSTRUCTOR:构造器<br>FIELD:字段声明(包括Enum实例)<br>LOCAL_VARIBLE:局部变量声明<br>METHOD:方法声明<br>PACKAGE:包声明<br>PARAMETER:参数声明<br>TYPE:类,接口或者enum声明 |
| @Retention  | 表示注解信息保存的时长,可选的RetentionPolicy参数包括:<br>SOURCE:源码阶段,编译时失效<br>CLASS:编译器,编译成Class文件还存在,但是运行时会被JVM丢弃<br>RUNTIME:一直留存到运行期,可以通过反射机制读取到注解的信息 |
| @Documented | 将此注解保存到JavaDoc中                                      |
| @Inherited  | 允许子类继承父类的注解                                       |
| @Repeatable | 允许一个注解可以被使用一次或者多次(Java8)                    |

## 定义注解
1. 注解的定义和接口很类似,@Interface,事实上,注解和Java接口一样,也会被编译成class文件;
2. 不含任何属性的接口被称为标记注解;
3. 自定义注解:
```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface UseCase {
    int id();
    String description() default "default Desription";
}
```
其中,description拥有一个默认值,因此使用时可不给值,但是id必须给出值。
需要额外注意的是:注解的属性定义是带括号的。
4. 在使用注解赋值的时候,如果没有特殊指定哪个属性,那么**默认是value()**

## 注解内部属性的类型
1.注解内部的属性类型大致可以取如下几种:
- 基本数据类型
- String
- enum
- Class
- Annotation(也就是注解嵌套)
- 数组
```
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface UseCase {
    int id();
    String description() default "default Desription";
    Class aclass();
    int[] ages() default  {12, 25};
    Constrains contrains() default @Constrains;
}
=====>
使用时：
@UseCase(id = 1, description = "123", aclass = AnnotationTest.class, ages = {10, 12})
    public void test() {

    }
```
2. 属性的默认值限制:任何非基本类型的元素,无论是在源代码声明中还是在注解接口中定义默认值时,都不能使用null作为其值;因此,当我们向表达某个元素缺失时,只能采用一些特殊的值(如:"")来表示该属性不存在值

## 注解深入理解
1. 对于一个注解,其一般有三个流程:
- 定义注解
- 使用注解
- 读取并执行相应流程
2. 注解主要是通过**反射被读取**(Class/Field/Method的getAnnonation()或者isAnnotationPresent()),那么就必须要求定义注解时的元注解必须的存活时间必须到运行期:
```
@Retention(RetentionPolicy.RUNTIME)
```
3. 注解不支持继承,不能使用extends来继承@interface