## 1.阻塞队列

当队列是空，从队列获取元素的操作被阻塞，当队列满，从队列放元素的操作被阻塞。
BlockingQueue是他们的主要接口，主要说这几个阻塞队列：

```java
    public static void main(String[] args) throws InterruptedException{
        /**
         * 第一组
         * add() remove()，如果add()超过容量就报错，remove也是。
         */
        BlockingQueue<String> queue = new ArrayBlockingQueue<>(3);
        queue.add("123");
        queue.remove("123");

        /**
         * 第二组
         * offer(),poll(),如果超过容量或者不够，就offer返回false，poll返回null，比较友好
         */

        /**
         * 第三组
         * put(),如果容量不够，put不进来，就阻塞等着.
         * take()会出队，然后put才能进来。
         */
        queue.put("123");
        queue.put("234");
        queue.put("873");
        System.out.println("------------");
        queue.take();
        queue.put("0912");

        /**
         * 第四组
         * offer(),下面这个offer就是如果时间超过我设置的，不等了不阻塞，直接放弃。
         */
        queue.offer("a",2L, TimeUnit.SECONDS);
    }
```

## 2.线程池

频繁的创建和销毁线程本来就会造成系统资源的浪费，在服务器负载过大的时候，如何让新线程等待或者友好的拒绝服务也是需要做到的。所以需要线程池，它可以管理和复用线程池，实现任务线程队列缓存策略和拒绝机制。

### 2.1 线程池如何使用

```java
 ExecutorService threadPoll = new ThreadPoolExecutor(2,5,1L, TimeUnit.SECONDS,
               new LinkedBlockingQueue<>(), Executors.defaultThreadFactory(),new ThreadPoolExecutor.AbortPolicy());
```

自己配参数。因为java提供的一些线程池，比如

newCachedThreadPool，最大线程数达到Integer的最大，显然不合理，有oom风险。

newScheduledThreadPoolExecutor，最大线程数也达到Integer的最大，显然不合理，有oom风险。

newSingleThreadPoll，阻塞队列的能接受的任务达到Integer的最大，显然不合理，有oom风险。

newFixedThreadPool，阻塞队列的能接受的任务达到Integer的最大，显然不合理，有oom风险。

反正他给提供的那些都不合理，都最好不要用。

### 2.2 线程池的重要参数

corePoolSize:核心线程数量，永远都保留的活跃线程数量。过大就资源浪费，过小要频繁的创建销毁线程。

maximunPoolSize：能容纳的最大线程数量。

keepAliveTime：表示线程池中的线程空闲时间，当空闲时间达到keppAliveTime，线程会被销毁，直到corePoolSize个线程为止。

workQueue：缓存队列，当请求线程数量>corePoolSize时，线程进入BlockingQ，使用的LinkedBlockingQ是一个单向链表，是生产消费队列模型。

theadFactory：表示线程工厂，他用来生成一组相同任务的线程。

handler：是拒绝策略，当缓存队列满，并且活动线程数大于maximunPoolSize的时候，线程就要执行这个拒绝策略了。比较友好的拒绝策略应该是这样的：保存到数据库进行削峰填谷，空闲了时候再拿出来执行；转向某个提示页面；打印日志。


线程池源码中的几种状态：他是用一个Integer表示，总共32位，前3位表示状态，后29位表示工作线程数。

111 Running
000 shutdown：不再接受新任务，但已经接受的任务可以执行
001 stop：全面拒绝，停止正在执行的线程
010 tidying：所有任务已经被终止
011 terminated：已清理完现场。

### 2.3 底层工作原理

拒绝策略，jdk默认有4种，默认就是抛异常。

1.抛出异常

2.CallerRunsPolicy：回退给调用者，不一定哪个就被回退了。

3.DiscardOldestPolicy：抛弃等待最久的，再把当前的任务再次提交。

4.DiscardPolicy：随机丢。

### 2.4 合理配置线程数量如何考虑？

首先看你多少核，cpu密集型，一般是cpu核数+1，io密集的话，瓶颈在io，大部分线程都阻在io上，可以配个cpu核数/1-阻塞系数，阻塞系数一般是0.8到0.9。

