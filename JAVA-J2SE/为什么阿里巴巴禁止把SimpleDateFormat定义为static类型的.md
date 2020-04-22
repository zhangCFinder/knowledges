[TOC]
![ea9c3173924c84d571b470335a3bd082](为什么阿里巴巴禁止把SimpleDateFormat定义为static类型的.resources/B3268460-6402-4A72-872E-4525D409FD56.png)

## SimpleDateFormat用法

SimpleDateFormat是Java提供的一个格式化和解析日期的工具类。它允许进行格式化（日期 -> 文本）、解析（文本 -> 日期）和规范化。SimpleDateFormat 使得可以选择任何用户定义的日期-时间格式的模式。

在Java中，可以使用SimpleDateFormat的format方法，将一个Date类型转化成String类型，并且可以指定输出格式。

```java
// Date转String
Date data = new Date();
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
String dataStr = sdf.format(data);
System.out.println(dataStr);
```

以上代码，转换的结果是：2018-11-25 13:00:00，日期和时间格式由"日期和时间模式"字符串指定。如果你想要转换成其他格式，只要指定不同的时间模式就行了。


在Java中，可以使用SimpleDateFormat的parse方法，将一个String类型转化成Date类型。

```java
// String转Data
System.out.println(sdf.parse(dataStr));
```
## 日期和时间模式表达方法
在使用SimpleDateFormat的时候，需要通过字母来描述时间元素，并组装成想要的日期和时间模式。常用的时间元素和字母的对应表如下：
![ce7ca7c258a6fda53d702a7c062a5a0d](为什么阿里巴巴禁止把SimpleDateFormat定义为static类型的.resources/7F9D7634-E493-4D48-9DC4-C209B5B99F46.png)

模式字母通常是重复的，其数量确定其精确表示。如下表是常用的输出格式的表示方法。
![f0a716225feeb4afef6f4006e47e3c71](为什么阿里巴巴禁止把SimpleDateFormat定义为static类型的.resources/72D83895-C5D6-47AF-90A5-24F12D866EF7.png)

## 输出不同时区的时间
```java
SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
sdf.setTimeZone(TimeZone.getTimeZone("America/Los_Angeles"));
System.out.println(sdf.format(Calendar.getInstance().getTime()));
```
## SimpleDateFormat线程安全性

由于SimpleDateFormat比较常用，而且在一般情况下，一个应用中的时间显示模式都是一样的，所以很多人愿意使用如下方式定义SimpleDateFormat：
```java
public class Main {

    private static SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    public static void main(String[] args) {
        simpleDateFormat.setTimeZone(TimeZone.getTimeZone("America/New_York"));
        System.out.println(simpleDateFormat.format(Calendar.getInstance().getTime()));
    }
}

```

**这种定义方式，存在很大的安全隐患。**

### 问题重现
```java
   /** * @author Hollis */ 
   public class Main {

    /**
     * 定义一个全局的SimpleDateFormat
     */
    private static SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");

    /**
     * 使用ThreadFactoryBuilder定义一个线程池
     */
    private static ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
        .setNameFormat("demo-pool-%d").build();

    private static ExecutorService pool = new ThreadPoolExecutor(5, 200,
        0L, TimeUnit.MILLISECONDS,
        new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory, new ThreadPoolExecutor.AbortPolicy());

    /**
     * 定义一个CountDownLatch，保证所有子线程执行完之后主线程再执行
     */
    private static CountDownLatch countDownLatch = new CountDownLatch(100);

    public static void main(String[] args) {
        //定义一个线程安全的HashSet
        Set<String> dates = Collections.synchronizedSet(new HashSet<String>());
        for (int i = 0; i < 100; i++) {
            //获取当前时间
            Calendar calendar = Calendar.getInstance();
            int finalI = i;
            pool.execute(() -> {
                    //时间增加
                    calendar.add(Calendar.DATE, finalI);
                    //通过simpleDateFormat把时间转换成字符串
                    String dateString = simpleDateFormat.format(calendar.getTime());
                    //把字符串放入Set中
                    dates.add(dateString);
                    //countDown
                    countDownLatch.countDown();
            });
        }
        //阻塞，直到countDown数量为0
        countDownLatch.await();
        //输出去重后的时间个数
        System.out.println(dates.size());
    }
}

```

以上代码，其实比较简单，很容易理解。就是循环一百次，每次循环的时候都在当前时间基础上增加一个天数（这个天数随着循环次数而变化），然后把所有日期放入一个**线程安全的、带有去重功能的Set中，然后输出Set中元素个数**。


正常情况下，以上代码输出结果应该是100。但是实际执行结果是一个小于100的数字。

**原因就是因为SimpleDateFormat作为一个非线程安全的类，被当做了共享变量在多个线程中进行使用，这就出现了线程安全问题。**

### 线程不安全原因

通过以上代码，我们发现了在并发场景中使用SimpleDateFormat会有线程安全问题。其实，JDK文档中已经明确表明了SimpleDateFormat不应该用在多线程场景中：
> Date formats are not synchronized. It is recommended to create separate format instances for each thread. If multiple threads access a format concurrently, it must be synchronized externally.


那么接下来分析下为什么会出现这种问题，SimpleDateFormat底层到底是怎么实现的？我们跟一下SimpleDateFormat类中format方法的实现其实就能发现端倪。
![7993e166ec1bbbec680e6c620aa94e77](为什么阿里巴巴禁止把SimpleDateFormat定义为static类型的.resources/C1EFD828-22BB-44B5-AF1D-C580A6D76714.png)


SimpleDateFormat中的format方法在执行过程中，会使用一个成员变量calendar来保存时间。这其实就是问题的关键。

由于我们在声明SimpleDateFormat的时候，使用的是static定义的。那么这个SimpleDateFormat就是一个共享变量，随之，SimpleDateFormat中的calendar也就可以被多个线程访问到。

假设线程1刚刚执行完calendar.setTime把时间设置成2018-11-11，还没等执行完，线程2又执行了calendar.setTime把时间改成了2018-12-12。这时候线程1继续往下执行，拿到的calendar.getTime得到的时间就是线程2改过之后的。

除了format方法以外，SimpleDateFormat的parse方法也有同样的问题。

所以，不要把SimpleDateFormat作为一个共享变量使用。

### 如何解决

前面介绍过了SimpleDateFormat存在的问题以及问题存在的原因，那么有什么办法解决这种问题呢？

解决方法有很多，这里介绍三个比较常用的方法。

#### 解决方案一: 使用局部变量
```java
for (int i = 0; i < 100; i++) {
    //获取当前时间
    Calendar calendar = Calendar.getInstance();
    int finalI = i;
    pool.execute(() -> {
        // SimpleDateFormat声明成局部变量
    SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        //时间增加
        calendar.add(Calendar.DATE, finalI);
        //通过simpleDateFormat把时间转换成字符串
        String dateString = simpleDateFormat.format(calendar.getTime());
        //把字符串放入Set中
        dates.add(dateString);
        //countDown
        countDownLatch.countDown();
    });
}
```

SimpleDateFormat变成了局部变量，就不会被多个线程同时访问到了，就避免了线程安全问题。

#### 解决方案二: 加同步锁

```java
for (int i = 0; i < 100; i++) {
    //获取当前时间
    Calendar calendar = Calendar.getInstance();
    int finalI = i;
    pool.execute(() -> {
        //加锁
        synchronized (simpleDateFormat) {
            //时间增加
            calendar.add(Calendar.DATE, finalI);
            //通过simpleDateFormat把时间转换成字符串
            String dateString = simpleDateFormat.format(calendar.getTime());
            //把字符串放入Set中
            dates.add(dateString);
            //countDown
            countDownLatch.countDown();
        }
    });
}
```

通过加锁，使多个线程排队顺序执行。避免了并发导致的线程安全问题。其实以上代码还有可以改进的地方，就是可以把锁的粒度再设置的小一点，可以只对`simpleDateFormat.format`这一行加锁，这样效率更高一些。

#### 解决方案三: 使用ThreadLocal

第三种方式，就是使用 ThreadLocal。 ThreadLocal 可以确保每个线程都可以得到单独的一个 SimpleDateFormat 的对象，那么自然也就不存在竞争问题了。
```java
/**
 * 使用ThreadLocal定义一个全局的SimpleDateFormat
 */
private static ThreadLocal<SimpleDateFormat> simpleDateFormatThreadLocal = new ThreadLocal<SimpleDateFormat>() {
    @Override
    protected SimpleDateFormat initialValue() {
        return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    }
};

//用法
String dateString = simpleDateFormatThreadLocal.get().format(calendar.getTime());
```

用 ThreadLocal 来实现其实是有点类似于缓存的思路，每个线程都有一个独享的对象，避免了频繁创建对象，也避免了多线程的竞争。当然，以上代码也有改进空间，就是，其实SimpleDateFormat的创建过程可以改为延迟加载。这里就不详细介绍了。


#### 解决方案四: 使用DateTimeFormatter

如果是Java8应用，可以使用DateTimeFormatter代替SimpleDateFormat，这是一个线程安全的格式化工具类。就像官方文档中说的，这个类` simple beautiful strong immutable thread-safe`。
```java
//解析日期
String dateStr= "2016年10月25日";
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy年MM月dd日");
LocalDate date= LocalDate.parse(dateStr, formatter);

//日期转换为字符串
LocalDateTime now = LocalDateTime.now();
DateTimeFormatter format = DateTimeFormatter.ofPattern("yyyy年MM月dd日 hh:mm a");
String nowStr = now .format(format);
System.out.println(nowStr);

```