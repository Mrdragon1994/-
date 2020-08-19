[toc]
# 简单泛型类
1. 泛型定义中的T,E称为类型参数；
2. 泛型类(generic class)就是将一个或多个泛型定义在**类**上：
```
public class Holder<T> {
    //泛型定义在类上，需要写在类名的后面,并用<T>加以显示
    public T a;
    public Holder() {
        //需要注意的是：构造函数是不加任何泛型<>的
    }
    public Holder(T a) {
        this.a = a;
    }
    public void set(T a) {
        this.a = a;
    }
    public T get() {return a;}
    
    public static void main(String[] args) {
        Holder<Automobile> holder = new Holder<>();
        //JDK7以后，右侧就不必再次申请泛型了，叫做“砖石语法”
        holder.set(new Automobile());
        Automoblie auto = holder.get();
    }
}
```
3. 泛型接口,如下定义一个泛型接口
```
public interface Base<T> {
    
}
```
若此时有一个实现类实现了该接口,需要注意的是,实现类要么写成泛型要么直接写明实际类型:
```
public class Sub<T> implements Base<T> {
    
}
=====
或者:
public class Sub implements Base<String> {
    
}
```
4. 继承一个泛型类
处理方式和实现一个泛型接口相同;
```
public class SubClass<T> extends BaseClass<T> {
    
}
=====
public class SubClass extends BaseClass<Integer> {
    
}
```

# 一个元组类库
1. 元组：它是将一组对象直接打包存储于单一对象中，可以从该对象读取其中的元素，但**不允许向其中存储新对象**；
2. 通常，元组可以具有任意长度并且元组中的对象是不同类型的；
```
// onjava/Tuple2.java
package onjava;

public class Tuple2<A, B> {
    public final A a1;
    public final B a2;
    public Tuple2(A a, B b) { a1 = a; a2 = b; }
    public String rep() { return a1 + ", " + a2; }

    @Override
    public String toString() {
        return "(" + rep() + ")";
    }
}
```

# 泛型方法
1. 如简单泛型中，定义了泛型类，泛型还可以加在方法上；
2. 类本身可能是泛型,也可能不是,但是它与方法是否是泛型没有关系;
3. 泛型方法独立于泛型类存在,且将单个方法泛型化比将整个类泛型化更加清晰易懂;
4. **静态方法无法访问类上定义的泛型,如果静态方法操作的引用数据类型不确定时,必须要将泛型定义在方法上**;
```
public class StaticGenerator<T>() {
    //public static void show(T t) //{
        //这样定义会报错,使用泛型的静态方法需要额外的泛型声明，如下
    //}
    public static <T> void show(T t) {
        //Right 
    }
}
```
5. **要定义泛型方法,需要把泛型参数列表<T>放在修饰符后面,返回值前面, <T>可以标示这个方法是一个泛型方法**;
```
public class GenericMethods {
    public <T> T method(T t) {
        //<T>代表这是一个泛型方法,该方法的入参需要泛型T,而返回值T代表这个方法的返回值类型是T
    }
    
    public T get() {
        //该方法没有<T>,表明该方法不是泛型方法,也就是意味着该方法的参数不需要泛型,但是该方法可以返回T类型的数据
    }
    
    public <T, K> void method1(T t, K v) {
        //同理,表明这个方法是一个泛型方法,其入参需要两种类型的参数,返回值的void
    }
}
```
6. ***需要注意的是:泛型类需要是实例化的时候就指定类型参数,而泛型方法,只有在调用该方法的时候,才需要指定参数的类型***；

# 变长参数和泛型方法
```
public class GenericVarargs {
    @SafeVarargs
    public static <T> List<T> makeList(T... args) {
        List<T> result = new ArrayList<>();
        for (T item : args)
            result.add(item);
        return result;
    }
}
```
@SafeVarargs 注解保证我们不会对变长参数列表进行任何修改。

# 类型变量(泛型)的限定
```
public class ArrayAlg {
    public static <T> T min(T[] a) {
        T smallest = a[0];
        for(int i = 1; i < a.length; i++) {
            if (smallest.compareTo(a[i]) > 0) smallest = a[i];
            return smallest;
        }
    }
}
```
由于smallest的类型是T,表明T可以是任意类型的,但是为了确保T类型拥有compareTo()方法,因此,我们必须对T类型进行一定的限定.
那就是只有实现了Comparable接口的类才可以,因此,该泛型方法应该写成：
```
public static <T extends Comparable> T min(T[] a) {
    .....
}
```
1. 当有多个类型限定时,类型限定用&来分隔:
```
T extends Comparable & Serializable
```
2. 类型变量用逗号分隔
```
public <T, U> void f(T t, U u) {
    ...
}
```

# 泛型擦除
1. 泛型的擦除的含义表明泛型只存在与编译器,运行期间泛型便会消失,如：List<String>的类型是List.class而不是List<String>.class;
```
@Test
    public void test1() {
        List<String> stringList = new ArrayList<>();
        List<Integer> integerList = new ArrayList<>();
        System.out.println(stringList.getClass() == integerList.getClass());
        System.out.println(stringList.getClass());
    }
```
```
输出：
true
class java.util.ArrayList
```
2. 擦除类型变量,将其替换为限定类型(**无限定的变量用Object**)
```
public class Pair<T> {
    private T first;
}
================>
//被擦除后为:
public class Pair {
    private Object first;
}
```
3. 如果有限定类型,就可以用**第一个**相应的限定类型来替换：
```
public class Interval<T extends Comparable & Serializable> {
    private T lower;
    private T upper;
}
=========>
//则被替换为:
public class Interval {
    private Comparable lower;
    private Coparable upper;
}
```
当然如果把泛型限定类型写为:
T extends Serializable & Comparable,
那么泛型擦除后就是：
```
private Serializable;
```
但是Serializable只是一个标签接口,里面不含有任何方法,因此这种接口应该放在泛型限定类型列表的最后。

# 迁移兼容性
1. 擦除主要的理由是从非泛化代码到泛化代码的转变过程,以及在不破坏现有类库的情况下将泛型融入到Java语言中;

# 约束与局限性
泛型并不是Java独有的特色,但是Java为了兼容以前的版本,对泛型做了一些限制。
## 不能使用基本类型实例化类型参数
不能用类型参数代替基本类型,就是泛型<T>中的T,不能使用基本类型来替换。
如:只有Pair<Double>而没有Pair<double>

## 运行时类型查询只适用于原始类型
所谓的类型查询,就是调用instanceof或者调用getClass()这样的方法时,会对instanceof后面和getClass()前面的进行类型检查。**而原始类型是指不带泛型时候的类型。**
```
@Test
    public void test2() {
        GenertorTest<String> a = new GenertorTest<>();
        //因为GenertorTest<String>在运行期间会被泛型擦除为Genertor,
        //因此,使用如下两个带有泛型的判定是无法通过编译的
        System.out.println(a instanceof GenertorTest<String>);
        System.out.println(a instanceof GenertorTest<T>);
        //这样的判定输出:true
        System.out.println(a instanceof GenertorTest);
    }
```

## 不能创建参数化类型的数组
例如,**不能**创建如下数组:
```
Pair<String>[] table = new Pair<String>[10]
```
如果有必须,可以使用如下方式:
```
ArrayList:ArrayList<Pair<String>>
```

## 不能实例化类型变量
不能使用像new T(...), new T[...]或者T.class这样的表达式中的类型变量,如下,是无法通过编译的:
```
public class GenertorTest<T> {

    private T t;
    public GenertorTest() {
        //无法编译
        t = new T();
    }
}
```
因为T没有给点限定的情况下,T是要被Object取代的,而我们又不想让其调用new Object()。
```
public GenertorTest(T t) {
        this.t = t;
    }

    public static <T> GenertorTest<T> makeGenetorTest(Supplier<T> constr) {
        return new GenertorTest<>(constr.get());
    }
    
调用时：
@Test
    public void test3() {
        GenertorTest<String> genertorTest1 = GenertorTest.makeGenetorTest(String::new);
    }
```

## 泛型类的静态上下文中类型变量无效
静态成员变量和静态方法的**返回值**不能是泛型T,如下,是无法通过编译的:
```
private static T a; //无法编译

private static T a() {
    //无法编译
}
//不能混为一谈的是,静态方法可以是泛型方法,但是返回值不能是泛型,可以是其他或者是包含了泛型的类
public static <T> void b() {
    //Right
}
public static <T> List<T> c() {
    //Right
}
```

## 不能抛出或者捕获泛型类的实例
既不能抛出也不能捕获泛型类对象,甚至泛型类无法继承Expection或实现Throwable。
如下,是无法编译的:
```
public class GenertorTest<T> extends Exception{ 
    //无法编译
}
```
---
```
public class GenertorTest<T extends Exception> extends Exception {
    //无法编译
}
```
不能抛出泛型类对象,因为抛出异常通常都是new XXException(),而new T()本身就是不支持的,
而捕获异常如：
```
catch(T e) //无法编译
```
也是无法编译的。
但是,使用类型变量是合法的：
```
public static <T extends Throwable〉void doWork(T t) throws T { // OK
try {
do work
}
catch (Throwable real Cause)
{
t.initCause(real Cause) ;
throw t ;
    }
}
```
因为类型变量T会被擦除为Throwable.

# 泛型类型的继承规则
1、 无论S和T有什么联系(S和T无联系,S继承T,T继承S等),通常Pair<T>和Pair<S\>都没有什么联系;

# 通配符类型
## 通配符概念
1. 通配符类型中,允许类型参数变化,如:通配符类型:
Pair<? extends Employee>
Pair<? super Employee>
2. 可以把
```
? extends Employee
的set和get看成:
? extends Employee get()
void set(? extends Employee)
```
get()的时候必然会获得一个Employee或其子类,但是set()的时候编译器无法知道存入的元素是否是Employee的子类或其本身,因此? extends Emplooy是无法调用set()方法的;
```
? super Employee
的set和get可以看成:
? super Employee get()
void set(? super Empployee)
```
get()的时候会获得Employee或其超类,因此无法准确获得其类型,故只能将它赋值给一个Object;
但是set()的时候可以set进入Employee或其子类,**因为?是Employee的基类,也必定是Employee子类的基类**；

**总结：带有超类限定的通配符可以向泛型对象写入;带有子类限定的通配符可以从泛型对象读取。**
3. 值得注意的是:**通配符不是类型变量,因此,不能在编写代码中使用"?"作为一种类型
因此，如下方法不是泛型方法：
```
public static void swap(Pair<?> p)
```

## 无限定通配符?
Pair<?>,无限定通配符对应的set和get方法如下:
```
? get()
void set(?)
```
getFirst返回值只能赋给一个Object,而set方法是不能够被调用的,甚至不能用Object调用;但是可以调用:
```
set(null)
```

## 通配符捕获
?不能作为类型变量,但是T可以明确指定一种类型,因此我们可以使用T来捕获?
```
//通配符方法
public static void swap(Piar<?> p) {
    ? t = p.getFirst();
    p.setFirst(p.getSecond);
    p.setSecond(t);
}
//使用通配符捕获方法
public static <T> void swapHelper(Pair<T> p) {
    T t = p.getFirst();
    p.setFirst(p.getSecond());
    p.setSecond(t);
}
//进而改进为：
public static void swap(Pair<?> p) {
    swapHelper(p);
}
```
# 探讨T和?
1. 在泛型中,T代表的是一种明确类型;而?代表的是一个通配;
2. T用在定义一个泛型类或者泛型方法上;?常用在引用一个定义好的泛型类上或者作为参数形式;
要明确知道：Object是String的父类,但是List<Object>不是List<String>的父类,因此需要用通配符;
```
public class Plate<T> {  //定义了一个泛型类
    
    //定义了一个泛型方法
    public <T> void method() {
        
    }
}

//定义一个普通类
class Fruit {}
//定义一个实现类
class Apple extends Fruit {}

//?此时通配符就可以用在实例引用上了(此时是有问题的)
Plate<Fruit> plate = new Plate<Apple>();

//作为参数形式
public void get(List<? extends Fruit> list) {
    
}
```
3. 虽然Fruit是Apple的基类，存在继承关系,但是Plate<Fruit>和Plate<Apple>是不存在任何关系的；
4. 如果想让plate引用成立,那么就可以使用通配符?或者限定通配符
```
Plate<? extends Fruit> palte = new Plate<Apple>();
```
**此时Plate<? extends Fruit>是Plate<Fruit>和Plate<Apple>的基类,但是虽然是基类,但是JVM仍旧无法预知?是什么类型,因此上界通配符,适用于get(),但不能set();**

而且get()操作得到的是Fruit及其基类:
```
Fruit fruit = plate.get()
Object object = plate.get()
Apple apple = plate.get() //ERROR
```
5. 下界通配符与此相对应,是一个适合set()但不适合get()的操作;
而且set()时适合存入自己及其实现类:
```
Plate<? super Fruit> plate2 = new Palte<Fruit>();
plate2.set(new Fruit());
plate2.set(new Apple());
plate2.set(new Object()); //ERROR
```
6. 无界限通配符?
```
Plate<?> plate = new Plate<Apple>();
//无界限通配符只能set null,而其get的也都是Object
```