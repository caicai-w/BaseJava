- 1.说说JUC包你都知道多少？
- 2.说说AQS
- 3.JUC中的锁，ReentrantLock
- 4.说说Atomic 原子类
- ConcurrentHashMap
- ConcurrentHashMap底层实现
- CopyOnWriteArrayList

## 1.说说JUC包你都知道多少？

并发包主要分为以下几个类族：
1.线程同步类。这些类使线程间的协调更加容易，逐步淘汰了使用object的wait(),和notify()进行同步的方式，主要代表为CountDownLatch，Semaphore，CyclicBarrier等。这些类在另一篇中有解释。
2.并发集合类。集合并发操作的要求是执行速度快、提取数据准。最有名的就是ConcurrentHashMap，它不断的优化。由刚开始的锁分段到后来的CAS，性能不断提升，除此之外，还有CopyOnWriteArrayList，BlockingQueue等。
3.线程管理类。根据实际场景的需要，提供了很多创建线程池的快捷方式。比如使用Executors静态工厂或者使用ThreadPoolExecutor等。
4.锁相关的类。锁以Lock接口为核心，派生出在一些实际场景中进行互斥操作的锁相关类。最有名的就是ReentrantLock。
java中常用的锁实现的方式有两种，一种是用并发包的，一种是同步代码块。
Lock是juc包的顶层接口，它的实现逻辑没有用到syn，而是用到volatile的可见性。

## 2.说说AQS

AQS的全称为（AbstractQueuedSynchronizer）
我们常见的锁都是基于它实现的，其实AQS是一个用来构建锁和同步器的框架.
内部实现的关键是：FIFO队列和state状态。

### 2.1.AQS核心思想

如果共享资源空闲，此时来了一个请求，就把这个线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要把这个线程线程阻塞，然后放到一个队列排队，AQS是用CLH队列锁实现的，将暂时获取不到锁的线程加入到队列中。
1.AQS使用 volatile int state来表示同步状态，使用CAS对该同步状态进行原子操作实现对其值的修改。

```
private volatile int state;
共享变量，使用volatile修饰保证线程可见性
```

状态信息通过protected类型的getState，setState，compareAndSetState进行操作。
2.CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配。

**AQS定义两种资源共享方式**：
Exclusive（独占）：只有一个线程能执行，如ReentrantLock。又可分为公平锁和非公平锁：
    公平锁：按照线程在队列中的排队顺序，先到者先拿到锁
    非公平锁：当线程要获取锁时，无视队列顺序直接去抢锁，谁抢到就是谁的
Share（共享）：多个线程可同时执行，如Semaphore/CountDownLatch。Semaphore、CountDownLatch、 CyclicBarrier、ReadWriteLock 我们都会在后面讲到。
ReentrantReadWriteLock 可以看成是组合式，因为ReentrantReadWriteLock也就是读写锁允许多个线程同时对某一资源进行读。

### 2.2AQS底层使用了模板方法模式

如果需要自定义同步器一般的方式是这样（模板方法模式很经典的一个应用）：

使用者继承AbstractQueuedSynchronizer并重写指定的方法。（这些重写方法很简单，无非是对于共享资源state的获取和释放）
将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。
AQS使用了模板方法模式，自定义同步器时需要重写下面几个AQS提供的模板方法：

```
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

以ReentrantLock为例，state初始化为0，表示未锁定状态。A线程lock()时，会调用tryAcquire()独占该锁并将state+1。此后，其他线程再tryAcquire()时就会失败，直到A线程unlock()到state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。

再以CountDownLatch以例，任务分为N个子线程去执行，state也初始化为N（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后countDown()一次，state会CAS(Compare and Swap)减1。等到所有子线程都执行完后(即state=0)，会unpark()主调用线程，然后主调用线程就会从await()函数返回，继续后余动作。

### 2.3. AQS 组件总结

- **Semaphore(信号量)-允许多个线程同时访问：** synchronized 和 ReentrantLock 都是一次只允许一个线程访问某个资源，Semaphore可以指定多个线程同时访问某个资源。
- **CountDownLatch （倒计时器）：**这个工具通常用来控制线程等待，它可以让某一个线程等待直到倒计时结束，再开始执行。
- **CyclicBarrier(循环栅栏)：** 比如我初始化一个Cyc类参数是5，那么每await()一次，就+1，当到5的时候，所有await阻塞的线程会一起被唤醒。初始化参数可以循环使用。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现tryAcquire-tryRelease、tryAcquireShared-tryReleaseShared中的一种即可。但AQS也支持自定义同步器同时实现独占和共享两种方式，如ReentrantReadWriteLock。
使用AQS能简单且高效地构造出应用广泛的的同步器，比如我们提到的ReentrantLock，Semaphore，其他的诸如ReentrantReadWriteLock，SynchronousQueue，FutureTask等等皆是基于AQS的.

### 3.ReentrantLock

```
ReentrantLock实现就两种：公平、非公平，两种实现都继承的Sync。
 public ReentrantLock() {
        sync = new NonfairSync();
    }
  public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }
```

NonfairSync和FairSync都是继承的Sync，Sync继承的AQS，除了这些ReentrantLock就没啥东西了。
先说Sync,下面是源码

```
 abstract static class Sync extends AbstractQueuedSynchronizer {
        private static final long serialVersionUID = -5179523762034025860L;

        /**
         * Performs {@link Lock#lock}. The main reason for subclassing
         * is to allow fast path for nonfair version.
         */
       这个抽象方法在Fair和Nonfair有不同的实现。
        abstract void lock();

        /**
         * Performs non-fair tryLock.  tryAcquire is implemented in
         * subclasses, but both need nonfair try for trylock method.
         */
        非公平尝试获取锁的方法，公平的直接写在FairSync。
        如果state=0，直接拿到锁，CAS修改state，并开始排他。else if表示可重入的情况。
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
       释放锁，如果占有锁的线程根本不是当前来释放的线程，throw。如果释放完state是0了，就释放锁。
        protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }
       判断当前洗线程是不是锁的持有者
        protected final boolean isHeldExclusively() {
            // While we must in general read state before owner,
            // we don't need to do so to check if current thread is owner
            return getExclusiveOwnerThread() == Thread.currentThread();
        }

        final ConditionObject newCondition() {
            return new ConditionObject();
        }

        // Methods relayed from outer class
        获取当前持有锁的线程
        final Thread getOwner() {
            return getState() == 0 ? null : getExclusiveOwnerThread();
        }

        final int getHoldCount() {
            return isHeldExclusively() ? getState() : 0;
        }

        final boolean isLocked() {
            return getState() != 0;
        }

        /**
         * Reconstitutes the instance from a stream (that is, deserializes it).
         */
        private void readObject(java.io.ObjectInputStream s)
            throws java.io.IOException, ClassNotFoundException {
            s.defaultReadObject();
            setState(0); // reset to unlocked state
        }
    }
```

NonfairSync继承Sync实现非公平锁

```
    static final class NonfairSync extends Sync {
        private static final long serialVersionUID = 7316153563782823691L;

        /**
         * Performs lock.  Try immediate barge, backing up to normal
         * acquire on failure.
         */
在lock的时候，先CAS把0设成1，如果设置成功，直接持有成功了，如果CAS没能行，
可能当前state不为0，也可能是CAS有人抢先了，就去执行Sync里面的nonfairTryAcquire().
可以看到nonfairTryAcquire()里面也是直接尝试去获取锁。问题：拿不到就一直去尝试吗？还有，这里的CAS都不会自旋，直接返回true和false。
        final void lock() {
            if (compareAndSetState(0, 1))
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);
        }

        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
    }
```

FairSync 公平锁,

```
     static final class FairSync extends Sync {
        private static final long serialVersionUID = -3000897897090466540L;

        final void lock() {
            acquire(1);
        }

        /**
         * Fair version of tryAcquire.  Don't grant access unless
         * recursive call or no waiters or is first.
         */
获取当前线程，如果当前队列为空，并且能成功CAS，才能去设置当前持有锁的线程是自己，
else if的分支依然做的是可重入的判断。
        protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
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
        }
    }
```

首先lock的 acquire(1)做了什么，

```
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }
```

没明白这个tryAcquire为什么下面没操作了。如果不成功，就去拿队列，把自己放到CLR队列的队尾，在等到的过程中处于休眠的状态，只有拿到了才会返回。
另外hasQueuedPredecessors()做了什么？

```
    public final boolean hasQueuedPredecessors() {
        // The correctness of this depends on head being initialized
        // before tail and on head.next being accurate if the current
        // thread is first in queue.
        Node t = tail; // Read fields in reverse initialization order
        Node h = head;
        Node s;
        //head没有next ----> false
        //head有next，next持有的线程不是当前线程 ----> true
        //head有next，next持有的线程是当前线程 ----> false
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());
    }
这个方法给的解释是，看看当前线程等待的时间是不是最长的，也就是查询是否有其他线程比当前线程等待获取锁花费了更多的时间，有就返回true，没有就返回false，也就是说该方法返回false，才进行addWaiter状态的更改尝试，其余和部分和非公平锁的部分一样。
```

### 4.说说Atomic 原子类

原子变量，由CAS实现，也就是比较并交换，只有当当前值和修改之间记录的值一样时，才会修改，它比的是地址。CAS是原子操作的一种，由cpu指令执行。
actomic包里的类基本都是使用unsafe实现的包装类，unsafe实际上都调用的native方法，实际上执行c代码，调用汇编，cas最终由一条cpu指令cmpxchg，完成操作。毕竟一条cpu指令不会被打断，所以cas是个原子操作。
但是cas有两个问题，一个是ABA，以为没有改过，但其实已经修改又改回来了，比如一个链表，你要删除节点Y，其实节点已经被删掉了，又正巧在Y的地址上新建了一个B，你要删除Y的时候，把新建的B给删了。要解决这个问题，可以用AtomicStampedReference，大概就是它维护了一个pair对象，里面不仅存指向对象的引用，还维护了一个版本号，就不会出现ABA问题了。第二个问题就是大量并发场景下CAS会造成cpu空转，因为一直自旋，java8又出了个LongAdder，它把内部数据分离成一个数据，每个线程访问时，通过hash给映射到不同的数字，最终计算的结果，用数组累计求和，就是有点负载均衡的感觉。



- 1.ConcurrentHashMap
- 2.ConcurrentHashMap和HashTable的区别
- 3.ConcurrentHashMap线程安全的具体实现方式/底层具体实现
- 4.说说CopyOnWriteArrayList

### 5.ConcurrentHashMap

1.7
java5.0在juc包中提供了大量支持并发的容器类，采用“锁分段”机制，Concurrentlevel分段级别，默认16，就是有16个段（segment)，每个段默认又有16个哈希表（table），每个又有链表连着。
![segment.png](https://upload-images.jianshu.io/upload_images/12328609-e52d6cd15a3deb55.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

好处：每个segment都是独立的锁，有多个线程可以并行，多线程访问容器里不同数据段的数据，就不会存在锁竞争。.
1.8
用 Node 数组+链表+红黑树的数据结构来实现，当某个solt内的元素个数增加到超过8个且table的容量大于或等于64时，链表转成红黑树，当某个slot内的元素个数减少到6个时，由红黑树再转成链表。
在转化的过程中使用同步块锁住当前slot的首元素，防止其他进程对当前slot进行增删改操作，转换完成后再利用CAS替换原有链表。
并发控制使用 synchronized 和 CAS 来操作。效率高，不阻塞不涉及上下文切换。

### 6. ConcurrentHashMap线程安全的具体实现方式/底层具体实现

1.7首先将数据分为一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问。
**ConcurrentHashMap 是由 Segment 数组结构和 HashEntry 数组结构组成**。
Segment 实现了 ReentrantLock,所以 Segment 是一种可重入锁，扮演锁的角色。HashEntry 用于存储键值对数据。

```source-java
static class Segment<K,V> extends ReentrantLock implements Serializable {
}
```

一个 ConcurrentHashMap 里包含一个 Segment 数组。Segment 的结构和HashMap类似，是一种数组和链表结构，一个 Segment 包含一个 HashEntry 数组，每个 HashEntry 是一个链表结构的元素，每个 Segment 守护着一个HashEntry数组里的元素，当对 HashEntry 数组的数据进行修改时，必须首先获得对应的 Segment的锁。

JDK1.8ConcurrentHashMap取消了Segment分段锁，采用CAS和synchronized来保证并发安全。数据结构跟HashMap1.8的结构类似，数组+链表/红黑二叉树。Java 8在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为O(N)）转换为红黑树（寻址时间复杂度为O(log(N))）
synchronized只锁定当前链表或红黑二叉树的首节点，这样只要hash不冲突，就不会产生并发，效率又提升N倍。

### 7. 说说CopyOnWriteArrayList

CopyOnWriteArrayList（写入并复制）也是juc里面的，它解决了并发修改异常，每当有写入的时候，就在底层重新复制一个新容器写入，最后把新容器的引用地址赋给旧的容器，在别人写入的时候，其他线程读数据，依然是旧容器的线程。这样是开销很大的，所以不适合频繁写入的操作。适合并发迭代操作多的场景。



