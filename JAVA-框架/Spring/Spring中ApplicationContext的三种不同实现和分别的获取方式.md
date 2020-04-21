[TOC]

Spring有两个核心接口：`BeanFactory` 和 `ApplicationContext`，其中ApplicationContext是BeanFactory 的子接口。它们代表Spring容器，Spring容器是生成Bean实例的工厂，并管理容器中的Bean，Bean是Spring管理的基本单位，在基于spring的javaEE应用中，所有的组件都被当成bean来处理。

   很多时候， ApplicationContext 都是以声明式方式操作容器，无须手动创建。例如：可利用像ContextLoader 的支持类，在 Web 应用启动时自动创建 ApplicationContext。当然，也可以采用编程方式创建 ApplicationContext。


### 1. FileSystemXmlApplicationContext
这种方式是通过程序在初始化的时候，导入Bean配置文件，然后得到Bean实例。
```java
ApplicationContext ctx = newFileSystemXmlApplicationContext("spring-config.xml"); //当前路径加载单个配置文件

String[] locations = {"bean1.xml", "bean2.xml", "bean3.xml"};

ApplicationContext ctx = new FileSystemXmlApplicationContext(locations ); //同时加载多个配置文件

ApplicationContext ctx = new FileSystemXmlApplicationContext("D:/project/bean.xml");//根据具体路径加载文件

```

对于FileSystemXmlApplicationContext:

默认表示的是两种:

1. 没有盘符的是项目工作路径,即项目的根目录;

2. 有盘符表示的是文件绝对路径.

注意：如果要使用classpath路径,需要前缀classpath:

### 2. ClassPathXmlApplicationContext
```java
ApplicationContext ctx = new ClassPathXmlApplicationContext("bean.xml");//加载单个配置文件

String[] locations = {"bean1.xml", "bean2.xml", "bean3.xml"};

ApplicationContext ctx = newClassPathXmlApplication(locations);//同时加载多个配置文件。

ApplicationContext ac = new ClassPathXmlApplicationContext("classpath:/*.xml");//用通配符同时加载多个配置文件

```
注：其中FileSystemXmlApplicationContext和ClassPathXmlApplicationContext与BeanFactory的xml文件定位方式一样是基于路径的。

ClassPathXmlApplicationContext和FileSystemXmlApplicationContext的区别如下：

1. 对于ClassPathXmlApplicationContext，`classpath:`前缀是不需要的,默认就是指项目的classpath路径下面;

2. 对于ClassPathXmlApplicationContext，如果要使用绝对路径,需要加上`file:`前缀表示这是绝对路径;

### 3. WebApplicationContext
spring和springmvc不在web容器的掌控范围里。因此，我们无法在这些类中直接使用Spring注解的方式来注入我们需要的对象，是无效的，web容器是无法识别的。

但我们有时候又确实会有这样的需求，比如在容器启动的时候，做一些验证或者初始化操作，这时可能会在监听器里用到bean对象；又或者需要定义一个过滤器做一些拦截操作，也可能会用到bean对象。那么在这些地方怎么获取spring的ApplicationContext中的bean对象呢？

#### 一，要想怎么获取 ApplicationContext, 首先必须明白 Spring 内部 ApplicationContext 是怎样存储的。

> 在web应用中，使用的是WebApplicationContext，它继承于ApplicationContext的一个接口，需要ServletContext实例,也就是说它必须拥有Web容器的前提下才能完成启动的工作。必须先启动Web容器，然后才能启动WebApplicationContext。
> 在非web应用下，Bean只有singleton和prototype两种作用域，WebApplicaitonContext为Bean添加了三个新的作用域：request/session/global session。

> Spring分别提供了用于启动WebApplicationContext的Servlet和Web容器监听器:
```java
org.springframework.web.context.ContextLoaderServlet;
org.springframework.web.context.ContextLoaderListener.
```
> 这两个方法都是在web应用启动的时候来初始化WebApplicationContext，我个人认为Listerner要比Servlet更好一些，因为Listerner监听应用的启动和结束，而Servlet得启动要稍微延迟一些，如果在这时要做一些业务的操作，启动的前后顺序是有影响的。
![5a05c591a204fc9f28748e5a2fdb60a5](Spring中ApplicationContext的三种不同实现和分别的获取方式.resources/93EF8300-5E55-456E-B8A6-DD5CD20683F1.png)



首先：从大家最熟悉的地方开始 
```xml
<listener>       
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>  
```

让我们看一看它到底实现了些啥。

```java
public class ContextLoaderListener implements ServletContextListener {
	private ContextLoader contextLoader;

	/**
	 * Initialize the root web application context.
	 */
	public void contextInitialized(ServletContextEvent event) {
		this.contextLoader = createContextLoader();
		this.contextLoader.initWebApplicationContext(event.getServletContext());
	}//下面的略
```

显然，ContextLoaderListener实现了ServeletContextListenet,在ServletContext初始化的时候，会进行Spring的初始化，大家肯定会想，Spring的初始化应该与ServletContext有一定关系吧？有关系吗？接下来让我们进入ContextLoader.initWebApplicationContext方法 

```java
	public WebApplicationContext initWebApplicationContext(ServletContext servletContext) throws IllegalStateException, BeansException {
		//从ServletContext中查找，是否存在以WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE为Key的值 
		if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
			throw new IllegalStateException("Cannot initialize context because there is already a root application context present - " + "check whether you have multiple ContextLoader* definitions in your web.xml!");
		}
		try {
			// Determine parent for root web application context, if any.
			ApplicationContext parent = loadParentContext(servletContext);
			// it is available on ServletContext shutdown. 
			this.context = createWebApplicationContext(servletContext, parent);
			//将ApplicationContext放入ServletContext中，其key为WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE 
			servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);
			//将ApplicationContext放入ContextLoader的全局静态常量Map中，其中key为：Thread.currentThread().getContextClassLoader()即当前线程类加载器 
			currentContextPerThread.put(Thread.currentThread().getContextClassLoader(), this.context);
			return this.context;
		}
	}
```

从上面的代码大家应该明白了Spring初始化之后，将ApplicationContext存到在了两个地方，那么是不是意味着我们可以通过两种方式取得ApplicationContext?

#### 第一种获取方式：


注意：`WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE = WebApplicationContext.class.getName() + ".ROOT"` ;即为 `
"org.springframework.web.context.WebApplicationContext.ROOT"`

那么咱们是不是可以这样获得ApplicationContext:
```java
request.getSession().getServletContext().getAttribute("org.springframework.web.context.WebApplicationContext.ROOT")  
```

确实可以，而且我们想到这种方法的时候，Spring早就提供给我们接口了：

```java
public abstract class WebApplicationContextUtils {
	public static WebApplicationContext getRequiredWebApplicationContext(ServletContext sc) throws IllegalStateException {
		WebApplicationContext wac = getWebApplicationContext(sc);
		if (wac == null) {
			throw new IllegalStateException("No WebApplicationContext found: no ContextLoaderListener registered?");
		}
		return wac;
	}
}
```


getWebApplicationContext方法如下：
```java

public static WebApplicationContext getWebApplicationContext(ServletContext sc) {       
		return getWebApplicationContext(sc, WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);      

}
```
所以，获取方式为：
```java
WebApplicationContext webApplicationContext = WebApplicationContextUtils.getWebApplicationContext(request.getSession().getServletContext());
UserService userService = (UserService) webApplicationContext.getBean("userService");
```
**注意：以上代码有一个前提，那就是servlet容器在实例化ConfigListener并调用其方法之前，要确保spring容器已经初始化完毕！而spring容器的初始化也是由Listener（ContextLoaderListener）完成，因此只需在web.xml中先配置初始化spring容器的Listener，然后在配置自己的Listener。**

#### 第二种获取方式：

前面说到Spring初始化的时候，将ApplicationContext还存了一份到ContextLoader的Map里面，那么我们是不是可以通过Map.get(key)获得，很不幸的是，这个Map是私有的。LOOK:
```java
private static final Map currentContextPerThread = 
CollectionFactory.createConcurrentMapIfPossible(1);
```

不过，不用担心Spring为我们提供了方法：再LOOK：
```java
	public static WebApplicationContext getCurrentWebApplicationContext() {
		return (WebApplicationContext) currentContextPerThread.get(Thread.currentThread().getContextClassLoader());
	}
```

第二种方法与第一种方法相比有什么好的地方呢？就是它不需要参数，只要在Web容器中，当Spring初始化之后，你不需要传入任何参数，就可以获得ApplicationContext为咱们服务。

所以，获取方式为：
```java
ApplicationContext ctx = ContextLoader.getCurrentWebApplicationContext();
```


#### 第三种方式：借用ApplicationContextAware
`ApplicationContext`的帮助类能够自动装载`ApplicationContext`，只要你将某个类实现这个接口，并将这个实现类在Spring配置文件中进行配置，Spring会自动帮你进行注入 `ApplicationContext.ApplicationContextAware`的代码结构如下：

```java
public interface ApplicationContextAware {              
    void setApplicationContext(ApplicationContext applicationContext) throws BeansException;
}  
```

就这一个接口。可以这样简单的实现一个ApplicationContextHelper类：
```java
public class ApplicationHelper implements ApplicationContextAware {
	private ApplicationContext applicationContext;

	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		this.applicationContext = applicationContext;
	}

	public ApplicationContext getApplicationContext() {
		return this.applicationContext;
	}
}
```

通过ApplicationHelper我们就可以获得想要的AppilcationContext类了

另： SpringContextHolder 静态持有ApplicationContext的引用，相当于一个ApplicationHelper

```java
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
 
/**
 * 以静态变量保存Spring ApplicationContext, 可在任何代码任何地方任何时候中取出ApplicaitonContext.
 */
public class SpringContextHolder implements ApplicationContextAware {
	private static ApplicationContext applicationContext;
 
	/**
	 * 实现ApplicationContextAware接口的context注入函数, 将其存入静态变量.
	 */
	public void setApplicationContext(ApplicationContext applicationContext) {
		SpringContextHolder.applicationContext = applicationContext; // NOSONAR
	}
 
	/**
	 * 取得存储在静态变量中的ApplicationContext.
	 */
	public static ApplicationContext getApplicationContext() {
		checkApplicationContext();
		return applicationContext;
	}
 
	/**
	 * 从静态变量ApplicationContext中取得Bean, 自动转型为所赋值对象的类型.
	 */
	@SuppressWarnings("unchecked")
	public static <T> T getBean(String name) {
		checkApplicationContext();
		return (T) applicationContext.getBean(name);
	}
 
	/**
	 * 从静态变量ApplicationContext中取得Bean, 自动转型为所赋值对象的类型.
	 */
	@SuppressWarnings("unchecked")
	public static <T> T getBean(Class<T> clazz) {
		checkApplicationContext();
		return (T) applicationContext.getBeansOfType(clazz);
	}
 
	/**
	 * 清除applicationContext静态变量.
	 */
	public static void cleanApplicationContext() {
		applicationContext = null;
	}
 
	private static void checkApplicationContext() {
		if (applicationContext == null) {
			throw new IllegalStateException(
					"applicationContext未注入,请在applicationContext.xml中定义SpringContextHolder");
		}
	}
}
```