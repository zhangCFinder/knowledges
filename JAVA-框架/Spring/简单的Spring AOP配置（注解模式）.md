[TOC]
#### 1. 首先创建一个接口
```java
public interface Iface {
	 String say();

	void printThrowException();
}
```
#### 2. 创建实现类
```java
@Service
public class IfaceImpl implements Iface {
	@Override
	public String say() {
		System.out.println("hello Code");
		return "returnCode";
	}

	@Override
	public void printThrowException() {
		System.out.println("Exception raised");
		throw new IllegalArgumentException();
	}
}
```

#### 3. 创建一个简单的aspect切面class，并加上@注解配置。
##### 基本名词解释:
1. @Before： 标识一个前置增强方法，相当于BeforeAdvice的功能.
2. @After： final增强，不管是抛出异常或者正常退出都会执行.
3. @AfterReturning：  后置增强，似于AfterReturningAdvice, 方法正常退出时执行.
4. @AfterThrowing：  异常抛出增强，相当于ThrowsAdvice.
5. @Around： 环绕增强，相当于MethodInterceptor.
* 指定切点（Pointcut）
```java
@Component //一个类单纯如果只添加了`@Aspect`注解，那么它并不能被 context:component-scan 标签扫描到。
@Aspect   //表示是一个aspect切面class，注解以后可以被Spring扫描到
public class Logging {

    @Pointcut("execution(* com.zhangC.aop.IfaceImpl.*(..))")  //定义切点
    public void getMethod(){}; //切点方法，方法名称相当于ID，方法名称随便取

    /**
     * This is the method which I would like to execute
     * before a selected method execution.
     */
    @Before("getMethod()")   //切点方法执行之前执行@Before
    public void beforeAdvice() {
        System.out.println("Going to setup student profile.");
    }

    /**
     * This is the method which I would like to execute
     * after a selected method execution.
     */
    @After("getMethod()")   //切点方法执行之后执行@After
    public void afterAdvice() {
        System.out.println("Student profile has been setup.");
    }

@Around("getMethod()")//环绕通知，当有环绕通知时，需要增加返回值，如果没有返回值，则 @AfterReturning中获取的返回值就会为空。
public Object aroundAdvice	(ProceedingJoinPoint joinPoint) throws Throwable {
	System.out.println("AroundBefore");
	Object res = null;
	try {
		res = joinPoint.proceed();
	} catch (Exception e) {
		e.printStackTrace();
	}
	System.out.println("AroundAfter");
	return res;
}

    /**
     * This is the method which I would like to execute
     * when any method returns.
     */
    @AfterReturning(pointcut = "getMethod()",returning = "retVal")//切点方法返回值完成后执行@AfterReturning
    public void afterReturningAdvice(Object retVal) {
        System.out.println("Returning:" + retVal.toString());
    }


    /**
     * This is the method which I would like to execute
     * if there is an exception raised.
     */
    @AfterThrowing(pointcut = "getMethod()",throwing = "ex")//切点方法抛出异常之后执行@AfterThrowing
    public void AfterThrowingAdvice(IllegalArgumentException ex) {
        System.out.println("There has been an exception: " + ex.toString());
    }
}
```

* 不指定切点

```java
@Aspect
@Component
public class LoggerAspect {
	@Around("execution(* com.zhangC.aop.IfaceImpl.*(..))")
	public void aspectAround(ProceedingJoinPoint joinPoint) throws Throwable{
		System.out.println("方法调用前");
		joinPoint.proceed();
		System.out.println("方法调用后");
	}


	@Before("execution(* com.zhangC.aop.IfaceImpl.*(..))")
	public void aspectBefore() throws Throwable{
		System.out.println("方法调用前");
	}

	@After("execution(* com.zhangC.aop.IfaceImpl.*(..))")
	public void aspectAfter() throws Throwable{
		System.out.println("方法调用前");
	}

	@AfterReturning(pointcut = "execution(* com.zhangC.aop.IfaceImpl.*(..))",returning = "retVal") //切点方法返回值完成后执行@AfterReturning
	public void aspectAfterReturning(Object retVal) throws Throwable{
		System.out.println("Returning:" + retVal.toString());
	}

	@AfterThrowing(pointcut = "execution(* com.zhangC.aop.IfaceImpl.*(..))",throwing = "ex") //切点方法抛出异常之后执行@AfterThrowing
	public void aspectAfterThrowing(IllegalArgumentException ex) throws Throwable{
		System.out.println("There has been an exception: " + ex.toString());
	}
}

```


4. SpringAOP.xml配置
```xml
<!--注解方式配置Spring AOP，Spring会自动到已注册的bean中去寻找@Aspect注解标记的class-->
    <aop:aspectj-autoproxy/>
```
5. 测试方法
```java
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
@RunWith(SpringJUnit4ClassRunner.class)
public class TestAop {
	@Autowired
	private Iface iface;

	@Test
	public void testAopBySpring(){
		iface.say();
	}
}

```

* 注意事项：
```xml
<aop:aspectj-autoproxy proxy-target-class="true"/>
```

1. 如果不设置proxy-target-class属性(**默认为false**)或者设置成false：
（1）代理类实现了接口，那么AOP实际上是使用的JDK的自动代理，代理的接口。
（2）代理类没有接口，那么还是使用的CGLib进行代理。
2. 如果将proxy-target-class属性设置成true，那么将始终使用CGLib进行代理。

#### @Pointcut或@Around等括号内表达式分析：
示例
```java
//@Pointcut或@Around等表示式 
@Pointcut("execution(* com.savage.aop.MessageSender.*(..))")
```
#####  execution表达式
1. 分析一下这个`execution(* com.seeyon.SpringBean.aop.Student.get*(..))`切点表达式：
    (1) 第一个`*`代表方法的返回值是任意的
    (2) `get*`代表以`get`开头的所有方法
    (3) `(..)` 代表方法的参数是任意个数
 2. 分析一下execution(* com.seeyon.SpringBean.aop..*.*(..))这个切点表达式：
    (1)  第一个`*`表示方法的返回值是任意的
    (2) ` aop..` 表示aop包以及aop的子包
    (3) `aop..*` 表示aop包以及aop的子包下的所有class
    (4) `aop..*.*` 表示aop包以及aop的子包下的所有class的所有方法
    (5) `(..)` 表示方法参数任意
 
 ##### within表达式
    
1. pointcutexp包里的任意类的任意方法： `within(com.test.spring.aop.pointcutexp.*)`

2. pointcutexp包和所有子包里的任意类的任意方法：`within(com.test.spring.aop.pointcutexp..*)`


 ##### this表达式
  `this(com.test.spring.aop.pointcutexp.Intf)`
  
表示被spring代理之后生成的对象必须为`com.test.spring.aop.pointcutexp.Intf` 才会被拦截，所有使用jdk动态代理生成的类不是这个类型的，就不会被拦截，只有用cglib代理出来的类才能被拦截

##### target表达式
```java
//@Around("target(java类或接口)")//实现了该接口的类、继承该类、该类本身的类---的所有方法（包括不是接口定义的方法，包含父类的方法）会被拦截，即指目标对象，也叫被代理对象会被拦截
如：@Around("this(com.aop.service.TestService)") 
```

##### this 和 target 的不同点：

1. this作用于代理对象，target作用于目标对象

2. this表示目标对象被代理之后生成的代理对象和指定的类型匹配会被拦截，匹配的是代理对象

3. target表示目标对象和指定的类型匹配会被拦截，匹配的是目标对象
#### args表达式

1. 匹配方法中的参数　　　　
`@Pointcut("args(com.ms.aop.args.demo1.UserModel)")`
匹配只有一个参数，且类型为com.ms.aop.args.demo1.UserModel的方法

2. 匹配多个参数
　`args(type1,type2,typeN)`

3. 匹配任意多个参数
`@Pointcut("args(com.ms.aop.args.demo1.UserModel,..)")`
匹配第一个参数类型为`com.ms.aop.args.demo1.UserModel` 的所有方法, .. 表示任意个参数


#### 注解表达式

1. **@within和@target** -- 针对类
带有@Transactional标注的**所有类**的任意方法： 
```java
@within(org.springframework.transaction.annotation.Transactional)
@target(org.springframework.transaction.annotation.Transactional)
```
> 注意：(未验证，依旧不理解)
　@target 和 @within 的不同点
　　
　@target(注解A)：判断被调用的目标对象中是否声明了注解A，如果有，会被拦截
　　
　@within(注解A)： 判断被调用的方法所属的类中是否声明了注解A，如果有，会被拦截
　　
　@target关注的是被调用的对象，@within关注的是调用的方法所在的类


2. **@annotation**  -- 针对方法
带有@Transactional标注的**任意方法**：
```java
@annotation(org.springframework.transaction.annotation.Transactional)
```
3. **@args**
方法参数所属的类型上有指定的注解，@Transactional注解
```java
@args(org.springframework.transaction.annotation.Transactional)
```
注意：是方法参数所属的类型上有指定的注解，不是方法参数中有注解
　　
* 匹配1个参数，且第1个参数所属的类中有Anno1注解
　`@args(com.ms.aop.jargs.demo1.Anno1)`
* 匹配多个参数，且多个参数所属的类型上都有指定的注解
　`@args(com.ms.aop.jargs.demo1.Anno1,com.ms.aop.jargs.demo1.Anno2)`
* 匹配多个参数，且第一个参数所属的类中有Anno1注解
　`@args(com.ms.aop.jargs.demo2.Anno1,..)`


