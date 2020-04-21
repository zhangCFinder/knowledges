
# 1. 简介
从这里开始，我们在介绍设计模式之前，需要强调一下角色。实际上从上篇工厂模式中，就已经有角色的概念。

大家想想，工厂模式中有哪些角色？没错，就是工厂角色和产品角色。

同样，在观察者模式中，也有两个角色，就是观察者和被观察者。

在理解设计模式的时候，首先要有个概念，就是每个角色都对应这一个类，比如观察者模式，观察者肯定对应着一个观察者类，被观察者肯定对应的被观察者类。

那么设计模式实际上就是通过面向对象的特性，将这些角色解耦。

观察者模式本质上就是一种**订阅/发布**的模型，从逻辑上来说就是**一对多**的依赖关系。什么意思呢？好比是一群守卫盯着一个囚犯，只要囚犯一有异动，守卫就必须马上采取行动（也有可能是更新状态，本质上也是一种行动），那么守卫就是观察者，囚犯就是被观察者。

在一个系统中，实现这种一对多的而且之间有一定关联的逻辑的时候，由于需要保持他们之间的协同关系，所以最简便的方法是采用紧耦合，

把这些对象绑定到一起。但是这样一来，一旦有扩展或者修改的时候，开发人员所面对的难度非常大，而且很容易造成Bug。

那么观察者模式就解决了这么一个问题，在保持一系列观察者和被观察者对象协同工作的同时，把之间解耦了。

# 2. 实例

## //被观察者

```java
public interface IObject {

    void addMonitor(IMonitor iMonitor);

    void removeMonitor(IMonitor iMonitor);

    void notifyMonitor(String state);
}

public class Subject implements IObject {
    List<IMonitor> iMonitorList = new ArrayList<IMonitor>();



    @Override
    public void addMonitor(IMonitor iMonitor) {
        iMonitorList.add(iMonitor);
    }

    @Override
    public void removeMonitor(IMonitor iMonitor) {

        iMonitorList.remove(iMonitor);
    }

    @Override
    public void notifyMonitor(String state) {

        for (IMonitor iMonitor : iMonitorList) {
            iMonitor.update(state);
        }
    }
}

```

## //观察者
```java
public interface IMonitor {

    public void update(String state);

}

public class Monitor implements IMonitor {
    private String monitorState;
    private String  name;
    private IObject subject;

    public Monitor(String name, IObject subject,String  monitorState) {
        this.name = name;
        this.subject = subject;
        System.out.println("我是观察者 " + name + "，我的初始状态为" + monitorState );
    }

    @Override
    public void update(String subjectState) {
        System.out.println("我是观察者 " + name + "，我的更新状态为" + subjectState);
    }
}

```

## Test

```java
public class Test {
    public static void main(String[] args) {
        IObject subject = new Subject();
        String SubjectInitState = "Stop!";
        subject.addMonitor(new Monitor("Monitor1",subject,SubjectInitState));
        subject.addMonitor(new Monitor("Monitor2",subject,SubjectInitState));
        subject.addMonitor(new Monitor("Monitor3",subject,SubjectInitState));

        subject.notifyMonitor("start!");

    }
```

结果如下:
```shell
我是观察者Monitor_1，我的初始状态是Stop!
我是观察者Monitor_2，我的初始状态是Stop!
我是观察者Monitor_3，我的初始状态是Stop!
我是观察者Monitor_1，我的状态是Start!
我是观察者Monitor_2，我的状态是Start!
我是观察者Monitor_3，我的状态是Start!
```

1. 在被观察者中，我定义了一个集合用来存放观察者，并且写了一个Add方法一个Remove方法来添加和移除观察者，这体现了一对多的关系，也提供了可以控制观察者的方式。所以，**我们得到第一个关键点：每个观察者需要被保存到被观察者的集合中，并且给被观察者提供添加和删除的方式。**

2. 在添加一个观察者的时候，把被观察者对象以构造函数的形式给传入了观察者。最后我让被观察者执行notifyMonitor方法，这时会触法所有观察着的update方法以更新状态。**所以我们得到第二个关键点，被观察者把自己传给观察者，当状态改变后，通过遍历或循环的方式逐个通知列表中的观察者。**

3. 好了，到这里你应该可以把握住观察者模式的关键了。但这里有个问题，被观察者是通过构造函数参数的形式，传给观察者的，而观察者对象时被Add到被观察者的List中。**所以，我们得到第三个关键点，虽然解耦了观察者和被观察者的依赖，让各自的变化不大影响另一方的变化，但是这种解耦并不是很彻底，没有完全解除两者之间的耦合。**

# 3. 总结

好的，最后我们还是来归纳一下观察者模式的关键点

1. 每个观察者需要被保存到被观察者的集合中，并且为被观察者提供添加和删除的方式。

2. 被观察者把自己传给观察者，当状态改变后，通过遍历或循环的方式逐个通知列表中的观察者。

3. 虽然解耦了观察者和被观察者的依赖，让各自的变化不大影响另一方的变化，但是这种解耦并不是很彻底，没有完全解除两者之间的耦合。

4. 在事件中，订阅者和发布者之间是通过把事件处理程序绑定到委托，并不是把自身传给对方。所以解决了观察者模式中不完全解耦的问题

