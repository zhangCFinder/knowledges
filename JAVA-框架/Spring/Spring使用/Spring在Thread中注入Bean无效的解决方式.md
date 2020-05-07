[TOC]
> 在Spring项目中，有时需要新开线程完成一些复杂任务，而线程中可能需要注入一些服务。而通过Spring注入来管理和使用服务是较为合理的方式。但是若直接在Thread子类中通过注解方式注入Bean是无效的。

> 因为Spring本身默认Bean为单例模式构建，同时是非线程安全的，因此禁止了在Thread子类中的注入行为，因此在Thread中直接注入的bean是null的，会发生空指针错误。


以下分别列举错误的注入方法和两种解决方式。

### 错误的注入方法
```java
@Controller
public class SomeController{
    @ResponseBody
    @RequestMapping("test")
    String testInjection(){
        // 直接创建并运行线程
        new SomeThread().start();
    }
}
// 直接编写线程
public SomeThread extends Thread {
    @Autowired
    SomeService someService;
    @Override
    public void run(){
        // do something...
        someService.doSomething();
        // 此时 someService实例是null.
    }
}
```

报NullpointException。

### 通过封装Thread子类注入

个人比较推荐这种方法，对外部代码的影响较小。
```java
@Controller
public class SomeController{
    // 通过注解注入封装线程的Bean
    @AutoWired
    SomeThread someThread;
    @ResponseBody
    @RequestMapping("test")
    String testInjection(){
        // 通过注入的Bean启动线程
        someThread.execute();
    }
}

@Component
public class SomeThread {
    // 封装Bean中注入服务
    @AutoWired
    SomeService someService
    public void execute() {
        new Worker().start();
    }
    // 线程内部类，Thread或者Runnable均可
    private class Worker extends Thread {
        @Override
        public void run() {
            // do something...
            SomeThread.this.someService.doSomething();
            // 此时someService已被注入，非null.
        }
    }
}

```

正常调用someService。

### 通过外部引入

即在可以注入的地方先得到可用的实例，在通过Thread子类的构造函数引入。这样会使得在进行代码修改时，影响到每个使用Thread子类的代码，修改工作量大。
```java
@Controller
public class SomeController{
    // 通过注解注入Service
    @AutoWired
    SomeService someService;
    @ResponseBody
    @RequestMapping("test")
    String testInjection(){
        // 通过构造函数从外部引入
        new SomeThread(someService).start();
    }
}

public class SomeThread {
    private SomeService someService;
    public SomeThread(SomeService someService){
        // 通过构造函数从外部引入
        this.someService  = someService;
    }
    @Override
    public void run() {
        // do something...
        someService.doSomething();
        // 此时someService非null.
    }
}
```
