# 一. 定义
在一个系统中如果有多个相同的对象，那么只共享一份就可以了，不必每个都去实例化一个对象。

在`Flyweight`模式中，由于要产生各种各样的对象，所以在`Flyweight`(享元)模式中常出现`Factory`模式。

`Flyweight`的内部状态是用来共享的,`Flyweight factory`负责维护一个对象存储池（`Flyweight Pool`）来存放内部状态的对象。

`Flyweight`模式是一个提高程序效率和性能的模式,会大大加快程序的运行速度.应用场合很多.

# 二. 实例
## 先定义一个抽象的Flyweight类：
```java
package Flyweight;
public abstract class Flyweight{
　public abstract void operation();
}
```
## 实现一个具体类：
```java
package Flyweight;
public class ConcreteFlyweight extends Flyweight{
　private String string;
　public ConcreteFlyweight(String str){
　　string = str;
　}
　public void operation()
　{
　　System.out.println("Concrete---Flyweight : " + string);
　}
}
```
## 实现一个工厂方法类：
```java
package Flyweight;
import java.util.HashMap;
public class FlyweightFactory{
　private HashMap flyweights = new HashMap();//定义了一个HashMap用来存储各个对象
　public FlyweightFactory(){}
　public Flyweight getFlyWeight(Object obj){
　　Flyweight flyweight = (Flyweight) flyweights.get(obj);//选出实例化的对象
　　if(flyweight == null){
　　　//产生新的ConcreteFlyweight
　　　flyweight = new ConcreteFlyweight((String)obj);
　　　flyweights.put(obj, flyweight);
　　}
　　return flyweight;
　}
　public int getFlyweightSize(){
　　return flyweights.size();
　}
}
```

## 最后看看Flyweight的调用：
```java
package Flyweight;
import java.util.Hashtable;
public class FlyweightPattern{
　FlyweightFactory factory = new FlyweightFactory(); 
　Flyweight fly1;
　Flyweight fly2;
　Flyweight fly3;
　Flyweight fly4;
　Flyweight fly5;
　Flyweight fly6;
　/* 构造方法 */
　public FlyweightPattern(){
　　fly1 = factory.getFlyWeight("Google");
　　fly2 = factory.getFlyWeight("Qutr");
　　fly3 = factory.getFlyWeight("Google");
　　fly4 = factory.getFlyWeight("Google");
　　fly5 = factory.getFlyWeight("Google");
　　fly6 = factory.getFlyWeight("Google");
　}
　public void showFlyweight(){
　　fly1.operation();
　　fly2.operation();
　　fly3.operation();
　　fly4.operation();
　　fly5.operation();
　　fly6.operation();
　　int objSize = factory.getFlyweightSize();
　　System.out.println("objSize = " + objSize);
　}
　public static void main(String[] args){
　　System.out.println("The FlyWeight Pattern!");
　　FlyweightPattern fp = new FlyweightPattern();
　　fp.showFlyweight();
　}
}
```
下面是运行结果：
```shell
Concrete---Flyweight : Google
Concrete---Flyweight : Qutr
Concrete---Flyweight : Google
Concrete---Flyweight : Google
Concrete---Flyweight : Google
Concrete---Flyweight : Google
objSize = 2
```

# 三. 总结
Flyweight(享元)模式是如此的重要，因为它能帮你在一个复杂的系统中大量的节省内存空间。在JAVA语言中，String类型就是使用了享元模式。String对象是final类型，对象一旦创建就不可改变。在JAVA中字符串常量都是存在常量池中的，JAVA会确保一个字符串常量在常量池中只有一个拷贝。String a="abc"，其中"abc"就是一个字符串常量。

可以共享的对象，也就是说返回的同一类型的对象其实是同一实例，当客户端要求生成一个对象时，工厂会检测是否存在此对象的实例，如果存在那么直接返回此对象实例，如果不存在就创建一个并保存起来，这点有些单例模式的意思。通常工厂类会有一个集合类型的成员变量来用以保存对象，如hashtable,vector等。在java中，数据库连接池，线程池等即是用享元模式的应用。
