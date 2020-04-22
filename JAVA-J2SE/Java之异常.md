[TOC]
# 异常的分类
Java异常可分为3种：
* 编译时异常:Java.lang.Exception（必须catch）
    
    程序正确，但因为外在的环境条件不满足引发。
    例如：用户错误及I/O问题----程序试图打开一个并不存在的远程Socket端口。这不是程序本身的逻辑错误，而很可能是远程机器名字错误(用户拼写错误)。
    对商用软件系统，程序开发者必须考虑并处理这个问题。Java编译器强制要求处理这类异常，如果不捕获这类异常，程序将不能被编译。

* 运行期异常:Java.lang.RuntimeException（可选是否catch）
    这意味着程序存在bug，如数组越界，0被除，入参不满足规范.....这类异常需要更改程序来避免，Java编译器强制要求处理这类异常。

* 错误:Java.lang.Error
    一般很少见，也很难通过程序解决。它可能源于程序的bug，但一般更可能源于环境问题，如内存耗尽。错误在程序中无须处理，而有运行环境处理。


其中：
* `Java.lang.Exception`  和 `Java.lang.Error`继承自`Java.lang.Throwable`;
* `Java.lang.RuntimeException`继承自`Java.lang.Exception`.

# 常见的RuntimeException
1. NullPointerException：见的最多了，其实很简单，一般都是在null对象上调用方法了。
```java
String s=null;
boolean eq=s.equals(""); // NullPointerException
```
2. NumberFormatException：继承IllegalArgumentException，字符串转换为数字时。
```java
int i= Integer.parseInt("ab3");
```
3. ArrayIndexOutOfBoundsException:数组越界
```java
int[] a=new int[3]; 
int b=a[3]; 
```
4. StringIndexOutOfBoundsException：字符串越界
```java
String s="hello"; 
char c=s.chatAt(6);
```
5. ClassCastException:类型转换错误
```java
Object obj=new Object(); 
String s=(String)obj; 
```
6. UnsupportedOperationException:该操作不被支持，如果我们希望不支持这个方法，可以抛出这个异常。既然不支持还要这个干吗？有可能子类中不想支持父类中有的方法，可以直接抛出这个异常。

7. ArithmeticException：算术错误，典型的就是0作为除数的时候。

8. IllegalArgumentException：非法参数，在把字符串转换成数字的时候经常出现的一个异常，我们可以在自己的程序中好好利用这个异常。


