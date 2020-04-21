[TOC]
JAVA中一共有八种基本数据类型，他们分别是
`byte、short、int、long、float、double、char、boolean`

### 1. 整型
其中byte、short、int、long都是表示整数的，只不过他们的取值范围不一样。

* byte的取值范围为-128~127，占用1个字节（-2的7次方到2的7次方-1）

* short的取值范围为-32768~32767，占用2个字节（-2的15次方到2的15次方-1）

* int的取值范围为（-2147483648~2147483647），占用4个字节（-2的31次方到2的31次方-1）

* long的取值范围为（-9223372036854774808~9223372036854774807），占用8个字节（-2的63次方到2的63次方-1）
#### 如何根据占用字节数计算取值范围(以byte为例)
> java中用补码表示二进制数，补码的最高位是符号位，最高位为“0”表示正数，最高位为“1”表示负数。
> * 正数补码为其本身；
> * 负数补码为其绝对值各位取反加1；
>例如：
>* +21，其二进制表示形式是00010101，则其补码同样为00010101
>* -21，按照概念其绝对值为00010101，各位取反为11101010，再加1为11101011，即-21的二进制表示形式为11101011

步骤：
1. byte为一字节8位，最高位是符号位，即最大值是01111111，因正数的补码是其本身，即此正数为01111111=2的6次方+2的5次方+。。。+2的0次方=2的7次方-1
十进制表示形式为127

2. 最大正数是01111111，那么最小负是10000000(最大的负数是11111111，即-1),此时均为补码的表示形式。

3. 10000000是最小负数的补码表示形式，我们把补码计算步骤倒过来就即可。10000000减1得01111111然后取反10000000
因为负数的补码是其绝对值取反，即10000000为最小负数的绝对值，而10000000的十进制表示是128，所以最小负数是-128

4. 由此可以得出byte的取值范围是-128到+127

5. 说明：各个类型取值范围的计算方法与此大致相同，感兴趣的同学可以自己试着计算

### 2. 浮点型
float和double是表示浮点型的数据类型，他们之间的区别在于精确度不同：
* float 3.402823e+38 ~ 1.401298e-45（e+38表示是乘以10的38次方，同样，e-45表示乘以10的负45次方）占用4个字节

* double 1.797693e+308~ 4.9000000e-324 占用8个字节

double型比float型存储范围更大，精度更高，所以通常的浮点型的数据在不声明的情况下都是double型的，如果要表示一个数据是float型的，可以在数据后面加上“F”。

浮点型的数据是不能完全精确的，所以有的时候在计算的时候可能会在小数点最后几位出现浮动，这是正常的。

**float和double或者其包装类，都可以直接用==和0做判断。**
#### 浮点数（float,double）整数部分达到8位及以上，会以科学计数法显示。
如何避免：
```java
public static void main(String[] args) {
double d = 123456789.128d;
String s1 = big(d);
String s2 = big2(d);
System.out.println(d);
System.out.println(s1);
System.out.println(s2);
}

// 方法一：NumberFormat
private static String big(double d) {
NumberFormat nf = NumberFormat.getInstance();
// 是否以逗号隔开, 默认true以逗号隔开,如[123,456,789.128]
nf.setGroupingUsed(false);
// 结果未做任何处理
return nf.format(d);
}

//方法二： BigDecimal
private static String big2(double d) {
BigDecimal d1 = new BigDecimal(Double.toString(d));
BigDecimal d2 = new BigDecimal(Integer.toString(1));
// 四舍五入,保留2位小数
return d1.divide(d2,2,BigDecimal.ROUND_HALF_UP).toString();
}
```
运行效果：
```log
1.23456789128E8
123,456,789.128
123456789.13
```


### 3. boolean型（布尔型）
这个类型只有两个值，true和false（真和非真）
boolean t = true；
boolean f = false；

### 4. char型（文本型）
用于存放字符的数据类型，占用2个字节，采用unicode编码，它的前128字节编码与ASCII兼容

字符的存储范围在\u0000~\uFFFF，在定义字符型的数据时候要注意加' '，比如 '1'表示字符'1'而不是数值1。
>char c = ' 1 ';
我们试着输出c看看，System.out.println(c);结果就是1，而如果我们这样输出呢System.out.println(c+0);
结果却变成了49。
如果我们这样定义c看看
char c = ' \u0031 ';输出的结果仍然是1，这是因为字符'1'对应着unicode编码就是\u0031
char c1 = 'h',c2 = 'e',c3='l',c4='l',c5 = 'o';
System.out.print(c1);
System.out.print(c2);
System.out.print(c3);
System.out.print(c4);
System.out.print(c5);

### 5. char与byte的区别

1. Char是无符号型的，可以表示一个整数，不能表示负数；而byte是有符号型的，可以表示-128—127 的数；如：
```java
char c = (char) -3; // char不能识别负数，必须强制转换否则报错，即使强制转换之后，也无法识别
System.out.println(c);//输出？
byte d1 = 1;
byte d2 = -1;
byte d3 = 127; // 如果是byte d3 = 128;会报错
byte d4 = -128; // 如果是byte d4 = -129;会报错
System.out.println(d1);//输出1
System.out.println(d2);//输出-1
System.out.println(d3);//输出127
System.out.println(d4);//输出-128
```

2. char可以表中文字符，byte不可以，如：
```java
char e1 = '中', e2 = '国';
byte f= (byte) '中';	//必须强制转换否则报错
System.out.println(e1);//输出'中'
System.out.println(e2);//输出'国'
System.out.println(f);//输出'45'
```

3. char、byte、int对于英文字符，可以相互转化，如：
```java
byte g = 'b';	//b对应ASCII是98
char h = (char) g;
char i = 85;	//U对应ASCII是85
int j = 'h';	//h对应ASCII是104
System.out.println(g);//输出'98'
System.out.println(h);//输出'b'
System.out.println(i);//输出'U'
System.out.println(j);//输出'104'
```