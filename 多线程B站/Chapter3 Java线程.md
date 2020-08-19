[toc]

# Chapter3 Java线程

## 创建和运行线程

1. 方法一:直接使用Thread
```
//创建线程对象
Thread t = new Thread() {
    public void run() {
        //doSomething
    }
}
//启动线程
t.start();
```
2.方法二:使用Runnable配合Thread
```
Runnable run = new Runnable() {
    public void run() {
        //doSomething
    }
}

Thread t = new Thread(run);
t.start();
```
推荐使用第二种方式创建线程;
使用λ表达式:
```
Runnable runnable1 = () -> log.debug(""); 
```
3. Thread和Runnable的联系
- Thread创建线程就是把线程和任务合并在一起,Runnable把线程和任务分开了;
- 用Runnbale更容易和线程池高级API使用;
- 用Runnable让任务脱离了Thread的继承体系,反而是一种组合手法,更加灵活(任务组合线程)
4. 方法三: 使用FutureTask配合Thread
FutureTask能够接受**Callable**类型的参数,并且能够处理返回的结果(调用get()方法);
```
public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<Integer> task = new FutureTask(() -> {
            log.info("running");
            Thread.sleep(3000);
            return 1233;
        });
        Thread t = new Thread(task, "t1");
        t.start();
        log.info("{}", task.get());
    }
=====>
输出:
14:12:04 [t1] c.FutureTaskTest - running
14:12:07 [main] c.FutureTaskTest - 1233
```
PS:FutureTask的get()方法是个阻塞方法。

##  查看进程线程的方法

1. windows
- 可以通过任务管理器查看进程和线程数
- tasklist 查看进程
- taskkill /f /pid 12345 杀死进程

2. Linux
- ps -ef 查看所有进程信息
- kill -9 12345 杀死进程
- top 以动态方式查看进程信息<br>top -H -p 1234 (-H代表查看线程,-p代表PID)

3. Java命令
- jps 命令查看所有Java进程
- jstack 1234 查看详细的线程信息
- jconsole(图形化界面)<br>
远程连接时,需要在启动程序时写一段配置:
```java
java -Djava.rmi.server.hostname=192.168.0.0 -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.port=1234(自己随意写一个)
-Dcom.sun.management.jmxremote.ssl=false
-Dcom.sun.management.jmxremote.authenticate=false Test2(Java程序名)
```



## 线程运行的原理

1. 栈和栈帧
JVM由堆,栈,方法区组成,栈的内存空间是给线程使用的,每个线程独享自己的栈。
每个栈由多个栈帧(Frame)组成,对应着每次方法调用时所占用的内存;
每个线程只能由一个活动栈帧,对应着当前正在执行的方法。
方法执行完,栈帧就被释放了。
2. 每个**线程**都有自己的程序计数器,用来读取当前该执行的代码行;每个被调用的方法都有自己栈帧,栈帧包含局部变量表,返回地址,锁等；
3. 每个线程有自己的程序计数器和栈帧,是互不干扰的.
4. 线程的上下文切换
- 因为以下的原因导致CPU不再执行当前线程,转而需要执行另外的线程:
    1. 线程的CPU时间片用完
    2. 垃圾回收
    3. 有更高优先级的线程需要运行
    4. 线程自己调用了sleep,yield,wait,join,park,synchronized,lock等方法
- 当上下文切换时,操作系统会保存当前的线程状态，并恢复另一个线程的状态,Java种对应的概念就是程序计数器,**它的作用是记住下一条JVM指令,是线程私有的**。
- 操作系统需要记录的状态包括:程序计数器,虚拟机栈中每个栈帧的信息,如局部变量,操作数栈,返回地址等；
- 上下文切换频繁会影响性能。



## 线程中的常用方法

| 方法名          | static | 功能说明                                           | 注意                                                         |
| --------------- | ------ | -------------------------------------------------- | ------------------------------------------------------------ |
| start()         |        | 启动一个新线程,在新的线程<br>运行run()方法中的代码 | start()方法只是让线程进入就绪,里面的代码不一定立刻运行,且每个线程对象的start()只能调用一次,多次调用同一个线程身上的start方法会出现IlleagleThreadStateException |
| run()           |        | 新线程启动后执行的方法                             | 如果在构造Thread对象时传递了Runnable参数,则线程启动后会调用Runnale中的run方法,否则默认不执行任何操作;但是可以创建子类对象,从而覆盖默认的run()方法 |
| join()          |        | 等到线程运行结束                                   | 一直等待                                                     |
| join(long n)    |        |                                                    | 有时间限制                                                   |
| setName(String) |        | 设置线程名称                                       |                                                              |
| getState()      |        | 获取线程状态                                       | Java中线程状态是6个enum表示,分别是:NEW,RUNNABLE,BLOCKED,WAITINT,TIMED_WAITING,TERMINATED |
| isInterrupted() |        | 判断线程是否被打断                                 | 不会清除打断标记                                             |
| isAlive()       |        | 判断线程是否还存活                                 |                                                              |
| inInterrupted() | static | 判断当前线程是否被打断                             | 会清除打断标记                                               |
| currentThread() | static | 获取当前正在执行的线程                             |                                                              |
| sleep(long n)   | static | 让当前线程休眠n秒                                  |                                                              |
| yield()         | static | 提示线程调度器让出当前线程对CPU的使用              | 主要用于调试和测试                                           |

- start与run
  
    1. start()是另外启动一个线程,从而去调用该线程对象覆盖的run();如果线程对象直接调用run(),其实是相当于主线程直接调用run()执行代码,根本无法起到异步多线程的作用;
- sleep与yield
    1. sleep()
    - 调用sleep会让当前线程从Running进入到Timed Waiting状态;
    - **其他线程可以使用原线程对象的interrupt方法**打断正在睡眠的线程,这时sleep会抛出InterruptedException;
    - 睡眠结束后的线程处于就绪状态;
    - 建议使用TimeUnit的sleep代替Thread的sleep方法来获得更好的可读性;
    ```
    public static void main(String[] args) throws InterruptedException {
        TimeUnit.SECONDS.sleep(1);
    }
    ```
    2. yield
    - 调用yield会让当前线程从Running进入Runnable状态,然后由任务调度器执行其他同优先级的线程,如果这时没有同优先级的线程,不能保证当前线程处于Runnable状态;
    - yield可以说是是仅仅给线程调度器一个提示,到底能不能转化状态,还需要看调度器的具体实现.
    3. 案例---防止CPU占用100%
    - 使用sleep(long n)间隔,防止对CPU满占用;
    - 可以使用wait和条件变量,这两种方法需要加锁
    - sleep适用于无需同步的场景
    - wait和条件变量适用于需要加锁的场景
- join(可以实现同步等待)
    - 调用哪个线程对象join,就要等待哪个线程运行结束
    ```
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(() -> {
                try {
                    Thread.sleep(500);
                    log.debug("t1");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
        });
        t.start();
        t.join();
        log.debug("main.....");
    }
    ```
    由于调用了t1的join方法,那么主线程就会等待t1运行结束(在哪里调用了谁的join,那么哪里就会被阻塞,等待谁的join,如上代码,在main方法中调用了t1的join,那么main就要等待t1的运行结束才会继续运行自己).
    - 等待多个线程的结束
    ```
    public static void main(String[] args) throws InterruptedException {
        Thread t = new Thread(() -> {
                try {
                    Thread.sleep(500);
                    r1 = 10;
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
        });
        Thread t1 = new Thread(() -> {
            try {
                Thread.sleep(700);
                r2 = 20;
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });
        t.start();
        t1.start();
        log.debug("join begin");
        t.join();
        log.debug("t join end");
        t1.join();
        log.debug("t1 join end");
        log.debug("main.....");
        log.debug("r1 = {}, r2 = {}", r1, r2);
    }
    ```
    分析上述代码,由于t1需要睡眠500ms,t2睡眠700ms,因此等待t1后,只需要再等待t2 200ms,main线程就可以运行了;
    如果把t2和t1的join方法互换位置,t2睡眠700ms,t1睡眠500ms,当main等到700m后,t1其实早已结束了睡眠,故t1的join方法早已失效了,main线程根本不需要再等待t1了;
    因此,不论t1,t2的join方法放在哪里,main线程等待的是那个时间长的.
    - join(long n)
    在main线程中执行t.join(n),如果n<线程t的执行时间,那么main将不再等待线程t执行完毕,而是直接往下继续执行;如果n>线程t的执行时间,那么main在t线程执行完毕后就直接向后执行,也不需要等待n的时间。

- interrupt(该方法会有一个打断标记,打断谁就调用谁身上的该方法)
    1. 打断阻塞线程,如:sleep,wait,join的线程;<br>
    **处于sleep,wait,join的线程被打断后,其isInterrupted返回false,并且也只有这三个方法被打断会抛出异常.**
    ```
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() ->
        {
            try {
                log.debug("enter sleeping...");
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                log.debug("wake up...");
                e.printStackTrace();
            }
        }, "t1");
        thread.start();
        Thread.sleep(1500);
        thread.interrupt();
        log.debug("is interrupe: {}", thread.isInterrupted());
    }
    =====>
    输出:
    09:16:10.761 [t1] c.sleepTest - enter sleeping...
    09:16:12.260 [t1] c.sleepTest - wake up...
    java.lang.InterruptedException: sleep interrupted
	at java.lang.Thread.sleep(Native Method)
	at com.thread.chapter3.SleepTest.lambda$main$0(SleepTest.java:12)
	at java.lang.Thread.run(Thread.java:748)
    09:16:12.260 [main] c.sleepTest - is interrupe: false
    在main线程中调用thread对象身上的interrupt(),用来表示main线程打断了处于阻塞的线程.
    ```
    2. 打断正常运行的线程
    打断正常运行的线程被打断后isInterrupted返回true
    ```
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            while (true) {
                //在main线程中调用thread.interrupt(),子线程仍旧处于运行,是否真的打断,需要自身逻辑实现
                if (Thread.currentThread().isInterrupted()) {
                    log.debug("被打断了,推出循环");
                    break;
                }
            }
        }, "t1");
        thread.start();
        Thread.sleep(1000);
        thread.interrupt();
        log.debug("interrupt: {}", thread.isInterrupted());
    }
    =====>
    09:22:28.709 [main] c.InterruptTest - interrupt: true
    09:22:28.709 [t1] c.InterruptTest - 被打断了,推出循环
    ```
    3. 注意interrupted和isInterrupted和interrupt
    - interrupt是用来打断线程的,但是仅仅是提示该线程要被打断,如果该线程的内部没有实现任何逻辑,那么这个打断并不会影响该线程停下来;
    - interrupted用来判断线程是否被打断,而且清除打断标记,第一次返回true,返回后就会将打断标志置位false,第二次就返回false;
    - isInterrupted是用来判断线程是否被打断,而且不会清除打断标记,多次调用返回同样的结果; 
    4. interrupt打断park线程(要想park能够阻塞线程,那么该线程的打断标记必须为false)
    LockSupport.park()也是使线程处于阻塞状态,但是打断park与打断join,yield,sleep有着很大的不同;首先,打断park并不会抛出异常,而是打断park后会继续向下执行代码;其次,用interrupt打断park,调用线程的isInterrupted方法返回的是true(就像打断了正常的线程一样);第三:如果打断了park()方法,那么此时打断标记是true,如果在后面继续调用park方法,线程是无法被阻塞的,需要将打断标记设为false才可以再次发挥作用
    ```
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            log.debug("调用park前...");
            LockSupport.park();
            log.debug("调用park后...");

            log.debug("再次调用park前...");
            Thread.interrupted(); //这里调用的是可以重置打断标志的方法
            LockSupport.park();
            log.debug("再次调用park后...");

        }, "t1");

        thread.start();
        TimeUnit.SECONDS.sleep(1);
        thread.interrupt();
    }
    =====>
    输出:
    16:12:05.087 [t1] c.ParkTest - 调用park前...
    16:12:06.087 [t1] c.ParkTest - 调用park后...
    16:12:06.087 [t1] c.ParkTest - 再次调用park前...
    16:12:06.087 [t1] c.ParkTest - 再次调用park后...
    ```
    观察上述代码,我们可以看到第一park前后相差了1s,是由于主线程在1s后打断其park的,但是第二次打断前后的日志是一起输出的,表明park根本没有阻塞住;<br>额外加一行Thread.interrupted();重置打断标记的方法,此时打断标记被置为false;因此park又可以继续发挥作用了;
    ```java
    输出:
    16:15:28.471 [t1] c.ParkTest - 调用park前...
    16:15:29.471 [t1] c.ParkTest - 调用park后...
    16:15:29.471 [t1] c.ParkTest - 再次调用park前...
    ```



## 主线程和守护线程

1. 默认情况下,JAVA进程需要等待其内部所有的线程等待结束后才结束;有一种特殊的线程叫做守护线程,只要其他线程结束,守护线程也就可以结束了;
2. 设置方法就是调用线程对象身上的setDaemon(true);
3. 应用场景:
    - 垃圾回收器就是一种守护线程;
    - Tomcat中的Accepter和Poller都是守护线程;



## 线程的状态

1. 从**操作系统**层面来描述:<br>
初始状态:仅仅创建了对象;<br>
可运行状态(就绪状态):可以被CPU调度,只要获得CPU的时间片便可运行;<br>
运行状态:获得了CPU的时间片,正在执行代码;当时间片用完会转变为就绪状态,而不是阻塞;<br>
阻塞状态:调用了某些阻塞方法,状态会转变为就绪状态;<br>
终止状态:当线程将程序代码执行完毕.<br>
2. 从**Java Thread内部枚举**层面描述:<br>
来自Thread.state()<br>
NEW:线程刚被创建,还没有调用start()方法;<br>
RUNNABLE:当调用了start()方法之后,这里的RUNNABLE涵盖了操作系统层面中的[可运行状态],[运行状态]和[阻塞状态];(这里的阻塞时操作系统层面的阻塞,和下面三种阻塞不一样)<br>
BLOCKED:synchronized<br>
WAITING:join()操作,wait()操作<br>
TIMED_WAITING:sleep()操作<br>
TERMANATED:线程执行完毕后
```java
public static void main(String[] args) throws InterruptedException {
        Thread t1 = new Thread(() -> {
            log.debug("t1~~~");
        }, "t1");

        Thread t2 = new Thread(() -> {
            while (true) {
            }
        }, "t2");
        t2.start();

        Thread t3 = new Thread(() -> {
            log.debug("t3");
        }, "t3");
        t3.start();

        Thread t4 = new Thread(() -> {
            synchronized (DifferentState.class) {
                try {
                    TimeUnit.SECONDS.sleep(60);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "t4");
        t4.start();

        Thread t5 = new Thread(() -> {
            try {
                t2.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "t5");
        t5.start();

        Thread t6 = new Thread(() -> {
            synchronized (DifferentState.class) {
                try {
                    TimeUnit.SECONDS.sleep(60);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "t6");
        t6.start();

        Thread.sleep(100);
        System.out.println("t1 " + t1.getState());
        System.out.println("t2 " + t2.getState());
        System.out.println("t3 " + t3.getState());
        System.out.println("t4 " + t4.getState());
        System.out.println("t5 " + t5.getState());
        System.out.println("t6 " + t6.getState());
    }
=====>
输出:
t1 NEW
t2 RUNNABLE
t3 TERMINATED
t4 TIMED_WAITING
t5 WAITING
t6 BLOCKED
```

sleep 不释放锁、释放cpu
join 释放锁、抢占cpu
yiled 不释放锁、释放cpu
wait 释放锁、释放cpu