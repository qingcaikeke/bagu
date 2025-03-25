### 参考资料

[CS-Notes/notes/Java 并发.md at master · CyC2018/CS-Notes](https://github.com/CyC2018/CS-Notes/blob/master/notes/Java 并发.md#await-signal-signalall)

**什么是公平锁和非公平锁？**

公平锁按申请锁顺序获取锁，先到先得，能避免线程饥饿，但是需要维护线程队列，效率会更低一些

非公平锁按随机顺序获得锁，都可以竞争，可能导致线程饿死

**创建3个线程打印按序abc**

实现runnable接口，任务接收的参数有字符和id，分别创建3个线程，加锁，给个count计数，只有count取余3等于id的时候才能打印

### 并发编程

**java的线程和操作系统的线程有什么关系**

java线程是由Java虚拟机（JVM）管理（创建、销毁）的，会映射到操作系统线程，会利用操作系统的线程调度和同步机制。

##### **线程创建的方法**

1.继承thread类，重写run方法 2.实现runnable接口，重写run方法，作为参数传入new thread（）构造函数 3.实现callable接口 ，作为参数传到FutureTask，然后用FutureTask创建线程 

3.线程池ExecutorService 4.匿名内部类（和runnable差不多）

本质上其实只有一种方法，new thread() .start()

补：采用实现更好：Java单继承，实现可以继承别的类；继承整个Thread开销过大，一般要求可执行就行

补：不能直接 thread.run()，会被当成main线程下的一个普通方法执行，而非多线程，而start()会使线程进入就绪态，等分到时间片后，自动执行run方法下的内容。

补：线程只是实现Runnable或实现Callable接口，还可以继承其他类。这种方式下，多个线程可以共享一个target对象，非常适合多线程处理同一份资源的情形。

```java
public class MyThread extends Thread {
	@override
	public void run() {}
	public static void main(String[] args) {
        new myThread().start();
    }
}
public class MyRunnable implements runnable {
	@override
	public void run() {}
}
	public static void main(String[] args) {
        MyRunnable runnable = new MyRunnable();
        new Thread(runnable).start(); // 多线程可以绑定同一个runnable对象
        或 new Thread(() ->{}).start 
    }
```

补：循环中使用lambda每次都创建一个新对象

```
创建10个runnable
for (int i = 0; i < 10; i++) {
    Thread thread = new Thread(printNumber::run1, "thread:" + i);
    thread.start();
}
共用一个
Runnable runnable = printNumber::run1;
for (int i = 0; i < 10; i++) {
    Thread thread = new Thread(runnable, "thread:" + i);
    thread.start();
}
```

补：终止一个正在运行的线程：Thread.interrupt(）

```java
创建runnable，重写了run方法就认为是一个runnable对象
runnable 是个接口，接口本身没法new，平常利用多态是 new 实现类，赋值给接口类型的引用变量
MyInterface myInterface = new MyClass();
所以下面的写法本质上是创建了一个匿名类
匿名类：MyInterface myInterface = new 接口(){ @override method_A}; 
		myInterface.method_A(param,param2)

Runnable runnable = new Runnable() {
    @Override
    public void run() {
        printNumber.run1();
    }
};
runnable又是个函数式接口，所以可以直接用"方法引用"创建runnable对象
Runnable runnable = () -> printNumber.run1();
Runnable runnable = printNumber::run1;
```



##### callable和future

runnable接口的run无返回值，callable接口有个泛型T，call方法返回T类型的值

`FutureTask`相当于对`Callable` 进行了封装，管理着任务执行的情况，存储了 `Callable` 的 `call` 方法的任务执行结果

能取消任务、查看任务是否执行完成、获取任务执行结果

```java
Callable<Integer> callable = new MyCallable();
FutureTask<Integer> task = new FutureTask<>(callable);
new Thread(task).start();
int result = task.get();
```

```java
callable是个函数式接口，所以可以用lambda，——>后面代表call方法的具体实现
FutureTask<Integer> task1 = new FutureTask<>(() -> {
    Thread.sleep(300);
    System.out.println("Ff");
    return Integer.MIN_VALUE;
});
```

##### 线程和对象的关系

任何对象都可以作为锁协调线程间通信

对象可以作为线程间传递信息的载体（消息队列，充当一个共享缓冲区的作用）

线程关联对象以执行对象的方法

```
class MessageQueue {
    private Queue<String> messages = new LinkedList<>();
    public synchronized void putMessage(String message) {
        messages.add(message);
        notify();
    }
    public synchronized String getMessage() {
        while (messages.isEmpty()) {
            try {wait();} 
            catch (InterruptedException e) {e.printStackTrace();}
        }
        return messages.poll();
    }
}
```

```
public class Main {
    public static void main(String[] args) {
        MessageQueue ms = new MessageQueue();
        Thread thread1 = new MyThread(ms);
        Thread thread2 = new MyThread2(ms);
        thread1.start();thread2.start();
    }
}
```

```
class MyThread extends Thread {
    private MessageQueue ms;
    @Override
    public void run() {
        try {
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        ms.updateLabelText("更新后的文本");
    }
}
```

##### 线程通信方式

共享内存：生产者消费者

消息传递：wait，notify（拿到锁才能调这两个方法，线程也可以看作一个对象？线程有线程id，有状态，锁对象有个队列，一个存等着抢锁的，一个存wait等着notify的；wait后，改状态不让抢cpu，赛队列里；调notify后，就拿出放到抢锁队列，能抢锁）选择合适状态的线程给cpu执行靠jvm

**线程状态**

初始：创建没start；就绪：start了，可以抢cpu了；运行：抢到cpu尝试执行；

等待：主动挂起，不会抢cpu，被notify后才会；object.wait() 、thread.join()；等notify；thread.sleep(long)，等定时结束

阻塞：抢到cpu发现没锁，被动等待，到时间了释放cpu，等下一次抢锁

##### **sleep() 和wait()**

核心：是否释放锁，用途，是否自动苏醒

sleep() 是thread的方法，不会释放锁；wait主要用于线程间通信，sleep用于定时任务或简单等待。

而 wait()是object的 方法，**在同步代码块中调用**，会释放了锁 。wait之后不会自动苏醒，需要notify，notify之后重新进入synchornize代码块，需要重新获得锁。

join():写在t2的runnable中，t1.join(5000)，表示t2等待t1 5s，或不指定时间，t2等待t1执行完成

补：只有获得了该对象的锁，才能调用该对象的wait方法，否则会抛异常

补：为什么wait不是thread的方法？Java中的锁是对象级的，wait如果放到thread类里，等待哪个锁就不明显了

##### thread.join()

在一个线程中调用另一个线程的`join`方法时，当前线程会被阻塞(是挂起而非忙等待)，直到被调用`join`方法的线程执行完毕。

可以用于同步执行顺序，如一个线程读数据，一个线程处理数据，要等待读完之后处理线程才能开始工作

join底层是wait，notify

##### io密集型和cpu密集型

io设备把数据传到内存（也用到cpu，告诉dma拷什么，拷到哪，拷完通知cpu）

cpu再把内存数据调进来

cpu密集型任务大量依赖cpu计算，一般线程数为核数+1，减少上下文切换次数

io密集型和以多来点线程，提高cpu利用率

##### 虚拟线程

平台线程（应用线程）和内核线程一对一，平台线程管理虚拟线程

适用于io密集型任务，非常轻量，可以把异步代码编写的更像同步代码

##### volatile关键字

**保证变量可见性**：一个线程修改了共享变量的值，其他变量可以立刻看到这个修改。多用于状态标记量和单例模式修饰instance确定其是否被创建

可见性：线程1对变量修改，线程2能及时知道并作出反应，具体而言，线程1修改cpu内存的值，强制刷新到主存，线程2强制自己的cpu内存失效人，然后获取最新值（禁用cpu缓存）

java中把变量声明为volatile，表示该变量共享且不稳定，每次使用都需要从主存读取，而非直接使用线程本地内存中的共享变量副本；补：每个线程都有自己的缓存空间，对变量的操作都是在缓存中进行的，之后再将修改后的值返回到主存中

**禁止指令重排序**：为了提高程序执行效率，**编译器和处理器**可能会对指令进行重新排序。但是，如果一个变量被volatile修饰，就禁止了指令重排，确保每个线程都能看到正确的操作顺序。**举例**：单例模式中，如果重排序 1.分配内存 2.初始化 3.返回指针指向 如果23重排序就可能导致空指针，线程1执行了13，然后线程2判断instance不为空，直接返回，而这时候实例还没有初始化，使用就会导致控制很

**底层**：内存屏障 + 强制刷新cpu缓存

可见性：改完值，插一个写屏障，使其他cpu核心的缓存失效；读前，插一个读屏障，强制从主存读

重排序：有好几种屏障，类似禁止上面的普通读和下面的volatile重排序；禁止上面的volatile写和下面的volatile读重排序；

**volatile不能保证原子性**，只能通过加锁解决

不能保证原子性，也不会阻止并发访问，但是性能更好，比方说两个线程同时执行++，然后写回到内存，有一个可能会被覆盖，因为i++不是原子操作（包含读，改，写回），虽然线程a读了值，没改，阻塞了，线程b读值，然后a改了，写回，但是这时候b读值的操作已经完成了，并且读的对当时来说是新值，然后直接执行改，写回，这样两个线程就有一次操作被覆盖了

##### **synchornize**

底层：wait队列，阻塞队列，对象头上有：锁状态(偏向时写偏向id)，锁计数器，正常为0

使用：锁实例方法(当前对象)，锁静态方法(Class对象)，锁代码块

是可重入锁（通过计数器实现），是**隐式锁**（自动加锁和释放锁），一个线程进入syn代码块，会尝试获取对象或者类的内置锁，如果没被占用，就成功获取，对象头中有两个相关标记位，偏向锁标记位和锁状态标记位。补：通过对象监视器实现，监视器有锁和计数器

补：怎么实现可重入的：有一个锁计数器，如果发现当前申请加锁的线程和持有锁的是一个线程，就把计数器+1，count减少到0就释放，owner指向null

**和volatile的区别**：volatile更轻量，只能修饰变量，只能保证多线程间数据可见性不保证原子性

syn能修饰方法和代码块，能保证原子性，主要解决多线程访问资源的同步性（顺序）

**和reentrantlock区别：**两者都是可重入锁。但lock多了一些高级功能，如公平锁(先来先获得)，可以实现选择性通知(等待唤醒)

是显示锁，需要手动释放

底层实现：synchronized 是一个关键字，是在JVM层面通过监视器实现的，而 ReentrantLock 是jdk层面的,能看到她的源码，他是基于AQS实现的；

##### synchronized锁优化（锁升级）

（无锁，偏向锁，自旋锁，重锁）

目的：希望花费最小的代价达到目的 ------- 补：对象头包括**锁状态，哈希码，垃圾回收信息**

**偏向锁**：线程进入同步块时，如果发现之前没人加过锁，会将**线程id标记在锁对象**头上，并标记为偏向锁，如果新线程想获取锁，如果发现锁对象有偏向锁标记，就判断持锁对象是否为当前线程，是的话直接获取锁(不用CAS了)，不是则升级为轻量级锁

为什么偏向锁要升级成自旋锁：如果不升级，多线程就变成串行执行了，所以说是为了防止a一直持有锁不放的情况

**自旋锁**（轻量级锁）：尝试获取锁的线程不会立即阻塞，而是采用循环的方式去尝试获取锁，避免线程频繁地进入和退出阻塞状态，以减少线程切换的开销（上下文切换，用户态内核态的切换）。自旋次数有限制，超过一定次数后如果还未获取到锁，线程会放弃自旋，转而进入阻塞状态，避免占用过多的CPU资源。——总结：自旋的去将锁对象头的偏向线程id修改为当前线程id

**重锁**：除了拥有锁的线程全部阻塞

##### **ReentrantLock**

是可重入锁，是显式锁，实现了lock接口，使用之前需要先创建 ReentrantLock 对象，然后使用 lock 方法进行加锁，使用完之后再调用 unlock 方法释放锁，可以创建**公平锁**（队列实现，公平锁多了一个判断是否有线程在排队的判断，否则直接CAS），可以实现轮询

**condition,await,signal**

##### 等于如何实现多线程之间顺序执行？

在t2的run方法里写t1.join，就是等待t1执行完之后t2才继续执行join后面的语句

wait，notify：线程等待某个条件，等待时挂起，其他线程的运行使这个条件满足后，会将其唤醒

await，signal：condition对象的方法，要用lock对象创建condition对象

##### 阻塞队列

内部使用了ReentrantLock和对应的Condition，队满put或队空take的时候会释放锁并等待唤醒，唤醒后抢到cpu执行权，再抢到锁才能继续put或take

LinkedBlockingQueue是个单向链表+两把锁+两个条件。具体一把锁用于入队，一把用于出队，即同一时刻只能有一个线程入队/出队，但可以一个入队线程和一个出队线程共同执行，为了维持线程安全，用一个AtomicInterger表示当前队内元素个数

插入删除等任务包括：判断是否已满 -定位 -插入 - 修改大小等任务；如果不是原子进行会产生线程安全问题，如同一位置的数据被覆盖

```json
 ReentrantLock putLock; Condition notFull;
 final AtomicInteger count =this.count; //多个线程认为当前有9个元素
 putLock.lock() //只有一个线程能获得锁,获得锁的线程加一后，其他线程抢到锁但会进入while，然后await，保证不超容量
 while(count.get() == capacity){
 	  notFull.await();
 }
 enqueue(node);
 c = count.getAndIncrement(); // c返回的是incr之前的值
 if (c + 1 < capacity) //如果其他线程全部await，不加这一行，只有下次消费后调用notFull才能唤醒生产
	notFull.signal();
 putLock.unlock();
 if (c == 0) // 之前是空，加这段保证及时唤醒
    signalNotEmpty();
```

ArrayBlockingQueue是一个对象数组+一把锁+两个条件，一把锁更简单，但并发性能会差点，数组是连续空间，对元素的操作会影响相邻元素，靠移动index确定增删位置，搞两个容易出现交叉；而链表靠指向next，受影响较小

##### threadLocal

线程本地变量，访问该变量的每个线程都会有一个该变量的副本，起到隔离作用，保证线程安全

底层：一个thread一个map，key是ThreadLocal对象，val是要存的对象

栈里的ThreadLocal引用指向堆中的ThreadLocal对象，堆中有一个entry，key指向堆中的ThreadLocal对象，val指向要存的大对象

栈里的线程引用指向堆里的线程，堆里的线程能找到ThreadLocalMap，map由多个entry组成[github-threadLocal](https://github.com/CoderLeixiaoshuai/java-eight-part/blob/master/docs/java/juc/内存泄露的原因找到了，罪魁祸首居然是Java TheadLocal.md)

内存泄漏：key(是弱引用，val不是，这样的设计是因为，threadMap的生命周期是和thread一样的，不用的key不清理就会内存泄漏，使用弱引用使不用的key可以直接被回收，之后再次访问map，会清除key为null对应的val

补：如果key是强引用，线程活着，map就活，threadLocal所在的entry就活，即使不会再用。尤其是使用线程池，线程会复用。

若引用指的是： `ThreadLocalMap` 中的 `Entry` 对 `ThreadLocal` 对象是弱引用

解决方案：每次使用完ThreadLocal，建议调用它的remove()方法，相当于手动清除了当前线程的 `ThreadLocal` 键对应的entry，把key和val都置为null；弱引用相当于一个兜底，gc把key回收，下次访问map时就会清理null key对应的val

补：使用开放寻址法解决哈希冲突，当发生冲突时，它会从当前位置开始，逐个向后查找空的位置，直到找到合适的位置来存放元素。threadLocal对象的hashCode也和正常不同，是每创建一个新对象，按一个固定值递增，即固定增量使数据分布均匀

补：每当线程主动访问tl对象，线程内部的map就会创建响应的键值对

一定要区分好，线程持有的是tl引用，访问tl对象，线程内部map创建键值对，key就是这个引用，val是变量副本，真正的val和tl对象都在堆里，所以hashcode实际上取决于堆里有多少tl对象

线程1和线程2的map，key都是userTl对象，map中，线程1是user1，线程2是user2

每个thread的map都是放在堆里的，5个thread堆里就有5个map

[关于线程池在生产环境中的使用 (qq.com)](https://mp.weixin.qq.com/s?__biz=MzkyMTM4MjI0OQ==&mid=2247484259&idx=1&sn=3c29de8daf9d470db3eec400235d6aea&chksm=c1853f65f6f2b673e5b5cc777019705420d5d93320d5dfcc08f65538d4abfb5116d75da128da&scene=21#wechat_redirect)

##### **线程池作用**

减少创建销毁线程的开销；高并发提供响应速度（不用等待创建）；可以通过线程数量控制并发度；统一管理资源，提高利用率

两个任务共用线程池有什么危害？资源竞争，相互影响，一个任务把线程全拿走了，另一个只能等待；难以监控每个服务的使用情况，并做出调优

**线程池中核心线程数量大小怎么设置**

cpu密集型线程数+1，防止过度切换，io密集型，可以2*cpu个数，多来几个，实际中需要测试找到一个理想的值

**线程池创建的几个参数**，线程池原理：讲一下这个流程就行

1.核心线程数2.最大线程数3.等待时间4.时间单位5.任务队列6.线程工厂7.拒绝策略

线程名称：重写了线程工厂，主要是 `为了线程的命名规范` ，这样在查询日志时，只要做好业务之间的隔离，就可以很容易的根据线程名称来定位到对应的业务，便于分析线上问题 

**拒绝策略：**1抛异常；2不处理新任务，将其丢掉；3丢掉最早未处理的任务；

4任务退回给调用者，使用调用者线程(main线程)执行任务，该方法保证所有任务不会被丢弃，但main用来执行任务，就没法向线程池提交其他任务，导致整体执行速度慢，所以如果可以，应该增加阻塞队列大小

```
ExecutorService es = Executors.newFixedThreadPool() //更建议使用后者
ThreadPoolExecutor pool = new ThreadPoolExecutor(3,6,60,s,任务队列，Executors.defaultThreadFactory())
```

**线程池的类型**

FixedThreadPool：核心线程数等于最大线程数，线程池中有空闲线程就立即执行，没有就放到任务队类，适合任务执行时间较短的情况

CachedThreadPool：线程数可以无限增大，同时闲置时对线程回收（从缓存中移除），即线程池中的线程数不是固定不变，任务队列不储存任务，只负责中转，所以效率比较高，适合任务执行时间较短、任务数不确定。或即时任务，需要尽快完成

SingleThreadExecutor：使用唯一的线程取执行任务，保证任务按提交顺序依次执行

ScheduledThreadPool：可以设置定期的执行任务，比如每隔 10 秒钟执行一次任务

SingleThreadScheduledExecutor：只有一个线程，定期的执行任务

补：利用Executors的静态方法都可能oom，fixed和single都用的无界的阻塞队列，schedule用的无界阻塞队列，cached允许无限创建线程

补：阻塞队列，有一个最大容量，队列为空时会阻塞获取元素的操作，或在队列已满时会阻塞添加元素的操作。底层用了ReentrantLock

补：阻塞是指线程在执行过程中暂时停止，并且不会消耗CPU资源，直到某个条件满足或者等待时间到达。

补：一般不同的业务使用不同的线程池，避免非核心业务对于核心业务的影响，一般非核心业务执行慢，核心业务在任务队列中等待，拿不到线程执行；父子任务同用一个线程池可能死锁，父把核心线程全用完了，子没有线程可用，父也无法提交

##### submit和execute

execute返回的是void，属于Executor接口

submit可以返回持有计算结果的Future对象，属于扩展后的ExecutorService接口

```
public interface ExecutorService extends Executor
```

##### 原子类

利用cas保证原子性，常用atomicInteger，常用方法

补：AtomicIntegerArray，使用原子的方式更新数组里的某个元素

public final int get() //获取当前的值

public final int getAndSet(int newValue)//获取当前的值，并设置新的值

public final int getAndIncrement()//获取当前的值，并自增

public final int getAndDecrement() //获取当前的值，并自减

public final int getAndAdd(int delta) //获取当前的值，并加上预期的值

#### 其他

##### 其他

**线程安全如何保证**：锁（synchornize，lock，jvm层面，jdk层面），原子类(cas+硬件)，生产者消费者，但为了共享数据的并发安全，也要靠锁；信号量，屏障，cas有时候也都行

并发场景如何顺序执行：锁，线程池，信号量，countdownlatch

分布式场景/分布式客户端呢：分布式锁，kafka+单分区

**加锁的线程挂了咋办**：如果synchornize会自动释放，如果lock，没办法，别的线程就是拿不到，所以要在finally写释放代码

##### **java中有哪些类型的锁**

乐观悲观，读写，公平非公平等

synchornize，reentrantlock，读写锁（ReentrantReadWriteLock），cas乐观锁

乐观锁和悲观锁区别：是否认为共享资源会经常承受并发，读多还是写多

##### **CAS**

三个操作数，V：变量内存地址；A：旧的预期值；B：准备设置的新值

核心原理是基于硬件层面的原子性保证，新值写入内存时，比较内置值是否与**预期原值**相同，缺点是会有aba问题和开销问题（会一直轮询），aba问题：当前值和预期值相等不意味着没改过。解决：加个版本号或时间戳

**还有哪些地方用了CAS**？AtomicInteger原子类，锁ReentrantLock等，并发容器ConcurrentHashMap

jvm如何实现cas的，调本地方法c/c++，配合硬件cpu实现，cpu会通过锁总线，不让其他处理器访问内存；或者

如果数据在cpu缓存中的状态是Exclusive的，可以直接修改(mesi协议)

cpu储存：寄存器 - l1l2缓存 - l3缓存(多核共享) - 主存

##### AQS

抽象队列同步器，juc包中的一个抽象类，为构建锁和同步器提供了一个通用的框架

用于实现ReentrantLock，Semaphore，countdownlatch等

AQS的核心思想：用一个状态变量来控制同步器的状态，状态变量会通过CAS更新；

同时维护一个等待队列(双向链表)来管理等待获取同步资源的线程。每个节点代表一个等待获取同步状态的线程，节点包含线程引用、等待状态等信息。

```
private volatile int state;
```

如果被请求的资源空闲，就把当前请求资源的线程置为工作线程，并把共享资源设为锁定状态；否则把线程阻塞等待，放入队列，等待唤醒及锁分配

##### semaphore

用于控制同时访问资源的线程数量，某段代码最多可以有n个线程访问，超出n等待，等某个线程执行完毕，下一个线程再进入；lock和synchronize一次只允许一个线程访问资源

```java
Semaphore semaphore = new Semaphore(3);
ExecutorService executorService = Executors.newCachedThreadPool();
for (int i = 0; i < 10; i++) {
    executorService.execute(()->{ //提交10个任务，因为是cached，所以创建了10个线程
        try {
            semaphore.acquire(); //最多3个线程同时执行
            System.out.print(semaphore.availablePermits() + " ");
        } catch (InterruptedException e) {
        } finally {
            semaphore.release();
        }
	});
}
```

##### CountDownLatch

允许count个线程被阻塞在一个地方，直到所有线程的任务执行完毕。不可重用。

底层：把AQS的state设置为count，当线程使用 `countDown()`方法时，CAS去改变state的值，直至为0；调用`await()` 方法的时候，如果 `state` 不为 0，就阻塞，其后语句不执行

场景：多线程处理多文件，最后需要将处理结果整合

**分段锁**

concurnent hashmap用了分段锁，共享资源被分成多个段或者区域，每个区域都有一个对应的锁。多个线程需要同时访问资源的不同部分时，它们可以并发地进行，因为只需要获取该区域对应的锁，而不是整个资源的锁。优点是提高并发性能，减少锁竞争的激烈程度，缺点是管理多个锁的复杂性。

**滑块锁**

和分段锁差不多，不过更高级，会根据当前的并发访问情况来动态地调整滑块的位置和大小，以最大程度地减少锁竞争。

会采用一些策略来决定何时以及如何调整锁的粒度。这些策略可以基于一些指标，如锁竞争的程度、线程的等待时间等。



### 手撕多线程

[八个经典的java多线程编程题目-CSDN博客](https://blog.csdn.net/shinecjj/article/details/103792151)

##### 线程1执行完线程2执行再main

```
在main线程中执行t.join想当于等待t执行完，main才执行
底层类似main线程调用t.join后，把自己lock.wait,然后等待t死亡后，被notify
t1.start();
t1.join();
t2.start();
t2.join();
```

##### 三个线程循环打印abc

线程1打a，2打b，3打c，循环10次 （线程1打印奇数，线程2打印偶数类似）

```
// 法1：synchornize+wait notify
// 线程2notify后，线程1不会立即执行，而是相当于切换了一个状态，之后继续抢锁，所以要等待线程2释放锁之后才有机会继续执行
// 保证count判断、输出、改变count - 是原子的，防止线程a打印完，改变
class printAB{
	int count=0
	public synchonize void printA(){ 
		if(count%2==0){
			sout;
			count++;
			notifyAll();
		}else{
			this.wait();
		}
	}
}
class main{
	PrintAB printAB = new PrintAB();
	Thread thead1 = new Thread(printAB::printA);
}
```

```
法2：ReentrantLock+condition，本质没啥区别，具体见leet
相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。
void print：
	lock.lock()
		if(flag!=1) conditionA.await()
		print(); flag = 2; conditionB.signal()
	lock.unlock()
```

##### 10个线程，第一个从1加到10，最后结果相加

```
class Task{
	Task(tens){}
	int add(){
		for(i:10) result+=tens*10+i;
	}
}
main{
	Task[]; Thread[]
	for(i:10)  
		new Task(i); threads[i] = thread;
		new Thread(Task::add) addTasks[i] = addTask;
		thread.start()
	threads:for
		thread.join()
	tasks:for
		sum+=task.getResult
}
法2：join改countDownLatch
```

##### todo手撕阻塞队列

##### todo手撕线程池

##### 互斥锁如何实现读写锁

一个计数器，计数器改变用一个互斥锁，读拿锁，改变计数器的值

一个互斥锁做写锁，写加锁，再拿计数器的锁，看值是不是0；

# jvm

#### jit

JIT（Just-In-Time Compilation，即时编译），jvm检测哪些代码被频繁执行(热点代码)

然后jit把热点代码编译为本地机器码（正常是解释字节码），下次就可以直接执行机器码了

**为什么要了解？**

java是由虚拟机自动管理内存的，一旦出现内存泄漏和溢出方面的问题，如果不了解虚拟机的内存分布，排查问题会是非常困难的。

当垃圾收集成为系统达到更高并发的瓶颈时，我们就需要对这些“自动化”的技术实施必要的监控和调节

**需了解** GC分代回收的思想和依据以及不同垃圾回收算法的回收思路和适合场景

[理想汽车二面：说说一个java文件从加载到执行的过程，运行时数据区域的划分 (qq.com)](https://mp.weixin.qq.com/s?__biz=MzU0OTkxNzA4Mw==&mid=2247483923&idx=1&sn=b8fa31d6d4e42720f77e123d44a7212a&chksm=fba9dae7ccde53f1eed54a8db36c74d6dba8043464ad4fe4866f16ed2a7b14f6d9a8f5faeba8&scene=21#wechat_redirect)

[米哈游二面，反复考察的类加载器，竟然这么简单 (qq.com)](https://mp.weixin.qq.com/s/mRhKxvgUlmU_RtnGPYsvng)

**为什么java跨平台性能好**

JVM从软件层面屏蔽了不同的操作系统在底层硬件和指令上的区别，针对同一份HelloWorld.class文件，JVM会根据不同的操作系统分别生成其对应的机器码。

#### **java的对象分配规则（对象怎么创建）**

类加载--验证--准备（默认值）--初始化（静态变量，代码块）--创建对象（分配空间--设置对象头---执行构造方法）

类加载 - 分配内存 - 零值初始化 - 设置对象头 -构造函数初始化 - 返回引用 - 可达性分析标记为不可达 - 堆内存不足垃圾回收

先看类是否被加载过，申请内存，赋默认值，再对属性赋初始值，最后将对象的引用地址赋值给变量

1.内存空间分配，一般堆内存（可能在栈中，涉及逃逸分析，对象的引用不会逃逸到方法调用栈之外，即不会被其他线程引用或返回给其他方法）

2.分配对象头，包括一些对象的元信息，如**哈希码，锁状态，垃圾回收信息**

3.零值初始化，所有成员变量初始化为0值

4.构造函数调用，初始化对象

5.返回对象引用

**对象在堆中的储存布局**

8字节的markword对象头（用于存储对象的元数据信息，如对象的哈希码、锁状态、GC标记等）

4字节的classpoint，指向T.class对象的指针。

成员变量 						对齐填充（对象大小必须是8字节的整数倍）

#### **类加载过程**

(JVM上运行如何处理class文件)+

1.根据全类名找到字节码文件，方法区分配内存，类加载器把字节码加载到内存，创建一个class对象储存类的元数据

2.验证字节码文件；为静态变量分配内存，赋默认值；把相关的方法或字段转换为直接引用（内存地址），以便快速访问

3.执行类的初始化代码，为静态变量赋初值，执行静态代码块。同时，JVM会保证类的初始化是线程安全的。（加锁了）

补：类加载器把class加载到内存，class对象在方法区，指向class文件，实例对象在堆，指向class对象

补：代码执行过程 编译成class--类加载器加载---运行时数据区储存信息---执行引擎执行代码（开启main线程）

#### **双亲委派机制** 

（什么时候加载一个类）

首先jvm是动态加载（懒加载）由类加载器把class文件加载到内存

自底向上**检查**该类是否已经被加载，自顶(父类)向下进行实际查找和**加载**。

当一个class文件需要被加载到内存时，首先调用自定义加载器，检查是否被自己加载过，记载过直接返回class对象，如果没有，调用父类加载器判断，一直推到顶层加载器Bootstrap，如果还是没被加载过，则调用findClass方法查看自己的加载范围，判断类是否属于该范围，如果是就自己加载，否则向下抛给子加载器

**优点：**

目的是为了**避免重复加载，保证类的唯一性和安全性**。防止核心类库被篡改

不同的类加载器加载相同的类，也会被是为不同的类，因为每个加载器都有自己的类命名空间，可以防止系统类库被篡改或替换，因为即使有人尝试加载一个与系统类库同名的类，它也不会覆盖系统类库。

补：类加载器是组合关系不是继承关系。双亲委派可以被打破，在某些应用服务器中，为了实现热部署（即不重启服务器就能更新应用程序）会使用特殊的类加载器来加载和卸载类。

```java
Class<?> loadClass(String name){
	//看自己加载过没有
    Class c = findLoadedClass(name);
    if(c==null){
    	// 去看父加载过没有
        if(parent!=null){
            parent.loadClass(name)
        }else{
            c = findBootstrapClassOrNull(name);
        }
        // 如果父类不能加载，看自己能不能加载
        if(c==null){
        	c = findClass(name)
        }
    }
}
```

**四种类加载器，具体都是干嘛的**

启动类加载器（**Bootstrap ClassLoader**）：它是JVM的一部分，用于加载Java核心类库，如java.lang中的类（object,thread,string,system）,他们使用本地代码(c++)实现的，不是Java类，因此在Java中无法直接获取对其的引用。

扩展类加载器（ext classLoader）：它是由Java实现的。

应用程序类加载器(App ClassLoader)：也称为系统类加载器，负责加载应用程序classpath下的类。

自定义类加载器：用户可以根据需要创建自己的类加载器，以加载特定位置或方式的类文件。

#### **JVM内存模型（运行时数据区）：**+++

线程独占：程序计数器、虚拟机栈，本地方法栈， 线程共享：堆，方法区

**程序计数器**（下一条要执行的指令的地址）：字节码的行号指示器，用于选取下一条要执行的字节码指令，实现顺序，分支，循环，异常处理等操作，**没有oom**

补：native方法：本地机器代码实现的计算机程序方法（常指c++），而非高级程序语言，用以提高性能与底层系统交互

**虚拟机栈**（函数上下文，局部变量）：为虚拟机执行 Java 方法 （也就是字节码）服务，每个线程对应一个栈，每个栈有多个栈帧，每个方法一个栈帧，储存方法的局部变量表，返回地址等

方法被调用创建一个栈帧放入栈顶

**本地方法栈**（和虚拟机栈一样，但是为本地方法服务）则为虚拟机使用到的 Native 方法服务

可能出现`StackOverFlow`栈内存不允许动态扩展，请求深度超过栈最大深度，或`oom`，可以动态扩展，扩展时无法申请足够的内存空间

**堆**（储存对象实例）：jdk1.7之后默认开启逃逸分析，如果某方法的对象引用没有被返回或被外面使用，可以直接在栈上分配内存

是垃圾收集器的主要管理区域，因此也叫gc堆，通常被分为新生代（**eden、from survivor、to survivor**）（1/3堆内存），老生代（2/3），元空间（直接内存区域），是出现oom错误的位置

**方法区（元空间）**：储存已被虚拟机加载的**类的加载信息及元信息，常量池，静态变量，方法字节码**（字符串常量池在堆空间，运行时常量池在方法区，方法区元空间实现）内存大小受限于操作系统为java虚拟机分配的逻辑内存空间。

元空间，它使用的是本地内存（Native Memory），也就是堆外内存。

补：类的元信息类（存在方法区）类的名称、访问修饰符、父类、实现的接口

**堆外内存**

不是方法区，方法区是堆的一个逻辑部分，也受jvm内存管理，分配回收等

堆外内存不受jvm回收机制，主要用于一些对性能要求较高的场景，如提高 I/O 操作效率、减少垃圾回收对应用程序的影响等（减少垃圾回收停顿，常用于高实时的系统）。

一般通过malloc在内核分配空间，再mmap映射到用户，使得jvm进程可以访问，io就不用从内核拷到用户了

#### **新生代和老年代**

eden：伊甸园，新生儿，survivor：幸存者

多数情况下，对象在eden区分配，没有足够空间时，发起一次Minor GC，内存不够时通过 **分配担保机制** 把新生代的对象提前转移到老年代中去，如果对象在 Eden 出生并经过第一次 Minor GC 后仍然能够存活，就进入survivor，年龄置为1，之后每经历依次minor GC，年龄+1，年龄增加到一定程度（默认15）就进入老年代

大对象（需要大量连续内存空间的对象，如：字符串、数组）直接进入老年代

补：为什么年龄默认是15？2^4-1,四个比特位；固定大小空间节省内存，快速判断，比较。

补1：minorGC看你的是整个新生代，但发生于eden空间不足

补2：什么时候进入老生代：survivor空间不足，大于某年龄的进入老年代；大对象直接老年代

当进行垃圾回收时，虚拟机会按照对象的年龄从小到大对其所占用的空间进行累积。**当某个年龄组的累积大小超过了Survivor区的一半时，系统会考虑提升这个年龄组的对象到老年代**。提升的年龄阈值是由当前年龄和配置中的`MaxTenuringThreshold`中较小的一个确定的。

- 新生代收集（Minor GC / Young GC）：只对新生代进行垃圾收集；

+ 整堆收集 (Full GC)：收集整个 Java 堆和方法区。

- **Minor GC 触发条件**：当新生代中的 Eden 区内存空间不足时，会触发 Minor GC，对新生代进行垃圾回收。
- **Full GC 触发条件**：老年代内存空间不足；调用`System.gc()`方法；动态年龄判断，当 Survivor 区中相同年龄的对象大小总和超过 Survivor 区的一半时，年龄大于或等于该年龄的对象会直接进入老年代，如果老年代空间不够，就会触发 Full GC。

经常出现youngGC怎么办？

调整新生代的大小，

避免创建过多的临时对象，及时释放不再使用的对象

避免大对象的频繁创建

**为什么要分代**

为了分代垃圾回收，新生代死的快，标记复制；老生代存活时间长，死的慢，标记整理或标记清除

**JVM异常问题？**

StackOverflowError（线程请求栈深度超过虚拟机所允许的最大深度）

OutOfMemoryError（堆内存不够用）（1.花很长时间执行垃圾回收只能回收很少的堆内存 2.创建新对象时，堆空间不足以存放新对象）

PermGen space（方法区内存不够用）

#### 死亡对象判断方法

一个对象没有任何引用指向它，他就会被回收，这个过程由JVM的垃圾回收器自动完成。

引用计数器，每当有一个地方引用它时，计数器加 1；当引用失效时，计数器减 1。当计数器为 0 时，对象被认为是死亡的，但这种你方法无法解决循环引用

**可达性分析**算法（从GC Roots对象开始，通过一系列的引用链来遍历所有的对象，如果一个对象不可达，则说明它已经死亡，可以被回收了）

**垃圾回收算法有哪些？**

标记清除法、标记复制算法、标记整理法、分代收集算法

新生代回收率高，老生代回收率低。整理复制移动的都是存活的

1.会产生大量不连续的空间碎片

2.把所有存活的向一端移动，清理调边界以外的

3.分半区，浪费空间

4.新生代使用复制算法（每次大批对象死去，只有少量存活，复制成本低）

老年代使用标记清除算法或者标记整理（因为回收频率不高）算法

#### **垃圾回收器**

Serial收集器，ParNe（**Parallel并行**）CMS（Concurrent Mark Sweep）收集器，G1 收集器

1. Serial收集器：单线程的垃圾回收器，使用标记-复制算法，适合小型应用程序或客户端应用程序。
2. Parallel收集器：多线程的垃圾回收器，使用标记-复制算法，适合在后台运行的中型应用程序。
3. CMS收集器：并发垃圾回收器，使用**标记-清除**算法，适合对响应时间有要求的中型应用程序。
4. G1收集器：并发垃圾回收器，使用**标记-整理**算法，适合对响应时间有要求且堆内存较大的应用程序。

其中，Serial收集器和Parallel收集器是新生代收集器，CMS和G1是老年代收集器。

G1的出现是为了替换CMS，现在默认是CMS

- **G1**：将堆内存划分为多个大小相等的 Region，每个 Region 可以是新生代、老年代或者空闲区域，G1 会动态地调整各个 Region 的角色。

#### G1 把内存分为一块一块的原因

- **更好的内存管理**：把内存划分为多个 Region 后，G1 可以更灵活地管理内存，能够根据不同 Region 的使用情况进行垃圾回收，避免了传统垃圾回收器在整个堆内存上进行回收的低效问题。
- **预测停顿时间**：G1 可以根据每个 Region 的垃圾占比，优先回收垃圾较多的 Region，从而可以预测垃圾回收的停顿时间，满足对响应时间要求较高的应用场景。
- **并行和并发处理**：多个 Region 可以并行地进行垃圾回收，提高了垃圾回收的效率，同时 G1 还支持并发标记，减少了对用户线程的影响。

**CMS和G1的区别**

CMS收集器是老年代的收集器，可以配合新生代的Serial和ParNew收集器一起使用
G1收集器收集范围是老年代和新生代。不需要结合其他收集器使用

**内存碎片**

取决于用了什么算法，标记删除就会产生内存碎片

连续内存能用局部性原理，但可能因为内存对其导致空间浪费？todo

