[TOC]
> Lambda 表达式”(lambda expression)是一个匿名函数，Lambda表达式基于数学中的λ演算得名，直接对应于其中的lambda抽象(lambda abstraction)，是一个匿名函数，即没有函数名的函数。Java 8的一个大亮点是引入Lambda表达式，使用它设计的代码会更加简洁。当开发者在编写Lambda表达式时，也会随之被编译成一个函数式接口。

## Lambda简介


Lambda表达式的语法由参数列表、箭头符号->和函数体组成。函数体既可以是一个表达式，也可以是一个语句块。比如：
```java
(int x, int y) -> x + y;
```


## Java Lambda表达式示例

下面就用一些例子来体验一下Lambda表达式。

### 遍历集合

比如我们现在要遍历一个List：
```java
List<String> list = Arrays.asList("Hello", "JDK8", "and", "Lambda");
```
JDK8之前的写法：
```
for (String s : list) {
    System.out.println(s);
 }
 ```
 用Lambda表达式写法：
 
 ```java
 list.forEach(s -> System.out.println(s));
 ```
 
 可以看到，无论是代码量和可读性都得到了提高。
 
 在此基础上还可以再用隐式表达式进行简化
 
 ```java
list.forEach(System.out::println);
```

### 匿名类


在Java中很多时候我们要用到匿名类，比如线程Runnable、FileFilter和Comparator等等。
而匿名类型最大的问题就在于其冗余的语法。

这里用Comparator做例子。

比如我们有一个Cat类，表示猫，有名字、高度和重量这些属性。

```java
package com.fengyuan.model;

import lombok.AllArgsConstructor;
import lombok.Data;

public class Cat {
    private String name;
    private double height;
    private double weight;
}

```

我们创建3只猫，存到List中：
```java
List<Cat> catList = new ArrayList<>();
// 请无视这些数据的合理性，我乱写的
catList.add(new Cat("cat1", 10.3, 3.6));
catList.add(new Cat("cat2", 9.3, 4.6));
catList.add(new Cat("cat3", 9.5, 4.0));
```


然后我们现在要对这个List进行排序，但是现在不知道是要怎么排，所以我们要定义一个比较器，指定用高度或者是重量来排序。

JDK8之前的写法：

```java
// 指定用高度来排序
Collections.sort(catList, new Comparator<Cat>() {
    @Override
    public int compare(Cat o1, Cat o2) {
        if (o1.getHeight() > o2.getHeight()) {
            return 1;
        } else if (o1.getHeight() < o2.getHeight()) {
            return -1;
        } else {
            return 0;
        }
     }
});
```

而用Lambda，可以这样写：
```java
Collections.sort(catList, (o1, o2) -> {
    if (o1.getHeight() > o2.getHeight()) {
        return 1;
    } else if (o1.getHeight() < o2.getHeight()) {
        return -1;
    } else {
        return 0;
    }
});
```

继续用方法引用，可以简写到极致：
```java
// 指定用重量排序
catList.sort(Comparator.comparing(Cat::getWeight));
```

```java
// 要逆向排列也很简单
catList.sort(Comparator.comparing(Cat::getWeight).reversed());
```

到最后这种写法，已经简写到极致，而且可读性非常高。

## 函数式编程
见 [Java8新特性学习-函数式编程](https://blog.csdn.net/icarusliu/article/details/79495534)


