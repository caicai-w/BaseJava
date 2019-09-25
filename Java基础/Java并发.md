### 1.为什么使用线程池

频繁的创建和销毁线程本来就会造成系统资源的浪费，在服务器负载过大的时候，如何让新线程等待或者友好的拒绝服务也是需要做到的。所以需要线程池，它可以管理和复用线程池，实现任务线程队列缓存策略和拒绝机制。

### 2.线程池如何使用

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

### 3.线程池的重要参数

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

### 4.底层工作原理

拒绝策略，jdk默认有4种，默认就是抛异常。

1.抛出异常

2.CallerRunsPolicy：回退给调用者，不一定哪个就被回退了。

3.DiscardOldestPolicy：抛弃等待最久的，再把当前的任务再次提交。

4.DiscardPolicy：随机丢。

### 5.合理配置线程数量如何考虑？

首先看你多少核，cpu密集型，一般是cpu核数+1，io密集的话，瓶颈在io，大部分线程都阻在io上，可以配个cpu核数/1-阻塞系数，阻塞系数一般是0.8到0.9。

