[toc]
# 反射
## 什么是反射
- 反射是Java特性之一,它可以让运行中的Java程序获取自身的信息,并且可以操作类或者对象的内部属性.
- 利用new创建对象,是Java的静态编译,是在编译时就确定了类型并且绑定了对象的;利用反射创建对象,是在运行时菜加载对象及其属性的,是在编译期未知的;
- 反射的核心是JVM在运行时才会动态加载类,调用方法和访问属性,它不需要在编译期知道运行的对象是谁;
- 反射主要提供的功能有：
    1. 在运行时判断任意一个对象所属的类;
    2. 在运行时构造任意一个类的对象;
    3. 在运行时判断任意一个类所具有的成员变量和方法;
    4. 在运行时调用任意一个对象的方法。
    
## 反射的优缺点
### 优点
- 可以在运行期对类型进行判断,动态类加载等操作;
- 提高代码的灵活度,JDBC动态加载数据库。

### 缺点
- 性能问题:由于反射是在运行期间动态加载的,因此JVM无法对代码进行优化;
- 安全限制:使用反射必须要求程序在一个没有安全限制的环境中运行;
- 内部暴露:反射允许访问私有属性或方法。

## 具提使用
### Class
1. Class存放着对应类型的==运行时==信息,JVM会为所有类型维护一个java.lang.Class对象,该Class对象存放着有关该对象的运行时信息.
2. 需要注意的是:Class是一个泛型类,Class<T>;
3. 获取Class的方式：
```
@Test
    public void test1() throws ClassNotFoundException {
        //第一种方式[类名.class]:Object.class
        Class<StudentForReflaction> class1 = StudentForReflaction.class;
        System.out.println(class1);
        //第二种方式[对象.getClass()]
        StudentForReflaction studentForReflaction = new StudentForReflaction();
        Class<? extends StudentForReflaction> class2 = studentForReflaction.getClass();
        //第三种方式[Class.forName(全路径名)]
        Class<?> class3 = Class.forName("com.cql.test.file.StudentForReflaction");
        //需要注意的是:只有第一种方法能够准确获得泛型T的类型，其余两种方式均不可以,因此得到的就是Class<?>
    }
```
4. 获取父类Class
```
通过getSupperClass()方法获取父类的Class;
@Test
    public void test2() {
        Class<Integer> aClass = Integer.class;
        System.out.println(aClass.getSuperclass());
        System.out.println(aClass.getSuperclass().getSuperclass());
        System.out.println(aClass.getSuperclass().getSuperclass().getSuperclass());
        System.out.println(aClass.getSuperclass().getSuperclass().getSuperclass().getSuperclass());
    }
======>
输出:
class java.lang.Number
class java.lang.Object
null
Exception in thread "main" java.lang.NullPointerException
```
可以看出Integer的父类是Number,Number的父类是Object,而Object没有父类,故返回null.
5. 当我们拿到一个Class时,可以获取到其属性,方法等,
> Class的方法中含有Declared时,表明这个方法能够取出当前类所有构造函数（包括私有,公开，不包括继承类）的Field,Method和Constructor.
>

>方法中不带Declared时,支持取出包括继承,公有的Filed,Method,Constructor
>

### Filed
1. 获取Filed
- Field getField(String name)
    根据字段名来获取某个**public**的Filed
- Field[] getFields()
    获取所有的public的Field
- Field getDeclaredField(String name)
    根据字段名获取当前类的某个Field
- Field[] getDeclaredFields()
    获取当前类的所有Field
```
public void test3() throws NoSuchFieldException {
        Class aClass = Student.class;
        //getField(String name)能获取到当前类,父类的公有和私有属性
        //当前类的公有属性,可以成功获取
        Field field = aClass.getField("score");
        //当前类的私有属性,不能成功获取
        Field field1 = aClass.getField("grade");
        //父类的公有属性,可以成功获取
        Field field2 = aClass.getField("name");
        
        //getDeclaredField(String name)可以获取当前类的公有和私有属性
        //当前类的公有属性,能成功获取
        Field field3 = aClass.getDeclaredField("score");
        //当前类的私有属性,能成功获取
        Field field4 = aClass.getDeclaredField("grade");
        //父类的公有属性,不能成功获取
        Field field5 = aClass.getDeclaredField("name");
    }
```
2. 获取Filed的信息
当拿到一个Field时,一个Field对象包含了一个字段的所有信息:
- getName(): 返回字段名称
- getType(): 返回字段类型,是一个Class实例
- getModifiers(): 返回字段的修饰符,是一个int,不同的bit代表不同的含义
- getAnnotation(): 获得属性身上的注解

可以使用Modifier.isFinal/isPublic/isPrivate来判断获得的int值
```
@Test
    public void test4() throws NoSuchFieldException {
        Class aClass = Student.class;
        Field field = aClass.getField("score");
        System.out.println(field.getName());
        System.out.println(field.getAnnotation(NotNull.class));
        System.out.println(field.getType());
        int modifier = field.getModifiers();
        System.out.println(Modifier.isFinal(modifier));
        System.out.println(Modifier.isPublic(modifier));
    }
=====>
输出:
score
@javax.validation.constraints.NotNull(message={javax.validation.constraints.NotNull.message}, groups=[], payload=[])
int
false
true
```
3. 获取字段属性的值
Field.get(对象)
即可获得绑定在这个对象身上的该属性的值(value)
```
@Test
    public void test4() throws NoSuchFieldException, IllegalAccessException {
        Class aClass = Student.class;
        Student student = new Student();
        Field field = aClass.getField("score");
        System.out.println(field.get(student));
    }
===>
输出:
26
```
当属性为私有时,我们通过Field获取其属性值时,会被提醒错误:
```
java.lang.IllegalAccessException: Class com.cql.test.file.ReflactionTest can not access a member of class com.cql.test.file.Student with modifiers "private"
```
此时,我们需要将该field授予访问权限;
```
public void test4() throws NoSuchFieldException, IllegalAccessException {
        Class aClass = Student.class;
        Student student = new Student();
        Field field1 = aClass.getDeclaredField("grade");
        field1.setAccessible(true);
        System.out.println(field1.get(student));
    }
=====>
输出:
100
```
4. 设置字段值
Field.set(对象, value)
通过Field实例既可以获取指定字段的值,也可以设置指定字段的值.
```
public void test4() throws NoSuchFieldException, IllegalAccessException {
        Class aClass = Student.class;
        Student student = new Student();
        Field field1 = aClass.getDeclaredField("grade");
        field1.setAccessible(true);
        field1.setInt(student, 12);
        System.out.println(field1.get(student));
    }
=====>
输出:
26 //原本的值是100
```
### Method
1. Method getMethod(String name, Class...paramTypes): 根据方法名和参数类型获取自己和基类public的Method;
2. Method[] getMethods():
获取自己和基类所有的public的Method
3. Method getDeclaredMethod(String name, Class paramTypes)
根据方法名和参数类型获取自己的所有修饰符下的Method
4. Method[] getDeclaredMethods()
获取当前类的所有的Method
```
public void test6() throws NoSuchMethodException {
        Class aClass = Student.class;
        //getMethod()获取本类和基类中public的方法
        //获取方法名为getScore,无参的方法
        System.out.println(aClass.getMethod("getScore"));
        //获取方法名为setScore,参数为int类型的方法
        System.out.println(aClass.getMethod("setScore", int.class));
        //获取基类的方法
        System.out.println(aClass.getMethod("getName"));
        //获取本类的静态方法
        System.out.println(aClass.getMethod("aaa"));

        //getDeclaredMethod获取本类中的方法,不限定权限修饰符
        //获取本类中方法名为getGrade,无参的私有方法
        System.out.println(aClass.getDeclaredMethod("getGrade"));
    }
=====>
输出:
public int com.cql.test.file.Student.getScore()
public void com.cql.test.file.Student.setScore(int)
public java.lang.String com.cql.test.file.Person.getName()
public static void com.cql.test.file.Student.aaa()
private int com.cql.test.file.Student.getGrade()
```
2. 获取Method身上的信息
- getName(): 返回方法的名称
- getReturnType(): 返回方法的返回值类型,是一个Class实例,如:String.class
- getParameterTypes():
返回方法的参数类型,是一个Class[]
- getModifier(): 返回方法的权限修饰符,是一个int类型的值,不同的bit代表了不同的权限
- getAnnotation(): 返回方法上的注解
```
public void test7() throws NoSuchMethodException {
        Class aClass= Student.class;
        Method method = aClass.getDeclaredMethod("setGrade", int.class);
        method.setAccessible(true);
        System.out.println(method.getName());
        System.out.println(method.getReturnType());
        System.out.println(Arrays.toString(method.getParameterTypes()));
        System.out.println(method.getParameterCount());
        System.out.println(method.getAnnotation(NotNull.class));
        int modifier = method.getModifiers();
        System.out.println(Modifier.isPrivate(modifier));
        System.out.println(Modifier.isStatic(modifier));
    }
=====>
输出:
setGrade
void
[int]
1
@javax.validation.constraints.NotNull(message={javax.validation.constraints.NotNull.message}, groups=[], payload=[])
true
false
```
3. 调用方法
- 调用普通方法
```
public void test8() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        //对象
        String str = "山重水复疑无路";
        //类的Class实例
        Method method = String.class.getMethod("substring", int.class);
        Method method1 = String.class.getMethod("substring", int.class, int.class);
        //该方法绑定到哪个对象身上,以及参数是什么
        System.out.println(method.invoke(str, 3));
        System.out.println(method1.invoke(str, 1 , 5));
    }
=====>
输出:
复疑无路
重水复疑
```
- 调用静态方法
由于静态方法的调用本就可以直接被类调用而不经过对象,因此利用反射调用静态方法时,无需指定实例对象,invoke()方法传入的第一个参数为null或者""
```
 public void test9() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        //Integer.parseInt()是一个静态方法
        Method method = Integer.class.getMethod("parseInt", String.class);
        System.out.println(method.invoke(null, "123"));
        System.out.println(method.invoke("", "245"));
    }
=====>
输出:
123
245
```
4. 多态
父类Person中定义了一个sayHello()方法,子类Student继承了父类重写了sayHello();当我们从父类Class获取的Method作用到子类实例时,调用的方法是子类的方法,**即仍旧遵循多态原则,总是调用实际类型的覆盖方法**.
```
class Person {
    public void sayHello() {
        System.out.println("sayHello From Person");
    }
} 
=====
class Student extends Person {
    @Override
    public void sayHello() {
        System.out.println("sayHello From Student");
    }
}
=====
@Test
    public void test10() throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {
        Class aClass = Person.class;
        Method method = aClass.getMethod("sayHello");
        method.invoke(new Student());
    }
=====>
输出:
sayHello From Student
```
### Constructor
1. 获取Constructor
- Constructor getConstructor(Class...paramTypes): 根据参数类型获取自己和基类的public的构造函数
- Constructors getConstructor(): 获取自己和基类的所有public的构造函数
- Contsructor getDeclaredConstructor(Class...paramTypes): 根据参数类型获取自己所有权限修饰符的构造方法
- Constructors getDeclaredConstructors(): 获取自己类中所有权限修饰符的构造函数
```
@Test
    public void test11() throws IllegalAccessException, InstantiationException, NoSuchMethodException, InvocationTargetException {
        Class aClass = Student.class;
        //利用无参构造创建实例,需要强转
        Student student = (Student) aClass.newInstance();
        //获取一个int类型参数的构造函数,需要强转
        Constructor constructor = aClass.getConstructor(int.class);
        Student student1 = (Student) constructor.newInstance(12);
        //获取一个String类型参数的构造函数,需要强转
        //这个构造函数是private,故一方面要用Declared,另一方面需要设置访问权限为true,方才可以创建实例
        Constructor constructor1 = aClass.getDeclaredConstructor(String.class);
        constructor1.setAccessible(true);
        Student student2 = (Student) constructor1.newInstance("老师");
        //调用三个参数的构造函数,需要强转
        Constructor constructor2 = aClass.getConstructor(int.class, int.class, String.class);
        Student student3 = (Student) constructor2.newInstance(12, 98, "医生");
    }
```
2. 通过代码可以得到以下几个结论：
- 通过Class实例直接调用newInstance()获取到的是无参构造函数;
- 获取有参构造函数时需要通过Class实例调用getConstructor()等方法;然后通过有参构造器调用newInstance(Object...params)创建对象实例;
- 如果获得的是非public的构造器,那么需要先设定访问权限为true才可以.

### Interface
通过调用Class实例的getInterfaces()可以获取其实现的所有接口.
```
 @Test
    public void test12() {
        Class aClass = Student.class;
        System.out.println(Arrays.toString(aClass.getInterfaces()));
    }
=====>
输出:
[interface java.io.Serializable, interface java.lang.Comparable]
```