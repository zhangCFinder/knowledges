
# 一、ThreadPoolExecutor

```java

public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)
```
每次提交任务时，如果线程数还没达到核心线程数 `corePoolSize`，线程池就创建新线程来执行。当线程数达到 `corePoolSize` 后，

新增的任务就放到工作队列 `workQueue` 里，而线程池中的线程则努力地从 workQueue 里拉活来干，也就是调用 poll 方法来获取任务。

如果任务很多，并且 workQueue 是个有界队列，队列可能会满，此时线程池就会紧急创建新的临时线程来救场，如果总的线程数达到了最大线程数 `maximumPoolSize`，则不能再创建新的临时线程了，转而执行拒绝策略 handler，比如抛出异常或者由调用者线程来执行任务等。

如果高峰过去了，线程池比较闲了怎么办？临时线程使用 poll（keepAliveTime, unit）方法从工作队列中拉活干，请注意 poll 方法设置了超时时间，如果超时了仍然两手空空没拉到活，表明它太闲了，这个线程会被销毁回收。

通过threadFactory可以扩展原生的线程工厂，比如给创建出来的线程取个有意义的名字。

# 二、FixedThreadPool/CachedThreadPool

```java

public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                 new LinkedBlockingQueue<Runnable>());
}

public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

从上面的代码你可以看到：

* FixedThreadPool 有固定长度（nThreads）的线程数组，忙不过来时会把任务放到无限长的队列里，这是因为 LinkedBlockingQueue 默认是一个无界队列。

* CachedThreadPool 的 maximumPoolSize 参数值是Integer.MAX_VALUE，因此它对线程个数不做限制，忙不过来时无限创建临时线程，闲下来时再回收。它的任务队列是 SynchronousQueue，表明队列长度为 0。

# 三、定制版的 ThreadPoolExecutor
跟 FixedThreadPool/CachedThreadPool 一样，Tomcat 的线程池也是一个定制版的 ThreadPoolExecutor。

通过比较 FixedThreadPool 和 CachedThreadPool，我们发现它们传给 ThreadPoolExecutor 的参数有两个关键点：

1. 是否限制线程个数。

2. 是否限制队列长度。

对于 Tomcat 来说，这两个资源都需要限制，也就是说要对高并发进行控制，否则 CPU 和内存有资源耗尽的风险。因此 Tomcat 传入的参数是这样的：

```java

//定制版的任务队列
taskqueue = new TaskQueue(maxQueueSize);

//定制版的线程工厂
TaskThreadFactory tf = new TaskThreadFactory(namePrefix,daemon,getThreadPriority());

//定制版的线程池
executor = new ThreadPoolExecutor(getMinSpareThreads(), getMaxThreads(), maxIdleTime, TimeUnit.MILLISECONDS,taskqueue, tf);
```
* Tomcat 有自己的定制版任务队列和线程工厂，并且可以限制任务队列的长度，它的最大长度是 maxQueueSize。

* Tomcat 对线程数也有限制，设置了核心线程数（minSpareThreads）和最大线程池数（maxThreads）。

除了资源限制以外，Tomcat 线程池还定制自己的任务处理流程。我们知道 **Java 原生线程池**的任务处理逻辑比较简单：

1. 前 corePoolSize 个任务时，来一个任务就创建一个新线程。

2. 后面再来任务，就把任务添加到任务队列里让所有的线程去抢，如果队列满了就创建临时线程。

3. 如果总线程数达到 maximumPoolSize，执行拒绝策略。

Tomcat 线程池扩展了原生的 ThreadPoolExecutor，通过重写 execute 方法实现了自己的任务处理逻辑：

1. 前 corePoolSize 个任务时，来一个任务就创建一个新线程。

2. 再来任务的话，就把任务添加到任务队列里让所有的线程去抢，如果队列满了就创建临时线程。

3. 如果总线程数达到 maximumPoolSize，**则继续尝试把任务添加到任务队列中去**。（此时是为了试一下看临时线程是不是有任务已经处理完了）

4. **如果缓冲队列也满了，插入失败，执行拒绝策略**。

观察 Tomcat 线程池和 Java 原生线程池的区别，其实就是在第 3 步，Tomcat 在线程总数达到最大数时，不是立即执行拒绝策略，而是再尝试向任务队列添加任务，添加失败后再执行拒绝策略。那具体如何实现呢，其实很简单，我们来看一下 Tomcat 线程池的 execute 方法的核心代码。

```java

public class ThreadPoolExecutor extends java.util.concurrent.ThreadPoolExecutor {
  
  ...
  
  public void execute(Runnable command, long timeout, TimeUnit unit) {
      submittedCount.incrementAndGet();
      try {
          //调用Java原生线程池的execute去执行任务
          super.execute(command);
      } catch (RejectedExecutionException rx) {
         //如果总线程数达到maximumPoolSize，Java原生线程池执行拒绝策略
          if (super.getQueue() instanceof TaskQueue) {
              final TaskQueue queue = (TaskQueue)super.getQueue();
              try {
                  //继续尝试把任务放到任务队列中去
                  if (!queue.force(command, timeout, unit)) {
                      submittedCount.decrementAndGet();
                      //如果缓冲队列也满了，插入失败，执行拒绝策略。
                      throw new RejectedExecutionException("...");
                  }
              } 
          }
      }
}
```

从这个方法你可以看到，Tomcat 线程池的 execute 方法会调用 Java 原生线程池的 execute 去执行任务，如果总线程数达到 maximumPoolSize，Java 原生线程池的 execute 方法会抛出 RejectedExecutionException 异常，但是这个异常会被 Tomcat 线程池的 execute 方法捕获到，并继续尝试把这个任务放到任务队列中去；如果任务队列也满了，再执行拒绝策略。

# 四、定制版的任务队列

在 Tomcat 线程池的 execute 方法最开始有这么一行：
```java
submittedCount.incrementAndGet();
```
这行代码的意思把 submittedCount 这个原子变量加一，并且在任务执行失败，抛出拒绝异常时，将这个原子变量减一：
```java
submittedCount.decrementAndGet();
```
其实 Tomcat 线程池是用这个变量 submittedCount 来维护已经提交到了线程池，但是还没有执行完的任务个数。

> Tomcat 为什么要维护这个变量呢？

这跟 Tomcat 的定制版的任务队列有关。Tomcat 的任务队列 TaskQueue 扩展了 Java 中的 LinkedBlockingQueue，

我们知道 LinkedBlockingQueue 默认情况下长度是没有限制的，除非给它一个 capacity。因此 Tomcat 给了它一个 capacity，

TaskQueue 的构造函数中有个整型的参数 capacity，TaskQueue 将 capacity 传给父类 LinkedBlockingQueue 的构造函数。
```java

public class TaskQueue extends LinkedBlockingQueue<Runnable> {

  public TaskQueue(int capacity) {
      super(capacity);
  }
  ...
}
```

这个 capacity 参数是通过 Tomcat 的 maxQueueSize 参数来设置的，但问题是默认情况下 maxQueueSize 的值是Integer.MAX_VALUE，等于没有限制，这样就带来一个问题：当前线程数达到核心线程数之后，再来任务的话线程池会把任务添加到任务队列，并且总是会成功，这样永远不会有机会创建新线程了。

为了解决这个问题，TaskQueue 重写了 LinkedBlockingQueue 的 offer 方法，在合适的时机返回 false，返回 false 表示任务添加失败，这时线程池会创建新的线程。那什么是合适的时机呢？请看下面 offer 方法的核心源码：

```java

public class TaskQueue extends LinkedBlockingQueue<Runnable> {

  ...
   @Override
  //线程池调用任务队列的方法时，当前线程数肯定已经大于核心线程数了
  public boolean offer(Runnable o) {

      //如果线程数已经到了最大值，不能创建新线程了，只能把任务添加到任务队列。
      if (parent.getPoolSize() == parent.getMaximumPoolSize()) 
          return super.offer(o);
          
      //执行到这里，表明当前线程数大于核心线程数，并且小于最大线程数。
      //表明是可以创建新线程的，那到底要不要创建呢？分两种情况：
      
      //1. 如果已提交的任务数小于当前线程数，表示还有空闲线程，无需创建新线程
      if (parent.getSubmittedCount()<=(parent.getPoolSize())) 
          return super.offer(o);
          
      //2. 如果已提交的任务数大于当前线程数，线程不够用了，返回false去创建新线程
      if (parent.getPoolSize()<parent.getMaximumPoolSize()) 
          return false;
          
      //默认情况下总是把任务添加到任务队列
      return super.offer(o);
  }
  
}
```

从上面的代码我们看到，只有当前线程数大于核心线程数、小于最大线程数，并且已提交的任务个数大于当前线程数时，也就是说线程不够用了，但是线程数又没达到极限，才会去创建新的线程。

这就是为什么 Tomcat 需要维护已提交任务数这个变量，它的目的就是在任务队列的长度无限制的情况下，让线程池有机会创建新的线程。

当然默认情况下 Tomcat 的任务队列是没有限制的，你可以通过设置 maxQueueSize 参数来限制任务队列的长度。