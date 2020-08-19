[toc]
# Chapter4 共享模式之管程
## 共享问题
```
private static int counter = 0 ;

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            for (int i = 0; i < 5000; i++) {
                counter++;
            }
        },"t1");

        Thread t2 = new Thread(() -> {
            for (int i = 0; i < 5000; i++) {
                counter--;
            }
        },"t2");

        t1.start();
        t2.start();
        t1.join();
        t2.join();
        log.debug("{} ", counter);
    }
    =====>
    输出:
    不确定(每次不一样)
```
问题分析:<br>
i++/i--/--i/++i转化为字节码后是4条指令构成的;

  - 从主存中获取i
  - 在线程内部准备数i
  - 在线程内部给i+1/-1
  - 将结果推向到主存中

负数情况：

![1](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\1.jpg)

正数情况：

![1597129927(1)](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\1597129927(1).jpg)

## 临界区Critical Section和竞态条件Race Condition

1. 在一段代码块内,存在对**共享资源**的多线程读写操作,称这块代码为**临界区**；

   ```java
   static int counter = 0;
   static void increment() { //临界区
       counter++; 
   }
   static void decrement() { //临界区
       counter--;  
   }
   ```

2. 竞态条件Race Condition: 多个线程在临界区执行,由于代码**执行序列不同**而导致的结果无法预测,我们便称这种情况出现了竞态条件；



## 阻塞式解决临界区和竞态条件方案---synchronized和Lock

### Synchronized关键字

1. synchronized俗称**对象锁**，它采用互斥的方式让同一个时刻至多只有一个线程拥有对象锁;从而保证拥有锁的线程可以安全执行临界区中的代码;

2. 语法:

   ```java
   synchronized(对象) {
   	//临界区
   }
   =====>
   需要保证的是多个线程访问的是同一个对象,才能保证synchronized发挥作用;
   ```

   ```java
   private static int counter = 0 ;
       private static final Object object = new Object();
   
       public static void main(String[] args) throws InterruptedException {
           Thread t1 = new Thread(() -> {
               synchronized (object) {
                   for (int i = 0; i < 5000; i++) {
                       counter++;
                   }
               }
           },"t1");
   
           Thread t2 = new Thread(() -> {
               synchronized (object) {
                   for (int i = 0; i < 5000; i++) {
                       counter--;
                   }
               }
           },"t2");
   
           t1.start();
           t2.start();
           t1.join();
           t2.join();
           log.debug("{} ", counter);
       }
   =====>
   输出:
   15:29:27.711 [main] c.SharedTest - 0 
   ```

   ![](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\1597131429(1).jpg)



![](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\1597131717(1).jpg)
3. synchronized是利用对象锁保证了临界区内代码的原子性;

4. 改进思考:

   - 如果把synchronized(obj)放在for循环外面,如何理解？

     答:那么就是获得锁的那个线程会将整个for循环执行完毕才退出锁;因为每次被synchronized锁住的**代码块必须全部执行完才能退出**;

   - 如果t1线程synchronized(obj1),t2线程synchronized(obj2),会发生什么？

     答:无法保证原子性,因为这是两个锁对象,是两个不同的锁

   - 如果t1加了synchronized(obj)而t2没有加,会发生什么？

     答:无法保证原子性,因为t2线程压根无需获得锁就能执行counter--

5. 把上述代码用面向对象的思路进行优化:

   ```java
   public static void main(String[] args) throws InterruptedException {
           Room room = new Room();
           Thread t1 = new Thread(() -> {
               for (int i = 0; i < 5000; i++) {
                   room.increment();
               }
           },"t1");
   
           Thread t2 = new Thread(() -> {
               for (int i = 0; i < 5000; i++) {
                   room.decrement();
               }
           }, "t2");
           t1.start();
           t2.start();
           t1.join();
           t2.join();
           System.out.println(room.getCounter());
       }
   
   class Room {
       private int counter = 0;
   
       public void increment() {
           synchronized (this) {
               counter++;
           }
       }
   
       public void decrement() {
           synchronized (this) {
               counter--;
           }
       }
   
       public int getCounter() {
           synchronized (this) {
               return counter;
           }
       }
   }
   ```

6. 语法2:方法上的synchronized

   - 放在成员方法上

   ```java
   class Test {
   	public synchronized void test() {
   		//do something
   	}
   }
   =====>等价于
   class Test {
       public void test() {
           synchronized(this) {
               
           } 
       }
   }
   ```

   - 放在静态方法上

     ```java
     class Test {
         public synchronized static void test() {
             //do something
         } 
     }
     =====>等价于
     class Test {
         public static void test() {
             synchronized(Test.class) {
                 //do somethind
             }
         }
     }
     ```



## 变量的线程安全性分析

1. 成员变量和静态变量是否线程安全？

   - 如果它们没有共享,那是线程安全的;
   - 如果它们被共享了,根据它们的状态能否改变分为：
     - 如果只有读操作,那么是线程安全的;
     - 如果有读写操作,则这段代码是临界区,需要考虑线程安全问题

2. 局部变量是否是线程安全？

   - 局部变量是线程安全的；
   - 但是局部变量引用的对象则未必：
     - 如果该对象没有逃离方法的作用访问范围,那么它是线程安全的;
     - 如果该对象逃离方法的作用范围,那么它不是线程安全的.

   ```java
   //局部变量的线程安全问题
   public static void test() {
       int i = 10;
       i++;
       //这里的i是否需要考虑线程安全问题？
   }
   //每个线程执行该方法时,都会在自己的虚拟机栈中创建一个栈帧,而i时一个局部变量,因此i是栈帧中私有的变量,互补干扰
   ```



## 常见线程安全类

- String
- Integer/Boolean
- StringBuffer
- Random
- Vector
- HashTable
- JUC包下的类

说它们是线程安全的,是指多个线程调用他们**同一个实例的某个方法**时,是线程安全的,也可以理解为：

- 它们的单个方法是**原子**的;

  ```java
  Vevtor v = new Vector();
  new Thread(() -> {
  	v.add(1)
  }, "t1").start();
  
  new Thread(() -> {
  	v.add(2)
  }, "t1").start();
  ```

  

- 但是需要注意:多个方法的**组合**就不是原子的了;

  ```java
  HashTable table = new HashTable();
  //线程1,线程2
  if(table.get("key") == null) {
  	table.put("key", value);
  }
  ```

  ![image-20200812092440299](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200812092440299.png)



### 不可变线程安全类

- String

- Integer

  上面的类是不可变类,因为其内部的状态不可以改变,因此它们的方法都是线程安全的;

  如:String类中的replace(),substring()方法也都是返回一个新的String来保证线程安全.

## Monitor概念(Monitor锁是重量级锁)

### 1. Java对象头(32位虚拟机为例)

普通对象:

```ruby
|---------------------------------------------------------------|
|                  Object  Header(64 bits)                      |
|_______________________________|_______________________________|
|         Mark Word(32 bits)    |   Klass Word(32 bits)         |
|-------------------------------|-------------------------------|
```

对象是如何知道自己是什么类型的,是由klass word作为指针去指向类对象

数组对象:

![1597283641(1)](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\1597283641(1).png)

其中Mark Word结构是:

![image-20200813095453600](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200813095453600.png)

Normal状态下：25bits是hashcode，4bits是分代年龄（可以从新生代入驻到老年代的一个tag）,biased_lock是1bit代表是不是偏向锁，2bit是加锁状态.

其中加锁状态：01代表未加锁，00是轻量级锁，10是重量级锁 ，11是GC

剩下的是不同状态下的mark word。

### 1.1 Monitor

1. Monitor被称为**监视器**或者**管程**，每个Java对象的对象头部分的Mark Word都可以关联一个Monitor对象（synchronized(obj)，**一个obj会对应一个Monitor**），如果使用synchronized给对象上锁之后,该对象头的Mark Word中就被设置了指向Monitor对象的指针:此时Mark Word就是上面的Heavyweight Locked状态；

   ![image-20200813150825161](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200813150825161.png)

   - 刚开始Monitor中的Owner为null;
   - 当Thread-2执行synchronized(obj)将会将Monitor的Owner设置为Thread-2，Monitor中只能有一个Owner;
   - 在Thread-2上锁过程中，如果Thread-3,Thread-4,Thread-5也来执行synchronized(obj),就会进入Monitor的EntryList从而进入Blocked状态；
   - Thread-2执行完同步代码块的内容，然后唤醒EntryList中等待的线程竞争锁，竞争时候是**非公平的**;
   - 图中WaitSet中的Thread-0,Thread-1是之前获得过锁，但是条件不满足从而进入WAITING状态，和wait-notify有关

   **注意**：synchronized必须是进入同一个对象的monitor才有上述的效果；

   ​			不加synchronized的对象不会关联Monitor，不会遵从上面的规则。

### 1.2 synchronized的原理(重量级锁的分析过程，轻量级/偏向锁会有区别)

   ```java
   static final Object lock = new Object();
   
   statci int counter = 0;
   
   public static void main(String[] args) {
       synchronized(lock) {
           counter++;
       }
   }
   ```

   主要的一些指令：

   ```ruby
   monitorenter  //将lock对象Mark Word置为Monitor指针
   monitorexit   //将lock队形Mark Word重置为normal状态，并且唤醒EntryList(在monitorenter时,mark word会转变为Heavyweight Locked状态，其中的hashcode,age等属性会存在Monitor中,等到monitorexit时，再还原回来)
   ```

---

   ```ruby
   成员变量counter++的字节码指令：
   getstatic   //拿到counter  
   iconst_1    //准备常数1
   iadd		//加1
   putstatic   //将结果赋值到counter中
   ```

重量级锁中：obj的mark word指向Monitor；Monitor的header存原来的mark word，obj属性指向obj锁对象，owner指向线程。

### 2. synchronized性能进阶

#### 2.1 轻量级锁（轻量级锁没有Blocked说法，因此如果出现了锁膨胀，就要升级为重量级锁）

   ​	应用场景：如果一个对象虽然有多个线程访问，但是多线程访问的时间时错开的，也就是没有竞争关系，那么可以使用轻量级锁优化；轻量级锁对使用者是透明的，语法仍旧是synchronized，也就是synchronized优先使用轻量级锁进行加锁，如果轻量级锁加锁失败了，那么才会改用重量级锁。

   ```java
   static final Object obj = new Object();
   public static void method1() {
       synchronized(obj) {
           method2();
       }
   }
   
   public static void method2() {
       synchronized(obj) {
           //do something
       }
   }
   ```

   ![1597330648(1)](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\1597330648(1).jpg)

   ​	Thread-0的锁记录包括两部分，lock record地址会指向object的mark word，Object reference会指向锁对象；

   ​	Object包括对象头和对象体两部分，对象头包括mark word和klass word，对象体包括一些成员变量等；

   ![](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\微信截图_20200813230211.png)

   PS：此时mark word中的锁状态必须是01（无锁）才能CAS成功

   ![image-20200813230457571](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200813230457571.png)

   ![image-20200813230905877](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200813230905877.png)

   如果是**重入**，就是**同一个线程执行同一个对象的多次synchronized**，那么此时lock Record置位null。

   ![image-20200813231221732](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200813231221732.png)

   ![image-20200813231404040](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200813231404040.png)

#### 2.2 锁膨胀

   ​	如果在尝试加轻量级锁的过程中，CAS操作无法成功，这时一种情况就是其他线程为此对象加上了轻量级锁（表明有竞争），这时需要进行锁膨胀，将轻量级锁转变为重量级锁。

   ```java
   static Object obj = new Object();
   public static void method1() {
       sychronized(obj) {
           //do something
       }
   }
   ```

   ![image-20200813233107864](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200813233107864.png)

   

   ![image-20200813233602182](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200813233602182.png)

   PS：此时，Thread-0的Lock Record中还记录着Object的Mark word信息，但是object中的mark word不再记录Thread-0的lock record了，而是记录了指向Monitor的指针，并且锁状态位置为10

   ![image-20200813233954459](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200813233954459.png)

#### 2.3 自旋优化

   ​	重量级锁竞争的时候，还可以使用自旋来进行优化，让没有获得锁的线程暂时先不用进入到EntryList中，如果当前线程自旋成功（即这时候持锁线程已经推出了同步块，释放了锁），这时当前线程就可以避免阻塞（其实也避免了上下文切换）了。

   ​	![image-20200814094002044](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200814094002044.png)

   

   ![image-20200814094022896](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200814094022896.png)

   需要注意的点：

   - 在Java6之后自旋锁是自适应的，比如对象刚刚的一次自旋擦偶哦成功过，那么认为这次自旋的可能性会高，因此就会多自旋几次，反之，自旋次数变少甚至补自旋；
   - 自旋会占用CPU，如果是单核处理器，自旋就是浪费；只有多核CPU的自旋才有意义；
   - Java 7 之后就不能人为控制自旋功能的开启与否了。

#### 2.4  偏向锁

​	轻量级锁在没有竞争时(只有自己这个线程)，每次**重入**仍然需要执行CAS操作；Java 6引入了偏向锁来做进一步优化，只有第一次使用CAS将线程ID设置到对象的mark word头（此时mark word不再存lock record或者monitor了），之后发现这个线程ID是自己的就表示没有竞争，不用重新CAS，以后只要不发生竞争，这个对象就归线程所有。

![image-20200814100036872](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200814100036872.png)

##### 偏向状态

![image-20200814101050945](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200814101050945.png)

 mark_word中的biased_lock如果为0表示未开启偏向锁，1表示开启偏向锁。

一个对象创建时：

- 如果开启了偏向锁（默认开启），那么对象创建后，mark word的后三位为**101**，这时它的thread,epoch,age都是0；
- 偏向锁默认是延迟的，不会在程序启动时立即生效，如果向避免延迟加载，可以加参数：-xx:BiasedLockingStartupDelay=0来禁用延迟；
- 如果没有开启偏向锁，那么对象创建以后，markword的后三位为001，它的hashcode,age都为0，第一次用到hashcode才会赋值；
- -XX:-UseBiasedLocking/-XX:+UseBiasedLocking  禁用/开启偏向锁

![image-20200814152341750](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200814152341750.png)

  

  解锁后仍旧是加锁时的状态，表示当前锁的偏向锁，偏向了dog这个对象，下次仍旧可以使用。

  **总结**：优先级是偏向锁 --> 轻量级锁 --> 重量级锁

![image-20200814153048462](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200814153048462.png)

  调用了hashcode()就会被禁用了偏向锁,因为对象头中的mark word如果处于偏向锁没有地方存储hashcode码；轻量级锁的hashcode会存在栈帧中，重量级锁存在Monitor中，当解锁会还原到对象头中。

**偏向锁和轻量级锁访问的两个线程是错开的，只是如果第一个线程使用了锁对象后，第二次再使用，如果还是该线程，那么就使用偏向锁；但是如果是另外一个线程使用了，偏向锁就会升级为轻量级锁；如果两个线程对同一个锁发生了竞争，那么锁就会升级为重量级锁**

<u>**撤销偏向锁的几种情况：**</u>

- <u>**调用锁对象的hashcode，因为偏向锁没有空间存hashcode**</u>
- <u>**其他线程使用了该锁对象，说明该锁对象不被某个线程所独有的**</u>
- <u>**调用了wait/notify，说明出现了线程的竞争**</u>

##### 批量重偏向

​	如果对象虽然被多个线程访问，但是没有竞争，这时偏向了线程T1的对象仍有机会重新偏向T2，重偏向会重置对象的Tread ID;

​	当撤销偏向锁阈值超过20次后，JVM会认为偏向出错，不再升级为轻量级锁，而是会在给这些对象加锁时重新偏向其他线程。

##### 批量撤销

​	如果当撤销阈值到40次时(也就是前39个都转变了)，JVM就会将这个类的所有对象都变为了不可偏向，即使是新建的对象，也默认是不可偏向的了。

#### 锁消除

```java
public void b() {
    Object obj = new Object();
    synchronized(obj) {
        //do something
    }
}
//JIT 即使编译对于重复多次执行的字节码再次进行优化，发现obj是一个拘捕变量，是不会被共享的，因此可以无需加synchronized，因此可以进行锁消除
```

默认是开启的，可以使用参数关闭锁消除的优化：

```ruby
java -XX:-EliminateLocks
```



## Wait/Notify

![image-20200814210647001](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200814210647001.png)

#### API介绍

- obj.wait() 让进入object监视器的线程到waitSet等待 ==> 等同于wait(0)；
- obj.wait(long n)
- obj.notify()在object上正在waitSet等待的线程中挑一个唤醒进入到EnteryList中；
- obj.notifyAll()让object上正在waitSet等待的线程全部唤醒

**它们是线程之间进行协作的手段，都属于Object对象的方法，必须获得次对象的锁，才能调用次对象身上的这几个方法。**



#### wait和notify的正确姿势

1. sleep(long n)和wait(long n)的区别

   - sleep()是Thread的静态方法，wait()是Object的实例方法；
   - sleep()不需要强制和synchronized配合使用，但是wait()需要和synchronized配合一起使用；
   - sleep()在睡眠的时候，不会释放锁对象；而wait()在等待的时候会释放对象锁。

   - 两者的共同点：它们的线程状态都是**TIMED_WAITING**
   
2. 模拟一些线程使用共享的对象room，小南线程为工作线程，其需要对变量hasCigarette进行判定：

   step1:

   ```java
   package com.thread.chapter4;
   
   import lombok.extern.slf4j.Slf4j;
   
   import java.util.concurrent.TimeUnit;
   
   @Slf4j(topic = "c.TestCorrectPostureStep1")
   public class TestCorrectPostureStep1 {
       static final Object room = new Object();
       static boolean hasCigarette = false; //有没有烟
       static boolean hasTakeOut = false;
   
       public static void main(String[] args) throws InterruptedException {
           new Thread(() -> {
              synchronized (room) {
                  log.debug("有没有烟？[{}]", hasCigarette);
                  if (!hasCigarette) {
                      log.debug("没有烟，歇会");
                      try {
                          TimeUnit.SECONDS.sleep(3);
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                  }
                  log.debug("有没有烟？[{}]", hasCigarette);
                  if (hasCigarette) {
                      log.debug("可以干活了");
                  }
              }
           }, "小南").start();
   
           for (int i = 0; i < 5; i++) {
               new Thread(() -> {
                   synchronized (room) {
                       log.debug("可以干活了");
                   }
               }, "其他人").start();
           }
   
           TimeUnit.SECONDS.sleep(1);
           new Thread(() -> {
               hasCigarette = true;
               log.debug("烟到了");
           }, "送烟的").start();
       }
   }
   =====>输出:
   22:06:30.619 [小南] c.TestCorrectPostureStep1 - 有没有烟？[false]
   22:06:30.622 [小南] c.TestCorrectPostureStep1 - 没有烟，歇会
   22:06:31.640 [送烟的] c.TestCorrectPostureStep1 - 烟到了
   22:06:33.639 [小南] c.TestCorrectPostureStep1 - 有没有烟？[true]
   22:06:33.639 [小南] c.TestCorrectPostureStep1 - 可以干活了
   22:06:33.639 [其他人] c.TestCorrectPostureStep1 - 可以干活了
   22:06:33.639 [其他人] c.TestCorrectPostureStep1 - 可以干活了
   22:06:33.639 [其他人] c.TestCorrectPostureStep1 - 可以干活了
   22:06:33.639 [其他人] c.TestCorrectPostureStep1 - 可以干活了
   22:06:33.639 [其他人] c.TestCorrectPostureStep1 - 可以干活了
   ```

   分析：

   - 其他干活的线程，必须一直阻塞等待小南线程完成，是因为小南线程中的sleep()不会释放锁对象，导致其他线程一直无法获得锁对象；
   - 小南线程也必须睡够3秒才能醒来，就是条件提前满足，也无法立刻醒来；

3. step2：使用wait()/notify()来解决问题

   ```java
   package com.thread.chapter4;
   
   import lombok.extern.slf4j.Slf4j;
   
   import java.util.concurrent.TimeUnit;
   
   @Slf4j(topic = "c.TestCorrectPostureStep1")
   public class TestCorrectPostureStep1 {
       static final Object room = new Object();
       static boolean hasCigarette = false; //有没有烟
       static boolean hasTakeOut = false;
   
       public static void main(String[] args) throws InterruptedException {
           new Thread(() -> {
              synchronized (room) {
                  log.debug("有没有烟？[{}]", hasCigarette);
                  if (!hasCigarette) {
                      log.debug("没有烟，歇会");
                      try {
                          room.wait();
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                  }
                  log.debug("有没有烟？[{}]", hasCigarette);
                  if (hasCigarette) {
                      log.debug("可以干活了");
                  }
              }
           }, "小南").start();
   
           for (int i = 0; i < 5; i++) {
               new Thread(() -> {
                   synchronized (room) {
                       log.debug("可以干活了");
                   }
               }, "其他人").start();
           }
   
           TimeUnit.SECONDS.sleep(1);
           new Thread(() -> {
               synchronized (room) {
                   hasCigarette = true;
                   log.debug("烟到了");
                   room.notify();
               }
           }, "送烟的").start();
       }
   }
   =====>
   22:29:33.678 [小南] c.TestCorrectPostureStep1 - 有没有烟？[false]
   22:29:33.681 [小南] c.TestCorrectPostureStep1 - 没有烟，歇会
   22:29:33.681 [其他人] c.TestCorrectPostureStep1 - 可以干活了
   22:29:33.681 [其他人] c.TestCorrectPostureStep1 - 可以干活了
   22:29:33.681 [其他人] c.TestCorrectPostureStep1 - 可以干活了
   22:29:33.681 [其他人] c.TestCorrectPostureStep1 - 可以干活了
   22:29:33.681 [其他人] c.TestCorrectPostureStep1 - 可以干活了
   22:29:34.679 [送烟的] c.TestCorrectPostureStep1 - 烟到了
   22:29:34.679 [小南] c.TestCorrectPostureStep1 - 有没有烟？[true]
   22:29:34.679 [小南] c.TestCorrectPostureStep1 - 可以干活了    
   ```

4. notify()只能随机唤醒一个线程，可改完notifyAll();

5. if()只能判定一次，可以改为while()进行多次判定，while()可以避免虚假唤醒

   ```java
    new Thread(() -> {
              synchronized (room) {
                  log.debug("有没有烟？[{}]", hasCigarette);
                  while (!hasCigarette) {
                      log.debug("没有烟，歇会");
                      try {
                          room.wait();
                      } catch (InterruptedException e) {
                          e.printStackTrace();
                      }
                  }
                  log.debug("有没有烟？[{}]", hasCigarette);
                  if (hasCigarette) {
                      log.debug("可以干活了");
                  }
              }
           }, "小南").start();
   ```

   

6. 使用wait()/notify()的正确方式：

   ```java
synchronized(obj) {
   	while(条件不成立) {
           obj.wait();
       }
       //干活代码do something
   }
   
   synchronized(obj) {
       //把条件改为成立；且使用notifyAll();
       obj.notifyAll();
   }
   ```
   
   - 使用while()可以避免虚假唤醒（就是不该唤醒的被唤醒了）；
   - 唤醒时要使用notifyAll()，以防虚假唤醒
   

## Park&Unpark

1. 它们都是LockSupport类中的方法，基本用法如下：

   ```java
   //暂停当前线程
   LockSupport.park();
   
   //恢复某个线程的运行
   LockSupport.unpark(Thread t);
   ```

   调用park()方法的线程状态是WAIT；

   与Object的wait, notify相比：

   - wait，notify和notifyAll必须配合Object Monitor一起使用，而park,unpark不必；
   
   - park & unpark是以线程为单位来【阻塞】和【唤醒】的，而notify只能随机唤醒一个对象身上的某个等待线程，notifyAll是唤醒某个对象身上所有等待的线程；
   
   - park & unpark可以先unpark，再调用park,而wait & notify只能先wait，再调用notify
   
   - 在一个线程内部，如果先用了LockSupport.park()，在其他线程里用interrupt唤醒，想要继续在该线程里使用park()，必须先将中断标记清楚掉，Thread.initerrupted()；
   
     ```java
     package com.thread.chapter4;
     
     import lombok.extern.slf4j.Slf4j;
     
     import java.util.concurrent.TimeUnit;
     import java.util.concurrent.locks.LockSupport;
     
     @Slf4j(topic = "c.LockAndUnpark1")
     public class LockAndUnpark1 {
         public static void main(String[] args) throws InterruptedException {
             Thread thread = new Thread(() -> {
                 log.debug("调用park前");
                 LockSupport.park();
                 log.debug("调用park后");
     
                 log.debug("再次调用park前");
                 LockSupport.park();
                 log.debug("再次调用park后");
             }, "t1");
     
             thread.start();
             TimeUnit.SECONDS.sleep(2);
             thread.interrupt();
         }
     }
     =====>
     输出:
     15:15:01.403 DEBUG [t1] c.LockAndUnpark1 - 调用park前
     15:15:03.403 DEBUG [t1] c.LockAndUnpark1 - 调用park后
     15:15:03.403 DEBUG [t1] c.LockAndUnpark1 - 再次调用park前
     15:15:03.403 DEBUG [t1] c.LockAndUnpark1 - 再次调用park后
     15:15:03.403 DEBUG [t1] c.LockAndUnpark1 - 三次调用park前
     15:15:03.403 DEBUG [t1] c.LockAndUnpark1 - 三次调用park后
     //这里在主线程里面调用了thread.interrupt()，但是没有在thread线程内部将中断标记恢复，因此第二次的park就不起作用了
     ```
   
     ```java
     package com.thread.chapter4;
     
     import lombok.extern.slf4j.Slf4j;
     
     import java.util.concurrent.TimeUnit;
     import java.util.concurrent.locks.LockSupport;
     
     @Slf4j(topic = "c.LockAndUnpark1")
     public class LockAndUnpark1 {
         public static void main(String[] args) throws InterruptedException {
             Thread thread = new Thread(() -> {
                 log.debug("调用park前");
                 LockSupport.park();
                 log.debug("调用park后");
                 Thread.interrupted();
                 log.debug("再次调用park前");
                 LockSupport.park();
                 log.debug("再次调用park后");
     
                 log.debug("三次调用park前");
                 LockSupport.park();
                 log.debug("三次调用park后");
             }, "t1");
     
             thread.start();
             TimeUnit.SECONDS.sleep(2);
             thread.interrupt();
         }
     }
     =====>输出:
     15:17:13.631 DEBUG [t1] c.LockAndUnpark1 - 调用park前
     15:17:15.631 DEBUG [t1] c.LockAndUnpark1 - 调用park后
     15:17:15.631 DEBUG [t1] c.LockAndUnpark1 - 再次调用park前
     ```
   
     
   
   - 在一个线程内部，先用了LockSupport.park()方法，如果再线程外部对其调用了lockSupport.unpark(thread)，那么还可以继续使用park()方法
   
     ```java
     package com.thread.chapter4;
     
     import lombok.extern.slf4j.Slf4j;
     
     import java.util.concurrent.TimeUnit;
     import java.util.concurrent.locks.LockSupport;
     
     @Slf4j(topic = "c.LockAndUnpark")
     public class LockAndUnpark {
         public static void main(String[] args) throws InterruptedException {
             Thread thread = new Thread(() -> {
                 log.debug("调用park前");
                 LockSupport.park();
                 log.debug("调用park后");
     
                 log.debug("再次调用park前");
                 LockSupport.park();
                 log.debug("在调用park后");
             }, "t1");
     
             thread.start();
             TimeUnit.SECONDS.sleep(2);
             LockSupport.unpark(thread);
         }
     }
     =====>输出：
     15:10:21.902 DEBUG [t1] c.LockAndUnpark - 调用park前
     15:10:23.902 DEBUG [t1] c.LockAndUnpark - 调用park后
     15:10:23.902 DEBUG [t1] c.LockAndUnpark - 再次调用park前
     阻塞在这里
     ```
   
     
   
2. 原理

   每个**线程**都有自己的一个Parker对象，由三部分组成_counter, _cond和__mutex；

   ![image-20200817091224032](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200817091224032.png)

   先调用park，再调用unpark:

   ![image-20200817091334099](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200817091334099.png)

   

   ![image-20200817091434092](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200817091434092.png)

   先调用unpark，再调用park

   ![image-20200817091525648](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200817091525648.png)

   

## 线程状态切换

![image-20200817091606402](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200817091606402.png)



#### 情况1：NEW ---> RUNNABLE

- 当调用t.start()方法时，由NEW--> RUNNABLE

#### 情况2：RUNNABLE <---> WAITING

​	t线程用synchronized(obj)获取了对象锁后

- 调用obj.wait()方法后，t线程从RUNNABLE ---> WAITING

- 调用obj.notify()，obj.notifyAll()，t.interrupt()时

  - 如果竞争锁成功，则t线程从WAITING ---> RUNNABLE
  - 如果竞争锁失败（即在synchronized(obj)时没有获得锁），线程就从 WAITING ---> BLOCKED

#### 情况3：RUNNABLE <---> WAITING

- **当前线程**调用t.join()，**当前线程**从RUNNABLE ---> WAITING
- t线程运行结束，或调用了当前线程的interrpt()方法，当前线程从WAITING ---> RUNNABLE

#### 情况4： RUNNABLE <---> WAITING

- 当前线程调用LockSupport.park()方法会让当前线程从RUNNABLE ---> WAITING;
- 调用LockSupport.unpark(目标线程)或者调用了目标现成的interrupt()，会让目标线程从WAITING ---> RUNNABLE

#### 情况5：RUNNABLE <---> TIMED_WAITING

**t线程**用synchronized(obj)获取了对象锁后

- 调用obj.wait(long n)方法后，**t线程**从RUNNALE --> TIMED_WAITING
- **t线程**等待超过了n，或者调用了obj.notify()，obj.notifyAll()，obj.interrupt()时
  - 如果竞争锁成功，**t线程**从TIMED_WAITING --> RUNNABLE
  - 如果竞争锁失败，**t线程**从TIMED_WAITING --> BLOCKED

#### 情况6：RUNNABLE <---> TIMED_WAITING

- **当前线程**调用t.join(long n)方法时，**当前线程**从RUNNABLE---> TIMED_WAITING
  - 注意是**当前线程**在t线程对象的监视器上等待
- **当前线程**等待超过了n，或**t线程**运行结束，或者调用了**当前线程**的interrupt()，**当前线程**从TIMED_WAITING ---> RUNNABLE

#### 情况7： RUNNABLE <---> TIMED_WAITING

- 当前线程调用Thread.sleep(long n)，当前线程从RUNNABLE --> TIMED_WAITING
- 当前线程等待时间超过了n，当前线程从TIMED_WAITING ---> RUNNABLE



![image-20200817155040485](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200817155040485.png)



![image-20200817155130728](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200817155130728.png)

## 几个带参数方法的说明

1. sleep(long n)

   代表当前线程被暂停n秒，期间不会释放锁，n秒后该线程会继续向下执行；

   可被interrupt()打断，打断后进入异常块中；

   ```java
   Thread t = new Thread(() -> {
       TimeUnit.SECONDS.sleep(1)
   }, "t1");
   t.start();
   t.interrupt();
   ```

   

2. wait() / wait(long n)

   wait()代表在Monitor对象的waitSet中等待，直到被Monitor的notify()打断后进入EntryList，等待竞争锁再次执行wait()后续代码;

   wait(long n)代表等待n秒后，可不被notify()唤醒直接进入到EntryList中等待下一轮竞争锁；

   可被interrupt()打断，打断后直接进入到EntryList中，进而进入异常块，正常代码后续将不被执行

   ```java
   Thread t = new Thread(() -> {
       synchronized(obj) {
           obj.wait();
       }
   }, "t1");
   t.start();
   t.interrupt();
   ```

   

3. join() / join(long n)

   join()在A线程中调用B线程对象的join()方法，A线程会一直等到B线程执行结束才会执行，此时相当于两个线程是串行执行；

   join(long n)表示在A线程中调用B线程的joint(long n)方法，表示A线程在等待B线程n秒后，由串行执行变为A，B并行执行；

   可被interrupt()打断，打断后进入异常块中

   ```java
   Thread t = new Thread(() -> {
       TimeUnit.SECONDS.sleep(1)
   }, "t1");
   t.start();
   
   Thread t1 = new Thread(() -> {
       t1.join();
   }, "t2");
   
   t1.start();
   t1.interrupt();
   ```

   

4. ReentrantLock.tryLock() / tryLock(long n)

   tryLock()如果获得不了锁，就会立刻结束等待锁；

   tryLock(long n)会在n的时间内等待锁，如果n的时间内没有获得锁，结束等待；

   该方法可被interrupt()打断；

   

## 多把锁

### 细粒度锁

细粒度锁的好处就是：提高并发度；但是与之俱来的缺点就是会导致死锁。



### 活跃性

由于一些因素的干扰，线程的代码无法执行完毕，活跃性特征包含死锁，活锁，饥饿。

#### 死锁

t1线程获得了A对象的锁，接下来想获取B对象的锁；t2线程获得了B对象的锁，接下来想获取A对象的锁，就会导致死锁。

```java
package com.thread.chapter4;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;

@Slf4j(topic = "c.DeadLock")
public class DeadLock {
    public static void main(String[] args) {
        Object A = new Object();
        Object B = new Object();
        Thread t1 = new Thread(() -> {
            synchronized (A) {
                log.debug("lock A");
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (B) {
                    log.debug("lock B");
                }
            }
        }, "t1");

        Thread t2 = new Thread(() -> {
            synchronized (B) {
                log.debug("lock B");
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                synchronized (A) {
                    log.debug("lock A");
                }
            }
        }, "t2");

        t1.start();
        t2.start();
    }
}

```



#### 定位死锁

- 检测死锁可以使用jconsole工具，或者使用jps定位进程id，再用jstack定位死锁：

- jps使用方式：

  - jps  //查看到进程ID

    ![image-20200817205210413](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200817205210413.png)

  - jstack 11660

    ![image-20200817205527287](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200817205527287.png)

- jconsole工具

  ![image-20200817205623908](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200817205623908.png)

![image-20200817205656360](G:\笔记\多线程B站\Chapter4 共享模式之管程.assets\image-20200817205656360.png)



#### 活锁

活锁出现在两个线程互相改变对方的结束条件，最后谁也无法结束。

解决思路：可以增加随机的睡眠时间，避免活锁的产生

#### 饥饿

有些线程始终因得不到资源而无法被CPU成功调度执行。



## ReentrantLock(JUC包下的类)

相对于synchronized，它具备如下特点：

- 可中断
- 可以设置超时时间
- 可以设置为公平锁
- 支持多个条件变量(可拥有不同的EntryList，以便依据不同的条件变量将线程存放在不同的队列中)
- 和synchronized一样，都支持可重入

### 基本语法

```java
//获取锁
ReentrantLock reentrantLock = new ReentrantLock();
reentrantLock.lock(); //lock()方法放在try{}里面和外面是一样的
try {
    //临界区
} finally {
    //释放锁
    reentrantLock.unlock();
}
```



#### 可重入锁验证

```java
package com.thread.chapter4;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.locks.ReentrantLock;

@Slf4j(topic = "c.ReentrantLockTest1")
public class ReentrantLockTest1 {
    private static ReentrantLock lock = new ReentrantLock();
    public static void main(String[] args) {
        lock.lock();
        try {
            log.debug("进入了主方法");
            method1();
        } finally {
            lock.unlock();
        }
    }

    public static void method1() {
        lock.lock();
        try {
            log.debug("进入了method1方法");
            method2();
        } finally {
            lock.unlock();
        }
    }

    public static void method2() {
        lock.lock();
        try {
            log.debug("进入了method2方法");
        } finally {
            lock.unlock();
        }
    }
}

```



#### 可打断验证---lockInterruptibly(),一直等待锁,是一个阻塞方法

```java
package com.thread.chapter4;

import lombok.extern.slf4j.Slf4j;

import java.sql.Time;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

@Slf4j(topic = "c.ReentrantLockTest2")
public class ReentrantLockTest2 {
    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            log.debug("尝试获得锁");
            lock.lock();
            try {
                log.debug("获取到了锁");
            } finally {
                log.debug("释放了锁");
                lock.unlock();
            }
        }, "t1");

        lock.lock();
        t1.start();
        TimeUnit.SECONDS.sleep(2);
        lock.unlock();
    }
}
=====>输出:
15:35:22.698 DEBUG [t1] c.ReentrantLockTest2 - 尝试获得锁
15:35:24.698 DEBUG [t1] c.ReentrantLockTest2 - 获取到了锁
15:35:24.698 DEBUG [t1] c.ReentrantLockTest2 - 释放了锁
t1线程会因lock()不到锁而一直等待有人释放
```



lock()方法是不可被打断的，需要用lockInterruptibly()

打断的意思是：当前线程没有获取lock的锁，也就是调用lock.lockInterruptibly()由于没有获得锁而进入了阻塞队列中，在其他线程中可以调用该线程对象的interrupt()方法将该线程唤醒，从而抛出异常，而不再在阻塞队列中继续等下去。

```java
package com.thread.chapter4;

import lombok.extern.slf4j.Slf4j;

import java.sql.Time;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

@Slf4j(topic = "c.ReentrantLockTest2")
public class ReentrantLockTest2 {
    private static ReentrantLock lock = new ReentrantLock();
    public static void main(String[] args) throws InterruptedException {
        Thread t1 =  new Thread(() -> {
            try {
                //如果没有竞争,那么此方法就会获取到lock对象的锁
                //但是如果有竞争,就会进入阻塞队列等待
                //但是被其他线程可以用interrupt()打断等待
                log.debug("尝试获得锁");
                lock.lockInterruptibly();
            } catch (InterruptedException e) {
                log.error("没有获得到锁");
                //被打断表示没有获得到锁
                return;
            }
            try {
                log.debug("获取到了锁");
            } finally {
                lock.unlock();
            }
        }, "t1");

        lock.lock();
        t1.start();
        TimeUnit.SECONDS.sleep(1);
        log.debug("打断T1线程");
        t1.interrupt(); //打断t1线程
    }
}

```



#### 锁超时---tryLock(long n, TimeUnit time)/tryLock()，等不到就停止等待，并返回true/false代表是否获得了锁

tryLock()不带参数的方法表示的是获取不到锁，立刻结束等待；

tryLock(long n, TimeUnit time)带参数的表示在指定时间内获得不到锁，结束等待；该方法是可被打断的，同样会抛出异常，表示没有成功获取到锁

```java
package com.thread.chapter4;

import jdk.nashorn.internal.ir.Block;
import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.ReentrantLock;

@Slf4j(topic = "c.ReentrantLockTest2")
public class ReentrantLockTest3 {
    private static ReentrantLock lock = new ReentrantLock();

    public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            //尝试获得锁
            log.debug("尝试获得锁");
            try {
                if (!lock.tryLock(2, TimeUnit.SECONDS)) {
                    log.debug("获取不到锁");
                    return;
                }
            } catch (InterruptedException e) {
                log.debug("因被唤醒,同样没有换的锁");
                e.printStackTrace();
                return;
            }
            try {
                log.debug("获得了锁");
            } finally {
                lock.unlock();
                log.debug("释放了锁");
            }
        }, "t1");

        log.debug("主线程获得到了锁");
        lock.lock();
        t1.start();
        TimeUnit.SECONDS.sleep(1);
        lock.unlock();
    }
}

```



#### 公平锁验证

ReentrantLock默认是不公平锁

构造函数可以填入一个布尔值，false是非公平锁，true是公平锁。

但是公平锁会降低并发度。



#### 条件变量(Condition类，是从ReentrantLock上取到的)

synchronized中也有条件变量，就是所讲的waitSet，当条件不满足时线程需要进入到waitSet中；

ReentrantLock的条件变量比synchronized强大之处在于，它支持多个条件变量，也就是支持多种不同条件变量的waitSet。

使用流程：

- 调用await()前需要获取锁lock.lock() -->类比与synchronized(obj)
- await()执行后，会释放锁，进入Condition中等待；
- await()的线程可以被唤醒(condition.signal()/signalAll())，被打断或超时会重新竞争锁；
- 竞争lock锁成功后，从await()后继续执行

```java
package com.thread.chapter4;

import lombok.extern.slf4j.Slf4j;

import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.ReentrantLock;

@Slf4j(topic = "c.ReentrantLockTest4")
public class ReentrantLockTest4 {
    private static boolean hasSigarette = false;
    private static boolean hasTakeout = false;
    private static ReentrantLock lock = new ReentrantLock();
    //等烟的休息室
    private static Condition waitSigaretteSet = lock.newCondition();
    //等外卖的休息室
    private static Condition waitTakeoutSet = lock.newCondition();

    public static void main(String[] args) throws InterruptedException {
        new Thread(() -> {
            lock.lock();
            try {
                log.debug("有烟吗？[{}]", hasSigarette);
                while (!hasSigarette) {
                    log.debug("没有");
                    waitSigaretteSet.await();
                }
                log.debug("烟来了可以干活了");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }, "t1").start();

        new Thread(() -> {
            lock.lock();
            try {
                log.debug("有外卖吗？[{}]", hasTakeout);
                while (!hasTakeout) {
                    log.debug("没有");
                    waitTakeoutSet.await();
                }
                log.debug("外卖来了可以干活了");
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();
            }
        }, "t2").start();

        TimeUnit.SECONDS.sleep(2);
        new Thread(() -> {
            lock.lock();
            try {
                //送来了外卖
                hasTakeout = true;
                waitTakeoutSet.signalAll();
            } finally {
                lock.unlock();
            }
        }, "t3").start();

        TimeUnit.SECONDS.sleep(3);
        new Thread(() -> {
            lock.lock();
            try {
                //送来了烟
                hasSigarette = true;
                waitSigaretteSet.signalAll();
            } finally {
                lock.unlock();
            }
        }, "t4").start();

    }
}

```
















