# 线程的基础概念

## 基础概念

### 线程和进程的区别

1. 根本不同：进程是运行中的程序，是操作系统分配的资源，线程是CPU调度的基本单位
2. 数量不同：线程是依附于进程的，一个进程中有一个或多个线程
3. 资源方面：进程之间的资源通常是独立的，同一个进程中线程共享进程中的一些资源，线程同时拥有自身独立的存储空间（一般存储在CPU中）
4. 开销不同：线程的创建、销毁、切换比起进程来说要快很多，线程之间的通信比较方便，进程之间的通信很麻烦，通常要借助内核才可以实现

### 同步异步、阻塞非阻塞

同步和异步：执行某个功能后，被调用者是否会主动反馈信息

阻塞和非阻塞：执行某个功能后，调用者是否需要一直等待结果的反馈

- 同步阻塞：比如用锅烧开水，水开后，不会主动通知你。烧水开始执行后，你需要一直等待水烧开
- 同步非阻塞：比如用锅烧水，水开后，不会主动通知你。烧水开始执行后，不需要一直等待水烧开，可以去执行其他功能，但是需要时不时的查看水烧开没
- 异步阻塞（不常见）：比如用水壶烧水，水开后，会主动通知你水烧开了。烧水开始执行后，需要一直等待水烧开
- 异步非阻塞：比如用水壶烧水，水开后，会主动通知你水烧开了。烧水开始执行后，不需要一直等待水烧开，可以去执行其他功能

## 线程的使用

## 操作系统中线程的状态

![](.\pic\juc\操作系统中线程的状态.png)

### java中线程的状态

![](E:\转正\学习\study note\重学\pic\juc\java中线程的状态.png)

### 线程常用方法

- 获取当前线程

- 线程的名字

- 线程的优先级

- 线程的让步

  Thread.yield()，线程进入就绪状态，随时可以再被CPU调度

- 线程的休眠

- 线程的强占

  Thread的非静态方法join

  需要在某一个线程中调用这个方法

  如果在main线程中调用了t1.join()，那么main线程会进入到等待状态，需要等待t1线程全部执行完毕，在恢复到就绪状态等待CPU调度

  如果在main线程中调用了t1.join(2000)，那么main线程会进入到等待状态，需要等待t1执行2s后，再恢复到就绪状态等待CPU调度。如果在等待期间，t1已经结束了，那么main线程自动变为就绪状态等待CPU调度。

- 守护线程

  默认情况下，线程都是非守护线程

  JVM会在程序中没有非守护线程时，结束掉当前JVM

  主线程默认是非守护线程，如果主线程执行结束，需要查看当前JVM内是否还有非守护线程，如果没有JVM直接停止

  GC执行后，被回收的对象会执行finalize方法，但是JVM不保证finalize方法一定被执行，因为执行这个方法的线程就是守护线程

- 线程的等待和唤醒

  可以让获取synchronized锁资源的线程通过wait方法进去到锁的**等待池**，并且会释放锁资源

  可以让获取synchronized锁资源的线程通过notify或者notifyAll方法，将等待池中的线程唤醒，添加到**锁池**（可以竞争锁的池）中

  notify随机唤醒等待池中的一个线程到锁池

  notifyAll将等待池中的全部线程都唤醒，并且添加到锁池

  在调用wait方法和notify和notifyAll方法时，必须在synchronized修饰的代码块或者方法内部才可以，因为要操作基于某个对象的锁的信息维护

### 线程的结束方法

线程的结束方式有很多，最常用的就是让线程的run方法结束，无论是return结束，还是抛出异常结束

#### stop方法（不用）

强制让线程结束，无论你在干嘛，不推荐使用这种方式，，已过时，但是他确实可以将线程干掉

#### 使用共享变量（很少会用）

这种方式用的也不多，有的线程可能会通过死循环来保证一直运行

可以通过修改共享变量破坏死循环，让线程退出循环，结束run方法

#### interrupt方式

通过打断WAITING或者TIMED_WAITING状态的线程，从而抛出异常自行处理

这种停止线程方式是最常用的一种，在框架和JUC中也是最常见的

```java
public static void main(String[] args) throws InterruptedException {
    Thread thread = new Thread(() -> {
        while (true) {
            // 获取任务 执行任务
            // 没有任务 休眠
            try {
                Thread.sleep(10000);
            } catch (InterruptedException e) {
                e.printStackTrace();
                System.out.println("线程被中断了");
                return;
            }
        }
    });
    thread.start();
    Thread.sleep(500);
    thread.interrupt();
}

public static void main(String[] args) throws InterruptedException {
    Thread thread = new Thread(() -> {
        while (true) {
            // 获取任务 执行任务
            // 没有任务 休眠
            synchronized (TestThreadStop.class) {
                try {
                    TestThreadStop.class.wait();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                    System.out.println("线程被中断了");
                    return;
                }
            }
        }
    });
    thread.start();
    Thread.sleep(500);
    synchronized (TestThreadStop.class) {
        thread.interrupt();
    }
}
```

## 并发编程的三大特性

### 原子性

#### 什么是并发编程的原子性

JMM（Java Memory Model），不同的硬件和不同的操作系统在内存上的操作有一定差异，Java为了解决相同代码在不同操作系统上出现的各种问题，用JMM屏蔽掉各种硬件和操作系统带来的差异

让java的并发编程可以做到跨平台

JMM规定所有变量都会存储在主内存中，在操作的时候，需要从主内存中复制一份到线程内存（CPU内存），在线程内部做计算，**然后再写回主内存中（不一定）**

**原子性的定义：原子性指一个操作是不可分割的，不可中断的，一个线程在执行时，另一个线程不会影响到他**

**原子性：多线程操作临界资源，预期的结果与最终结果一致**

i++,通过对i++的分析，可以查看出，++的操作，一共分为了三步，首先是线程从主内存拿到数据保存到CPU的寄存器中，然后在寄存器中进行+1操作，最终将结果写回到主内存当中

#### 保证并发编程的原子性

##### synchronized

synchronized可以让避免多线程同时操作临界资源，同一时间点，只会有一个线程正在操作临界资源

##### CAS

compare and swap也就是比较和交换，他是一条CPU的并发原语

他在替换内存中的某个位置的值时，首先查看内存中的值与预期值是否一致，如果一致，执行替换操作，这个操作是一个原子性操作

>  Java中基于Unsafe的类提供了对CAS的操作的方法，JVM会帮助我们将方法实现CAS汇编指令

但是要清楚CAS只是比较和交换，在获取原值的这个操作上，需要自己实现

> Doug Lea在CAS的基础上帮助我们实现了一些原子类，其中就包括现在看到的AtomicInteger，还有其他很多原子类

**CAS的缺点**：

- ABA问题

  第三个操作从A->B的线程并不清楚中间发生了A->B B->A，可以引入版本号的方式，来解决ABA的问题。Java中提供了一个类在CAS时，针对各个版本追加版本号的操作。AtomicStampeReference

- 自旋时间过长问题

  - 可以指定CAS一共循环多少次，如果超过这个次数，直接失败/或者挂起线程（自旋锁、自适应自旋锁）
  - 可以在CAS一次失败后，将这个操作暂存起来，后来需要获取结果时，将暂存的操作全部执行，再返回最后的结果

- 只能对一个变量进行原子性操作

##### Lock锁

ReentrantLock可以直接对比synchronized，在功能上来说，都是锁

但是ReentrantLock的功能性相比synchronized更丰富

ReentrantLock底层是基于AQS实现的，有一个基于CAS维护的state变量来实现锁的操作

##### ThreadLocal

ThreadLocal实现原理：

- 每个Thread中都存储着一个成员变量，ThreadLocalMap
- ThreadLocal本身不存储数据，像是一个工具类，基于ThreadLocal去操作ThreadLocalMap

- ThreadLocalMap本身就是基于Entry[]实现的，因为一个线程可以绑定多个ThreadLocal，这样一来，可能需要存储多个数据，所以采用Entry[]的形式实现

- 每一个线程都有自己独立的ThreadLocalMap，再基于ThreadLocal对象本身作为key，对value进行存取

- ThreadLocalMap的key是一个弱引用，弱引用的特点是：即便有弱引用，在GC时，也必须被回收。这里是为了在ThreadLocal对象失去引用后，如果key的引用是强引用，会导致ThreadLocal对象无法被回收

  > 强引用：GC无法回收，内存满了抛出OOM异常
  >
  > 软引用：内存满了才会回收
  >
  > 弱引用：GC时被回收，无论内存满没满
  >
  > 虚引用：

ThreadLocal内存泄漏问题：

- 如果ThreadLocal引用丢失，key因为弱引用会被GC回收掉，如果同时线程还没有被回收，就会导致内存泄漏，内存中的value无法被回收，同时也无法获取到
- 只需要在使用完毕ThreadLocal对象之后，及时的调用remove方法，移除Entry即可

![](E:\转正\学习\study note\重学\pic\juc\ThreadLocalMap.png)

### 可见性

#### 什么是可见性问题

可见性问题是基于CPU位置出现的，CPU处理速度非常快，相对CPU来说，去主内存获取数据这个事情太慢了，CPU就提供了L1，L2，L3的三级缓存，每次去主内存拿完数据后，就会存储到CPU的三级缓存，每次去三级缓存拿数据，效率肯定会提升

这就带来了问题，现在CPU都是多核，每个线程的工作内存（CPU三级缓存）都是独立的，会导致每个线程中做修改时，只改自己得工作内存，没有及时的同步到主内存，导致数据不一致问题

#### 解决可见性的方式

1. volatile

   volatile是一个关键字，用来修饰成员变量

   如果属性被volatile修饰，相当于会告诉CPU，对当前属性的操作，不允许使用CPU的缓存，必须去和主内存操作

   volatile的内存语义：

   - volatile属性被写：当写一个volatile变量，JMM会将当前线程对应的CPU缓存及时的刷新到主内存中
   - volatile属性被读：当读一个volatile变量，JMM会将对应的CPU缓存中的内存设置为无效，必须去主内存中重新读取共享变量

   其实加了volatile就是告知CPU，对当前属性的读写操作，不允许使用CPU缓存，加了volatile修饰的属性，会在转为汇编之后，追加一个lock的前缀，CPU执行这个指令时，如果带有lock前缀会做两个事情：

   - 将当前处理器缓存行的数据写回到主内存
   - 这个写回的数据，在其他的CPU内核的缓存中，直接无效

   总结：volatile就是让CPU每次操作这个数据时，必须立即同步到主内存，以及从主内存读取数据

2. synchronized

   synchronized也是可以解决可见性问题的，synchronized的内存语义

   如果涉及到了synchronized的同步代码块或者是同步方法，获取锁资源之后，将内部涉及到的变量从CPU缓存中移除，必须去主内存中重新拿数据，而且在释放锁之后，会立即将CPU缓存中的数据同步到主内存

3. Lock

   Lock锁保证可见性的方式和synchronized完全不同，synchronized基于他的内存语义，在获取锁和释放锁时，对CPU缓存做一个同步到主内存的操作

   Lock锁是基于volatile实现的，Lock锁内部再进行加锁和释放锁时，会对一个由volatile修饰的state属性进行加减操作

   如果对volatile修饰的属性进行写操作，CPU会执行带有lock前缀的指令，CPU会将修改的数据，从CPU缓存立即同步到主内存，同时也会将其他的属性也立即同步到主内存中，还会将其他CPU缓存行中的这个数据设置为无效，必须重新从主内存中拉取

   > 以我的理解是只要对volatile修饰的属性进行写操作了，就会重新从主内存中重新读取

4. final

   final修饰的属性，再运行期间是不允许修改的，这样一来，就间接的保证了可见性

   final并不是说每次读取数据从主内存读取，他没有这个必要，而且final和volatile是不允许同时修饰一个属性的

   final修饰的内容已经不允许再次被写了，而volatile是保证每次读写数据去主内存读取，并且volatile会影响一定的性能

### 有序性

#### 什么是有序性

在Java中，.java文件中的内容会被编译，在执行前需要再次转为CPU可以识别的指令，CPU在执行这些指令时，为了提升执行效率，在不影响最终结果的前提下（满足一些要求），会对指令进行重排

Java中的程序是乱序执行的

*单例模式由于指令重排序可能会出现问题*：

单例初始化分为三步，分配内存空间，初始化，地址赋值给指针，地址赋值给指针可能排在初始化前面，线程可能会拿到没有初始化的对象，导致在使用时，可能由于内部属性为默认值，导致出现一些不必要的问题

#### as-if-serial

as-if-serial语义：

不论指令如何重排序，需要保证单线程的程序执行结果是不变的

而且如果存在依赖的关系，那么也不可以做指令重排

```java
// 这种情况肯定不能做指令重排序
int i = 0;
i++;
```

#### happens-before

有一些规则，不需要过分关注

#### volatile

如果需要让程序对某一个属性的操作不出现指令重排，除了满足happens-before原则之外，还可以基于volatile修饰属性，从而对这个属性的操作，就不会出现指令重排的问题了

volatile如何实现的禁止指令重排

内存屏障概念，将内存屏障看成一条指令

会在两个操作之间，添加上一道指令，这个指令就可以避免上下执行的其他指令进行重排序

## 锁

### 锁的分类

#### 可重入锁、不可重入锁

Java中提供的synchronized，ReentrantLock，ReentrantReadWriteLock都是可重入锁

重入：当前线程获取到A锁，在获取之后尝试再次获取A锁是可以直接拿到的

不可重入：当前线程获取到A锁，在获取之后尝试再次获取A锁，无法获取到的，因为A锁被当前线程占用着，需要等待自己释放锁再获取锁

#### 乐观锁、悲观锁

Java中提供的synchronized，ReentrantLock，ReentrantReadWriteLock都是悲观锁

Java中提供的CAS操作，就是乐观锁的一种实现

- 悲观锁：获取不到锁资源时，会将当前线程挂起（进入BLOCKED、WAITING），线程挂起会涉及到用户态和内核态的切换，而这种切换是比较消耗资源的

  - 用户态：JVM可以自行执行的指令，不需要借助操作系统执行
  - 内核态：JVM不可以自行执行，需要操作系统才可以执行

- 乐观锁：获取不到锁资源，可以再次让CPU调度，重新尝试获取锁资源

  Atomic原子性类中，就是基于CAS乐观锁实现的

#### 公平锁、非公平锁

Java中提供的synchronized只能是非公平锁

Java中提供的ReentrantLock，ReentrantReadWriteLock可以实现公平锁和非公平锁

- 公平锁：线程A拿到了锁资源，线程B没有拿到，线程B去排队，线程C来了，锁被A持有，同时线程B在排队，直接排到B的后面，等待B拿到锁资源或者是B取消后，才可以尝试去竞争锁资源
- 非公平锁：线程A获取到了锁资源，线程B没有拿到，线程B去排队，线程C来了，先尝试竞争一波
  - 拿到锁资源：开心，插队成功
  - 没有拿到锁资源：依然要排到B的后面，等待B拿到锁资源或者是B取消后，才可以尝试去竞争锁资源

#### 互斥锁、共享锁

Java中提供的synchronized、ReentrantLock是互斥锁

Java中提供的ReentrantReadWriteLock，有互斥锁也有共享锁

- 互斥锁：同一时间点，只会有一个线程持有着当前互斥锁
- 共享锁：同一时间点，当前共享锁可以被多个线程同时持有

### 深入synchronized

#### 类锁、对象锁

synchronized的使用一般就是同步方法和同步代码块

synchronized的锁是基于对象实现的

如果使用同步方法

- static：此时使用的是当前类.class作为锁（类锁）
- 非static：此时使用的是当前对象作为锁（对象锁）

#### synchronized的优化

在JDK1.5的时候，Doug Lee推出了ReentrantLock，lock的性能远高于synchronized，所以JDK团队在JDK1.6中，对synchronized做了大量的优化

- 锁消除：在synchronized修饰的代码中，如果不存在操作临界资源的情况，会触发锁消除，即便写了synchronized，也不会触发

  ```java
  public synchronized void method() {
      // 没有操作临界资源
      // 此时这个方法的synchronized可以认为没有
      System.out.println("23");
  }
  ```

- 锁膨胀：如果在一个循环中，频繁的获取和释放锁资源，这样带来的消耗很大，锁膨胀就是将锁的范围扩大，避免频繁的竞争和获取锁资源带来的不必要的开销

  ```java
  public void a() {
      for (int i = 0; i < 999999; i++) {
          synchronized (对象) {
              
          }
      }
      
      // 这时上面的代码会触发锁膨胀
      synchronized (对象) {
          for (int i = 0; i < 99999l i++) {
              
          }
      }
  }
  ```

- 锁升级：ReentrantLock的实现，是先基于乐观锁的CAS尝试获取锁资源，如果拿不到锁资源，才会挂起线程（切换内核态）。synchronized在JDK1.6之前，获取锁通过ObjectMonitor切换到内核态，获取不到挂起当前线程，所以synchronized性能才比较差

  > synchronized锁在优化前就是一个地地道道的重量级锁，重量级锁的实现其实就是依赖于多线程争夺Monitor锁的拥有权，而Monitor锁的实现则依赖于操作系统底层的互斥原语mutex，因此每一次获取Monitor锁的时候都会经过用户态到内核态的切换，性能很低。这便是重量锁的由来。

  synchronized就在JDK1.6做了锁升级的优化
  
  - 无锁、匿名偏向：当前对象没有作为锁存在
  - 偏向锁：如果当前锁资源，只有一个线程在频繁的获取和释放，那么这个线程过来，只需要判断，当前指向的线程是否是当前线程
    - 如果是，直接拿着锁资源走
    - 如果当前线程不是，基于CAS的方式，尝试将偏向锁指向当前线程，如果获取不到，触发锁升级，升级为轻量级锁（偏向锁状态出现了锁竞争的情况）
  - 轻量级锁：会采用自旋锁的方式去频繁的以CAS的形式获取锁资源（采用的是**自适应自旋锁**）
    - 如果成功获取到，拿着锁资源走
    - 如果自旋了一定时间，没拿到锁资源，锁升级
  - 重量级锁：就是最传统的synchronized方式，通过monitor锁，如果拿不到锁资源，就挂起当前线程（用户态&内核态）

#### synchronized实现原理

synchronized是基于对象实现的

对象在堆内存中的存储结构：

对象头

类引用

实例数据

对象填充

![](E:\转正\学习\study note\重学\pic\juc\对象头.png)

MarkWord中标记着四种锁的信息：无锁、偏向锁、轻量级锁、重量级锁

<p style="color: red">hashcode是懒加载的，在调用hashcode方法后才会保存在对象头中，如果对象已经存了hashcode的值，就不会变成偏向锁，而是直接变为轻量级锁，如果锁本身是偏向锁，在调用hashcode就会直接升级为重量级锁</p>

<p style="color: red">轻量级锁和重量级锁的分代年龄存在GC元数据中？</p>

#### synchronized的锁升级

为了可以在Java中看到对象头的MarkWord信息，需要导入依赖

```xml
<dependecy>
	<groupId>org.openjdk.jol</groupId>
    <artifacId>jol-core</artifacId>
    <version>0.9</version>
</dependecy>
```

锁默认情况下，开启了偏向锁延迟

> 偏向锁在升级为轻量级锁时，会涉及到偏向锁撤销，需要等到一个安全点（STW，创建Lock Record？），才可以做偏向锁撤销，在明知道有并发的情况下，就可以选择不开启偏向锁，或者是设置偏向锁延迟开启
>
> 因为JVM在启动时，需要加载大量的.class文件到内存中，这个操作会涉及到synchronized的使用，为了避免出现偏向锁撤销操作，JVM启动初期，有一个延迟5s开启偏向锁的操作
>
> 如果正常开启了偏向锁了，那么不会出现无锁状态，对象会直接变为匿名偏向

锁升级状态的转变：

![](E:\转正\学习\study note\重学\pic\juc\锁升级.jpg)

Lock Record和ObjectMonitor：

![](E:\转正\学习\study note\重学\pic\juc\LockRecord和ObjectMonitor.jpg)

#### 重量锁底层ObjectMonitor

需要去找到openjdk，在百度中直接搜索openjdk，第一个链接就是

找到ObjectMonitor的两个文件，hpp，cpp

先查看核心属性

```hpp
  // initialize the monitor, exception the semaphore, all other fields
  // are simple integers or pointers
  ObjectMonitor() {
    _header       = NULL; //存储着MarkWord 对象头信息 年龄 hashcode
    _count        = 0; // 竞争锁的线程个数
    _waiters      = 0, // wait的线程个数
    _recursions   = 0; // 标识当前synchronized锁重入的次数
    _object       = NULL;
    _owner        = NULL; // 持有锁的线程
    _WaitSet      = NULL; // 保存wait的线程信息，双向链表
    _WaitSetLock  = 0 ;
    _Responsible  = NULL ;
    _succ         = NULL ;
    _cxq          = NULL ; // 获取锁资源失败后，线程要放到当前的单向链表中
    FreeNext      = NULL ;
    _EntryList    = NULL ; // _cxq以及被唤醒的waitSet中的线程，在一定机制下，会放到EntryList
    _SpinFreq     = 0 ;
    _SpinClock    = 0 ;
    OwnerIsThread = 0 ;
    _previous_owner_tid = 0;
  }
```

### 深入ReentrantLock

#### ReentrantLock和synchronized的区别

核心区别：

- ReentrantLock是个类，synchronized是关键字，当然都是在JVM层面实现互斥锁的

效率区别：

- 如果竞争比较激烈，推荐ReentrantLock去实现，不存在锁升级概念。而synchronized是存在锁升级概念的，如果升级到重量级锁，是不存在锁降级的

底层实现区别：

- 实现原理是不一样，ReentrantLock基于AQS实现的，synchronized是基于ObjectMonitor

功能向的区别：

- ReentrantLock的功能比synchronized更全面
  - ReentrantLock支持公平锁和非公平锁
  - ReentrantLock可以指定等待锁资源的时间
  - ReentrantLock可以只尝试获取锁

选择哪个：如果你对并发编程特别熟练，推荐使用ReentrantLock，功能更丰富。如果掌握的一般般，使用synchronized会更好
