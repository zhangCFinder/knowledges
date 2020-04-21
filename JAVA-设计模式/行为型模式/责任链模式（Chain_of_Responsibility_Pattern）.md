# 一. 定义

责任链模式也叫职责链模式。为了避免请求发送者与多个请求处理者耦合在一起，将所有请求的处理者通过前一对象记住其下一个对象的引用而连成一条链；当有请求发生时，可将请求沿着这条链传递，直到有对象处理它为止。

# 二. 优缺点
## 优点：
1. 降低耦合度。它将请求的发送者和接收者解耦。

2. 增强给对象指派职责的灵活性。通过改变链内的成员或者调动它们的次序，允许动态地新增或者删除责任。
## 缺点：
1. 责任链太长或者每条链判断处理的时间太长会影响性能。特别是递归循环的时候

2. 职责链建立的合理性要靠客户端来保证，增加了客户端的复杂性，可能会由于职责链的错误设置而导致系统出错，如可能会造成循环调用。

3. 不能保证每个请求一定被处理。由于一个请求没有明确的接收者，所以不能保证它一定会被处理，该请求可能一直传到链的末端都得不到处理。

# 三. 使用场景
1. 有多个对象可以处理一个请求，哪个对象处理该请求由运行时刻自动确定。

2. 可动态指定一组对象处理请求，或添加新的处理者。

3. 在不明确指定请求处理者的情况下，向多个处理者中的一个提交请求。
# 四. 实例
职责链模式主要包含以下角色：

* **抽象处理者（Handler）角色**： 定义一个处理请求的接口，包含抽象处理方法和一个后继连接。

* **具体处理者（Concrete Handler）角色**： 实现抽象处理者的处理方法，判断能否处理本次请求，如果可以处理请求则处理，否则将该请求转给它的后继者。

* **客户类（Client）角色**： 创建处理链，并向链头的具体处理者对象提交请求，它不关心处理细节和请求的传递过程。
## 示例：模拟程序员提交功能给测试人员，测试人员觉得没问题之后提交给技术总监，技术总监都觉得没问题后才会通知老板，老板满意之后就可以上线了。
![b16c0341bad204493dc2aa0b0a1fb283](责任链模式（Chain_of_Responsibility_Pattern）.resources/1CA13170-6F4B-4F06-99F7-67133EB23536.png)

ReviewPerson（抽象处理者角色）：
```java
public abstract class ReviewPerson {
    protected ReviewPerson person;

    abstract void handle(String program);

    public ReviewPerson getPerson() {
        return person;
    }

    public void setPerson(ReviewPerson person) {
        this.person = person;
    }
}
```

Tester、CTO、Boss（具体处理者角色）：
```java
public class Tester extends ReviewPerson{
    private final String NAME = "测试人员";
    @Override
    void handle(String program) {
        if("没有Bug的功能！".equals(program)){
            System.out.println(NAME + "：没问题，提交给技术总监...");
            getPerson().handle(program);
        }else {
            System.out.println(NAME + "：有Bug呀，再改改！");
        }
    }
}
public class CTO extends ReviewPerson{
    private final String NAME = "技术总监";
    @Override
    void handle(String program) {
        if("没有Bug的功能！".equals(program)){
            System.out.println(NAME + "：没问题，提交给老板...");
            getPerson().handle(program);
        }else {
            System.out.println(NAME + "：有Bug呀，再改改！");
        }
    }
}
public class Boss extends ReviewPerson{

    private final String NAME = "老板";

    @Override
    void handle(String program) {
        if("没有Bug的功能！".equals(program)){
            System.out.println(NAME + "：功能完成，可以上线了！");
        }else {
            System.out.println(NAME + "：有Bug呀，再改改！");
        }
    }
}
```

Programmer（客户类角色）：
```java
public class Programmer {
    public static void main(String[] args) {
        ReviewPerson tester = new Tester();
        ReviewPerson cto = new CTO();
        ReviewPerson boss = new Boss();

        tester.setPerson(cto);
        cto.setPerson(boss);

        tester.handle("没有Bug的功能！");
    }
}
```

输出结果：
```shell
测试人员：没问题，提交给技术总监...
技术总监：没问题，提交给老板...
老板：功能完成，可以上线了！
```
# 五. 总结
我认为责任链模式的好处在于客户端不需要知道处理请求的内部实现，而是交给处理者内部相互之间的调用。不过需要注意的是职责链不能过长，否则可能会导致性能下降，并且还需要保证请求一定会被处理，不要出现像“踢皮球”一样，踢着踢着就没下文了。

