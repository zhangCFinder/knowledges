[TOC]
# 1. 不可变类
所谓不可变类，是指当创建了这个类的实例后，就不允许修改它的属性值。 它包括：

* 8种基本数据类型: `boolean,byte, char, double ,float, integer, long, short`

* jdk的不可变类：`jdk的java.lang包中 Boolean, Byte, Character, Double, Float, Integer, Long, Short, String`

# 2. 可变类

可变类，是当你获得这个类的一个实例引用时，你可以改变这个实例的内容。 

可变类对象间用“=”赋值，则会是使两个对象实际上会引用同一个实例。

所以，只有实现深度clone才能使可变类对象的修改不影响原始对象的值。

然而，对于不可变类，可变类的特性也是很有意义的，有的时候我们不希望重复创建相同的内容的实例。因此，我们也可以利用不可变类获得实例缓存。如：

```java
Integer a =Integer.valueOf(10);

Integer b = Integer.valueOf(10);
```
则只会在第一次创建取值为10的Integer对象。也就是说a和b指向同一处内存上的内容。

# 3. 如何创建不可变类
* 所有成员都是private

* 不提供对成员的改变方法，例如：setXXXX

* 确保所有的方法不会被重载。手段有两种：使用final Class(强不可变类)，或者将所有类方法加上final(弱不可变类)。

* 如果某一个类成员不是原始变量(primitive)或者不可变类，必须通过在成员初始化(in)或者get方法(out)时通过深度clone方法，来确保类的不可变。

反言之，任何提供了外部可修改途径的自定义类都是可变类。因而，只有实现深度clone才能使可变类对象按值赋给另一个对象，而不是按引用。

例如，在自定义的类 A 中，假设用户声明一个不可变类 B 的field，并提供了可修改 B 值的途径。显然，A便因此成为一个可变类，尽管它包括有不可变类的字段，或者根本没有可变类的字段。

# 4. clone在不可变类中的实践

有时候你要实现的immutable类中可能包含mutable的类，比如java.util.Date,尽管你将其设置成了final的，但是它的值还是可以被修改的，为了避免这个问题，我们建议返回给用户该对象的一个拷贝，这也是Java的最佳实践之一。下面是一个创建包含mutable类对象的immutable类的例子：
```java
public final class ImmutableReminder{
    private final Date remindingDate;
   
    public ImmutableReminder (Date remindingDate) {
        
        if(remindingDate.getTime() < System.currentTimeMillis()){
            throw new IllegalArgumentException("Can not set reminder for past time: " + remindingDate);
        }
        this.remindingDate = new Date(remindingDate.getTime());
    }
   
    public Date getRemindingDate() {
        return (Date) remindingDate.clone();
    }
}
```
上面的`getRemindingDate()`方法可以看到，返回给用户的是类中的remindingDate属性的一个拷贝。

这样的话如果别人通过`getRemindingDate()`方法获得了一个Date对象，然后修改了这个Date对象的值，那么这个值的修改将不会导致`ImmutableReminder`类对象中`remindingDate`值的修改。