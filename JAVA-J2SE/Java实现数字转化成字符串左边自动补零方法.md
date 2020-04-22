[TOC]
# 方法一,使用NumberFormat
```java
import java.text.NumberFormat;
public class NumberFormatTest {
    public static void main(String[] args) {
        // 待测试数据
        int i = 1;
        // 得到一个 NumberFormat的实例
        NumberFormat nf = NumberFormat.getInstance();
        // 设置是否使用分组
        nf.setGroupingUsed(false);
        // 设置最大整数位数
        nf.setMaximumIntegerDigits(4);
        // 设置最小整数位数    
        nf.setMinimumIntegerDigits(4);
        // 输出测试语句
        System.out.println(nf.format(i));
    }
}
```
# 方法二，使用String.format()函数
```java
public class TestStringFormat {      
public static void main(String[] args) {      
int youNumber = 1;      
// 0  代表前面补充 0     
// 4  代表长度为 4     
// d  代表参数为正数型      
    String str = String.format("%04d", youNumber);      
    System.out.println(str); // 0001     
  }      
}   
```
# 方法三，使用DecimalFormat
```java
//流水号加1 后返回，流水号长度为 4
private static final String STR_FORMAT = "0000";
public static String haoAddOne(String liuShuiHao){
    Integer intHao = Integer.parseInt(liuShuiHao);
    intHao++;
    DecimalFormat df = new DecimalFormat(STR_FORMAT);
    return df.format(intHao);
}
```
# 方法四，使用substring
```java
//前提是你的长度已经确定！比如规定现实 10位！
int i_m = 27 ;
String str_m = String.valueOf(i_m);
String str ="0000000000";
str_m=str.substring(0, 10-str_m.length())+str_m;
System.out.println(str_m);
```


