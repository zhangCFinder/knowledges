this关键字总是指向调用该方法的对象。有两种情况：
1. 构造器中引用该构造器正在初始化的对象；
2. 在方法中引用调用该方法的对象。（这样，在该方法中，就不用再new一个新对象了）

示例：
````java
public class dog
{
     public void jump(){
          System.out.println("正在执行jump方法")；
     }
     public void run(){
          this.jump{};//使用this引用调用run()方法的对象
          System.out.println("正在执行run方法")；
     }
}
```
