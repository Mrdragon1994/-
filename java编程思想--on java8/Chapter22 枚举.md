[toc]
# 枚举
## 基本enum特性
1. 定义枚举
```
public enum Shrubbery {
    GROUND(1, "123"), CRAWLING(2, "234"), HANGING(3, "345");
    private int age;
    private String name;

    Shrubbery(int age, String name) {
        this.age = age;
        this.name = name;
    }
}
```
2. 通过枚举的名调用values()可以获取到一个枚举的数组;<br>每一个枚举子类调用ordinal()返回的int值代表这个enum实例在枚举种声明的顺序,从0开始;<br>可以使用==来比较enum实例;**编译器会自动提供equals()和hashcode()方法**;<br>Enum类默认实现了Comparable和Serializable接口,因此默认具备了compareTo()方法;<br>调用getDeclaringClass()方法可以获知其所属的enum类;<br>name()方法则返回enum实例声明时的名字,和使用toString()是一样的效果;<br>valueOf()在Enum的一个static方法,它根据给定的名字返回相应的enum实例,若不存在,则抛出异常;
```
    @Test
    public void test1() {
        for (Shrubbery s : Shrubbery.values()) {
            System.out.println(s.ordinal());
            System.out.println(s.name());
            System.out.println(s.getDeclaringClass());
            System.out.println(s.getClass());
            //compareTo返回0代表相等,返回负数代表s在入参的前面(返回负几就代表前几个位置,反之正数代表之后)
            System.out.println(s.compareTo(Shrubbery.HANGING)); 
            System.out.println(s == Shrubbery.HANGING);
            System.out.println(s.equals(Shrubbery.HANGING));
        }

        Shrubbery s =  Enum.valueOf(Shrubbery.class, "HANGING");
        System.out.println(s);
    }
=====>
输出:
0
GROUND
class com.cql.test.enumtest.Shrubbery
class com.cql.test.enumtest.Shrubbery
-2
false
false
1
CRAWLING
class com.cql.test.enumtest.Shrubbery
class com.cql.test.enumtest.Shrubbery
-1
false
false
2
HANGING
class com.cql.test.enumtest.Shrubbery
class com.cql.test.enumtest.Shrubbery
0
true
true
HANGING
```
## 枚举种添加方法
1. 枚举除了不能继承自一个enum之外,我们可以将enum看作是一个常规的类;其可以有属性,构造方法,普通方法,静态方法甚至main()方法;(静态方法只能由枚举类名调用其实例无法调用)
```
public enum OzWitch {
    // 实例必须在方法前声明,且最后一个实例必须加**分号**
    WEST("Miss Gulch, aka the Wicked Witch of the West"),
    NORTH("Glinda, the Good Witch of the North"),
    EAST("Wicked Witch of the East, wearer of the Ruby " +
            "Slippers, crushed by Dorothy's house"),
    SOUTH("Good by inference, but missing");
    private String description;
    // 构造方法必须是包级或者private访问权限
    private OzWitch(String description) {
        this.description = description;
    }
    public String getDescription() { return description; }
    public static void main(String[] args) {
        for(OzWitch witch : OzWitch.values())
            System.out.println(
                    witch + ": " + witch.getDescription());
    }
}
=====>
输出:
WEST: Miss Gulch, aka the Wicked Witch of the West
NORTH: Glinda, the Good Witch of the North
EAST: Wicked Witch of the East, wearer of the Ruby
Slippers, crushed by Dorothy's house
SOUTH: Good by inference, but missing
```
2. 覆盖其他方法
可以在枚举中重写其他方法,如:toString():
```
import java.util.stream.*;
public enum SpaceShip {
    SCOUT, CARGO, TRANSPORT,
    CRUISER, BATTLESHIP, MOTHERSHIP;
    @Override
    public String toString() {
        String id = name();
        String lower = id.substring(1).toLowerCase();
        return id.charAt(0) + lower;
    }
    public static void main(String[] args) {
        Stream.of(values())
         .forEach(System.out::println);
    }
}
=====>
输出:
Scout
Cargo
Transport
Cruiser
Battleship
Mothership
```
3. switch语句现支持enum,默认情况下,可以不给default语句;但是如果我们在case语句中调用了return,那么就需要写default;
4. 枚举类比较特殊:**编译器会为其提供values()/valueOf()方法,并且编译器会把枚举类标记为final,因此无法继承一个Enum**;
5. 由于values()是编译器插入到enum中定义的static方法,因此我们如果将一个enum实例向上转型为Enum,那么values()方法将不可用,但是我们可以通过调用Enum.class.getEnumConstants()获取其enum实例;
6. 如5所讲,getEnuConstants()是定义在Class上的方法,因此我们可以对不是枚举的类调用该方法:
```
public class NonEnum {
    public static void main(String[] args) {
        Class<Integer> intClass = Integer.class;
        try {
            for(Object en : intClass.getEnumConstants())
                System.out.println(en);
        } catch(Exception e) {
            System.out.println("Expected: " + e);
        }
    }
}
=====>
输出:
Expected: java.lang.NullPointerException
```
## 实现而非继承
1. 所有的enum都继承自java.lang.Enum类,由于Java不支持多重继承,所以我们定义的枚举不能在继承其他类;
2. enum可以实现一个或多个接口.