# 一. 定义
给定一个语言，定义它的文法的一种表示，并定义一个解释器，这个解释器使用该表示来解释语言中的句子。
# 二. 优缺点
## 优点：
1. 可扩展性比较好，灵活；

2. 增加新的解释表达式不需改动原有的代码，符合开闭原则；

3. 易于实现简单文法。
## 缺点：
1. 当语法规则数目太多时，增加了系统的复杂度；

2. 解释器模式采用递归调用方法，执行效率低，调试困难；

3. 对于复杂的文法比较难维护；

4. 解释器模式会引起类膨胀。
# 三. 使用场景
某个特定类型问题发生频率足够高。

**解释器模式在实际的系统开发中使用的很少，因为会引起效率、性能以及维护等问题。**
# 四. 实例
**抽象表达式角色(Expression)** ： 声明抽象解释操作，是所有终结符表达式、非终结符表达式的父类。这个接口主要是有一个interpret()方法，称作解释操作。

**终结符表达式角色(Terminal Expression)** ： 实现了抽象表达式角色所要求的接口，主要是一个interpret()方法；文法中的每一个终结符都有一个具体终结表达式与之相对应。

**非终结表达式角色(Nonterminal Expression)** ： 文法中的每一条规则都需要一个具体的非终结符表达式，非终结符表达式一般是文法中的运算符或者其他关键字。

**环境角色(Context)** ： 这个角色的任务一般是用来存放文法中各个终结符所对应的具体值。

## 示例：模拟计算器的加减法
Context：
```java
public class Context {
    private int number1;
    private int number2;

    public Context(int number1, int number2) {
        this.number1 = number1;
        this.number2 = number2;
    }
   
    //为省篇幅，省略get、set方法
}
```
Expression：
```java
public interface Expression {
    int interpret(Context context);
}
```
Plus类、Min类：
```java
public class Plus implements Expression {
    @Override
    public int interpret(Context context) {
        return context.getNumber1() + context.getNumber2();
    }
}
public class Min implements Expression {
    @Override
    public int interpret(Context context) {
        return context.getNumber1() - context.getNumber2();
    }
}
```
Test:测试类
```java
public class Test {
    public static void main(String[] args) {
        int result = new Plus().interpret(new Context(10,new Min().interpret(new Context(30,10))));
        System.out.println("10 + （30-10） = " + result);
    }
}

```
输出结果：
```shell
10 + （30-10） = 30
```
# 五. 总结

增加新的解释表达式较为方便。如果用户需要增加新的解释表达式只需要对应增加一个新的终结符表达式或非终结符表达式类，原有表达式类代码无须修改，符合“开闭原则”。

但学习解释器模式的难度比较大，因为对于复杂解释文法难以维护。每一条规则至少需要定义一个类，如果包含太多文法规则，类的个数将会急剧增加，导致系统难以管理和维护。

