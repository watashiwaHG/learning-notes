# AQS

## 1. 什么是AQS

在java中是juc包下的一个抽象类AbstractQueuedSynchronizer，规定了一系列和锁相关的规范

```java
public abstract class AbstractQueuedSynchronizer
```

在java中有很多并发类就是实现了AbstractQueuedSynchronizer，比如：

1. CountDownLatch
2. ThreadPoolExecutor
3. ReentrantLock
4. ReentrantReadWriteLock
5. Semaphore

> 其实都是这些类中的某个静态内部类实现了AbstractQueuedSynchronizer

## 2.AQS中核心内容是什么

主要有三个属性：

1. state 

   ```java
   private volatile int state;
   ```

   

   > 在不同实现类中表达的意思不同，在ReentrantLock中代表当前锁是否被占用

2. 同步队列

   ```java
   /**
    * Head of the wait queue, lazily initialized.  Except for
    * initialization, it is modified only via method setHead.  Note:
    * If head exists, its waitStatus is guaranteed not to be
    * CANCELLED.
    */
   private transient volatile Node head;
   
   /**
    * Tail of the wait queue, lazily initialized.  Modified only via
    * method enq to add new wait node.
    */
   private transient volatile Node tail;
   ```

   > 如果线程没能抢到锁就进入同步队列等待，此队列为双向队列

3. 单向链表

   ```java
   public class ConditionObject implements Condition, java.io.Serializable {
       private static final long serialVersionUID = 1173984872572414699L;
       /** First node of condition queue. */
       private transient Node firstWaiter;
       // ...
   }
   ```

   > AQS中内部类ConditionObject的firstWaiter来管理，await和signal方法

## 3.ReentrantLock

- NonfairSync

  非公平锁，首先会抢一次锁，失败后再判断当前锁是否被占用，没被占用则再抢一次，如果被当前线程占用，state + 1，如果还是没有抢到则进入同步队列

  主要方法：

  - lock()

    ```java
    // 先CAS抢一次锁，成功则将exclusiveOwnerThread赋值为当前线程
    if (compareAndSetState(0, 1))
        setExclusiveOwnerThread(Thread.currentThread());
    // 失败则执行acquire方法
    else
        acquire(1);
    ```

    

  - tryAcquire()

    ```java
    return nonfairTryAcquire(acquires);
    // nonfairTryAcquire方法
    final Thread current = Thread.currentThread();
    int c = getState();
    // 如果锁没被占，则再CAS抢一次锁
    if (c == 0) {
        if (compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    // 如果锁被占，但占用锁的线程为当前线程，则state加1
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    // 没抢到锁，返回false
    return false;
    ```

    之后在队列中也是调用此方法尝试获取锁

- FairSync

  公平锁，首先判断锁是否被占用，如果没有则判断当前线程是否为同步队列的第一个，是则抢一次锁，否则判断占用锁的线程是否为当前线程，是则state + 1，没抢到锁则进入同步队列

  - lock

    ```java
    // 直接调用acquire
    acquire(1);
    ```

    

  - tryAcquire

    ```java
    // 与非公平锁几乎一样
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        // 判断当前线程是否为同步队列中的排头，是则抢锁
        if (!hasQueuedPredecessors() &&
            compareAndSetState(0, acquires)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    }
    else if (current == getExclusiveOwnerThread()) {
        int nextc = c + acquires;
        if (nextc < 0)
            throw new Error("Maximum lock count exceeded");
        setState(nextc);
        return true;
    }
    return false;
    ```

    之后在队列中也是调用此方法尝试获取锁

acquire方法

```java
public final void acquire(int arg) {
    // 执行子类重写的tryAcquire方法
    if (!tryAcquire(arg) &&
        // 失败则加入到同步队列中
        acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
        selfInterrupt();
}
```

addWaiter方法

```java
// 将当前线程封装到Node中
Node node = new Node(Thread.currentThread(), mode);
// Try the fast path of enq; backup to full enq on failure
Node pred = tail;
// tail默认为null，判断是否为第一次向同步队列中添加Node
if (pred != null) {
    // 不是第一次则将新添加的Node的prev指向原tail指向的节点
    node.prev = pred;
    // CAS将tail从原来的指向的节点改为指向新添加的Node
    if (compareAndSetTail(pred, node)) {
        // 原tail指向的节点的next指向新添加的next
        pred.next = node;
        return node;
    }
}
// 第一次添加则初始化同步队列，添加一个保安（排头Node，由head指针指向），再将新添加的Node放在保安的next
enq(node);
return node;
```

acquireQueued方法

```java
final boolean acquireQueued(final Node node, int arg) {
    // 记录
    boolean failed = true;
    try {
        boolean interrupted = false;
        // 无限循环获取锁和挂起？
        for (;;) {
            // 获取node的prev节点p
            final Node p = node.predecessor();
            // 如果p为head节点（保安）则代表当前节点已经是同步队列的排头，可以尝试获取锁
            if (p == head && tryAcquire(arg)) {
                // 获取锁成功后，则将当前节点变为head节点（保安）
                setHead(node);
                // 原保安节点的next置为null
                p.next = null; // help GC
                failed = false;
                return interrupted;
            }
            // 如果不是排头则挂起
            if (shouldParkAfterFailedAcquire(p, node) &&
                parkAndCheckInterrupt())
                interrupted = true;
        }
    } finally {
        if (failed)
            cancelAcquire(node);
    }
}
```

shouldParkAfterFailedAcquire挂起方法

```java
/** waitStatus value to indicate thread has cancelled */
static final int CANCELLED =  1;
/** waitStatus value to indicate successor's thread needs unparking */
static final int SIGNAL    = -1;
/** waitStatus value to indicate thread is waiting on condition */
static final int CONDITION = -2;
/**
 * waitStatus value to indicate the next acquireShared should
 * unconditionally propagate
 */
static final int PROPAGATE = -3;

private static boolean shouldParkAfterFailedAcquire(Node pred, Node node) {
    // 获取前一个节点的waitStatus
    int ws = pred.waitStatus;
    // 如果waitStatus为-1，则可以挂起
    if (ws == Node.SIGNAL)
        /*
         * This node has already set status asking a release
         * to signal it, so it can safely park.
         */
        return true;
    // 如果waitStatus为1，代表CANCELLED，表示上一个节点要跑路了
    if (ws > 0) {
        /*
         * Predecessor was cancelled. Skip over predecessors and
         * indicate retry.
         */
        // 循环向之前的节点遍历，找到waitStatus不为1的
        do {
            node.prev = pred = pred.prev;
        } while (pred.waitStatus > 0);
        // 将此节点作为当前节点的prev节点
        pred.next = node;
    } else {
        /*
         * waitStatus must be 0 or PROPAGATE.  Indicate that we
         * need a signal, but don't park yet.  Caller will need to
         * retry to make sure it cannot acquire before parking.
         */
        // 否则CAS将prev节点的waitStatus置为-1
        compareAndSetWaitStatus(pred, ws, Node.SIGNAL);
    }
    return false;
}
```

