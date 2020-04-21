[TOC]
# 编译期String字符串的限制

* 我们都知道JVM里面是包含常量池的,(是一种对字符串的性能优化，不用反复创建新的字符串了)

* 当我们使用字符串字面量直接定义String的时候，是会把字符串在常量池中存储一份的。

* 常量池中的每一项常量都是一个表，都有自己对应的类型。

* Java中的UTF-8编码的Unicode字符串在常量池中以CONSTANT_Utf8_info类型表,结构如下：
```java
CONSTANT_Utf8_info {

     u1 tag;

     u2 length;

     u1 bytes[length];

}
```

u2类型的length的值就表明了这个UTF-8编码字符串长度是多少字节。

所以CONSTANT_Utf8_info型常量对应的最大长度也就是java中UTF-8编码的字符串的长度，

顺便提一下Class文件中的方法和字段也是引用CONSTANT_Utf8_info型常量来描述名称的。

u2是无符号的16位整数，二进制首位标识符号，因此理论上允许的的最大长度是15个1即为2^16-1=65535


## 编译器javac下String的长度
创建一个测试类
```java
public class TestStr {
    public static void main(String[] args) {
             String LongStr ="aaaa..."//一共65535个a
             System.out.println(LongStr.length());
     }
}
```

使用javac命令编译它。编译报错。相应目录没有生成对应的TestStr.class文件。

去除一个字符串，使用65534个字符串。
```java
public class TestStr {
    public static void main(String[] args) {
             String LongStr ="aaaa..."//一共65534个a
             System.out.println(LongStr.length());
     }
}
```

javac命令编译它。编译正常。相应目录生成对应的TestStr.class文件。

我们在看看Oracle JDK的编译工具Javac内部，javac也是java写的。

```java
/** Check a constant value and report if it is a string that is
 *  too large.
 */private void checkStringConstant(DiagnosticPosition pos, Object constValue) {
    if (nerrs != 0 || // only complain about a long string once
        constValue == null ||
        !(constValue instanceof String) ||
        ((String)constValue).length() < Pool.MAX_STRING_LENGTH)
        return;
    log.error(pos, "limit.string");
    nerrs++;
}
...
```

再看看Pool.MAX_STRING_LENGTH
```java

public class Pool {

    ...
    
    public static final int MAX_STRING_LENGTH = 0xFFFF;
    
    ...
}
```

通过上边代码可以看到 MAX_STRING_LENGTH = 0xFFFF 而 0xFFFF 是十进制的 65535。

但是上面我们得出的结果是Javac编译下最大长度是65534，

是因为 Javac 源码中做的限制是((String)constValue).length() < Pool.MAX_STRING_LENGTH)

注意是 < 而不是 <= ， 小于65535那自然最多只能是65534了。


但是U2类型能表达的最大值是65535。上面65535个长度的字符串在javac下报错了是受到了javac编译器的限制了。

如果你在上面65534长度生成的TestStr.class中手动在添加一个字符串(注意是在javac编译后的class文件中添加)是可以得到65535长度的结果。


**总结一下：在Javac编译器下，字符串String的最大长度限制也即是U2类型所能表达的最大长度65534。避开javac最大长度是65535。**


# 运行期String的字符串限制

String内部是以char数组的形式存储，数组的长度是int类型，

那么String允许的最大长度就是Integer.MAX_VALUE了。

又由于java中的字符是以16位存储的，因此大概需要4GB的内存才能存储最大长度的字符串。

# 总结一下
1. Java中的字符串String最大长度，编译期如果是javac编译就是65534。如果绕过javac编译的限制，其最大长度可以达到u2类型变达的最大值65535。

2. Java中的字符串String最大长度运行期大约4G。
