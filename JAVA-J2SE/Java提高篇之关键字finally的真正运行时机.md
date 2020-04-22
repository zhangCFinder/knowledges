
当在 catch 块和 finally 块同时return的时候，到底会return什么呢？

示例一：

```java
public static int testFinally1() {
    try {
        Integer.parseInt("exception here");
    } catch (Exception e) {
        System.out.println("catch block 1");
        return 11;
    } finally {
        System.out.println("finally block 1");
        return 12;
    }
}

```

事实上，即使` catch` 块有`return`语句， `finally `块必然执行的逻辑还是成立的。

上面的方法，`return`的是`12`。先输出`catch block 1`，然后输出`finally block 1`，最后返回`12`。


示例二:
```java
public static int testFinally3() {
    try {
        System.out.println("start");
        Integer.parseInt("testFinally3");
        System.out.println("never run");
    } catch (Exception e) {
        System.out.println("catch block 3");
        return iamReturn();
    } finally {
        System.out.println("finally block 3");
    }
    return 31;
}

public static int iamReturn() {
    System.out.println("return block");
    return 666;
}
```

根据我们之前发现的规律， `finally` 块必然执行，而 `catch `块的非`return`部分需要在` finally `块之前执行。那么最终返回的应该是666。

输出结果：
```log
start
catch block 3
return block
finally block 3
```

由此，我们可以总结 finally 块的真正运行时机了：

1. finally 块必然执行，不论发生异常与否，也不论在 finally 之前是否有return。

2. 不管在 try 块中还是在 catch 块中包含 return，**finally 块总是在 return 之前执行。**

3. 如果 finally 块中有return，那么 try 块和 catch 块中的 return 就没有执行机会了。

示例三：
```java
 public static int testFinally2() {
    try {
        System.out.println("start");
        return 21;
    } catch (Exception e) {
        System.out.println("catch block 2");
    } finally {
        System.out.println("finally block 2");
    }
    return 22;
}
```
输出结果：
```log
start
finally block 2
21
```