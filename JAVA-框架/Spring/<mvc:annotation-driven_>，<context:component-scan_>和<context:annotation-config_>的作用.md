[TOC]

### 一、mvc:annotation-driven的作用

Spring 3.0.x中使用了mvc:annotation-driven后，默认会帮我们注册默认处理请求，参数和返回值的类，其中最主要的两个类：DefaultAnnotationHandlerMapping 和 AnnotationMethodHandlerAdapter ，分别为HandlerMapping的实现类和HandlerAdapter的实现类，是spring MVC为@Controllers分发请求所必须的，即解决了@Controller注解使用的前提配置。

在spring mvc 3.1以上，DefaultAnnotationHandlerMapping与AnnotationMethodHandlerAdapter对应变更为： 
**DefaultAnnotationHandlerMapping -> RequestMappingHandlerMapping 
AnnotationMethodHandlerAdapter -> RequestMappingHandlerAdapter 
AnnotationMethodHandlerExceptionResolver -> ExceptionHandlerExceptionResolver**  
以上都在使用了annotation-driven后自动注册。而且对应分别提供了AbstractHandlerMethodMapping , AbstractHandlerMethodAdapter和 AbstractHandlerMethodExceptionResolver以便于让用户更方便的实现自定义的实现类。 


* HandlerMapping的实现类的作用

实现类RequestMappingHandlerMapping，它会处理@RequestMapping 注解，并将其注册到请求映射表中。
 

* HandlerAdapter的实现类的作用

实现类RequestMappingHandlerAdapter，则是处理请求的适配器，确定调用哪个类的哪个方法，并且构造方法参数，返回值。
 
当配置了mvc:annotation-driven/后，Spring就知道了我们启用注解驱动。然后Spring通过<context:component-scan/>标签的配置，会自动为我们将扫描到的@Component，@Controller，@Service，@Repository等注解标记的组件注册到工厂中，来处理我们的请求。

### 二、使用的场景：


如果在web.xml中servlet-mapping的url-pattern设置的是/，而不是如.do。表示将所有的文件，包含静态资源文件都交给spring mvc处理。就需要用到<mvc:annotation-driven />了。如果不加，DispatcherServlet则无法区分请求是资源文件还是mvc的注解，而导致controller的请求报404错误。

```xml
 <servlet-mapping>
        <servlet-name>mvc-dispatcher</servlet-name>
        <url-pattern>/</url-pattern>
 </servlet-mapping>
```


基础的springmvc.xml的配置：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:aop="http://www.springframework.org/schema/aop"
xmlns:tx="http://www.springframework.org/schema/tx" xmlns:jdbc="http://www.springframework.org/schema/jdbc"
xmlns:context="http://www.springframework.org/schema/context"
xmlns:mvc="http://www.springframework.org/schema/mvc"
xsi:schemaLocation="http://www.springframework.org/schema/jdbchttp://www.springframework.org/schema/jdbc/spring-jdbc-3.0.xsd
http://www.springframework.org/schema/aophttp://www.springframework.org/schema/aop/spring-aop-3.0.xsd
http://www.springframework.org/schema/beanshttp://www.springframework.org/schema/beans/spring-beans-3.0.xsd
http://www.springframework.org/schema/contexthttp://www.springframework.org/schema/context/spring-context-3.0.xsd
http://www.springframework.org/schema/txhttp://www.springframework.org/schema/tx/spring-tx-3.0.xsd
http://www.springframework.org/schema/mvchttp://www.springframework.org/schema/mvc/spring-mvc-3.2.xsd">

<!--扫描Controller,并将其生命周期纳入Spring管理-->
<context:annotation-config/>

<context:component-scan base-package="com.how2java.controller">
<context:include-filter type="annotation"
expression="org.springframework.stereotype.Controller"/>
</context:component-scan>

<!--注解驱动，以使得访问路径与方法的匹配可以通过注解配置-->
<mvc:annotation-driven />

<!--通过location，可以重新定义资源文件的位置-->
<mvc:resources mapping="/styles/**" location="/WEB-INF/resource/styles/"/>

<!--静态页面，如html,css,js,images可以访问-->
<mvc:default-servlet-handler />

<!-- 视图定位到/WEB/INF/jsp 这个目录下 -->
<bean
class="org.springframework.web.servlet.view.InternalResourceViewResolver">
<property name="viewClass"
value="org.springframework.web.servlet.view.JstlView" />
<property name="prefix" value="/WEB-INF/jsp/" />
<property name="suffix" value=".jsp" />
</bean>
</beans>
```

### 三 源码分析

通常如果我们希望通过注解的方式来进行Spring MVC开发，我们都会在`***-servlet.xml`中加入`<mvc:annotation-driven/>`标签来告诉Spring我们的目的，那么这个标签到底做了什么呢，我们先看看它的解析类，我们知道所有的自定义命名空间（像mvc，context等）下的标签解析都是由BeanDefinitionParser接口的子类来完成的，先看图片：
![846e3ee6406b6f577ad60a21cba613e5](<mvc:annotation-driven_>，<context:component-scan_>和<context:annotation-config_>的作用.resources/E7F6AF87-EA31-4DD0-95CB-12D6EDAA6258.png)

我们看到有多个AnnotationDrivenBeanDefinitionParser，他们是用来处理不同命名空间下的<annotation-driven/>标签的，我们今天研究的是<mvc:annotation-driven/>标签，所以我们找到对应的实现类是：org.springframework.web.servlet.config.AnnotationDrivenBeanDefinitionParser。


#### AnnotationDrivenBeanDefinitionParser

通过阅读类注释文档，我们发现这个类主要是用来向工厂中注册了：

| 注册类 |作用 |
| --- | --- |
|RequestMappingHandlerMapping|处理@RequestMapping注解|
|BeanNameUrlHandlerMapping |将controller类的名字映射为请求url|
|以下三个用来处理请求的|具体点说就是确定调用哪个controller的哪个方法来处理当前请求|
|RequestMappingHandlerAdapter | 处理@Controller注解的处理器，支持自定义方法参数和返回值|
|HttpRequestHandlerAdapter| 继承HttpRequestHandler的处理器|
|SimpleControllerHandlerAdapter |继承自Controller接口的处理器|
|ExceptionHandlerExceptionResolver| 处理异常的解析器|
|ResponseStatusExceptionResolver | 处理异常的解析器|
|DefaultHandlerExceptionResolver| 处理异常的解析器|

### Spring是怎么解析`<mvc:annotation-driven/>`标签的？

首先，必须要有一个继承自"org.springframework.beans.factory.xml.NamespaceHandlerSupport"的类，在其init方法中，注册自己的解析器，注册mvc解析器的类为MvcNamespaceHandler。

一般针对每个元素，都有一个解析器，比如：针对annotation-driven，就有一个解析器：就是前面提到的AnnotationDrivenBeanDefinitionParser。 

解析器必须实现org.springframework.beans.factory.xml.BeanDefinitionParser接口，这个接口只有一个parse方法，它有两个参数，第一个参数org.w3c.dom.Element就是我们在xml文件中声明的<mvc:annotation-driven/>结点，拿到这个结点信息，就可以开始具体的业务了。

### Spring怎么知道处理mvc开头的标签就调用`MvcNamespaceHandler`中注册的解析器的呢？


这需要有一个"mvc”<–>MvcNamespaceHandler这样一个映射关系，那么这个映射关系在哪里呢？就在META-INF目录下的spring.handlers:源文件中的内容：

    http\://www.springframework.org/schema/mvc=org.springframework.web.servlet.config.MvcNamespaceHandler  
 
这里定义了只要是http\://www.springframework.org/schema/mvc命名空间的标签，就使用org.springframework.web.servlet.config.MvcNamespaceHandler中的解析器。

 头文件里说的http://www.springframework.org/schema/mvc/spring-mvc-3.1.xsd，并不是真的到网上去下载这个文件，在spring.schemas文件中，定义了它指向org/springframework/web/servlet/config/spring-mvc-3.1.xsd这个文件（在jar包里）。
所以，在Spring中，想使用自己的命名空间：
  1. 首先需要一个xsd文件，来描述自定义元素的命名规则，并在再Spring的配置文件的<benas>头中引用它。
  2. 然后需要实现一个BeanDefinitionParser接口，在接口的parse方法中，解析将来在Spring配置文件中出现的元素。（如果xsd声明可以有多个元素，需呀实现多个BeanDefinitionParser接口）
  3. 最后需要继承一个NamespaceHandlerSupport类，在它的init方法中，调用registerBeanDefinitionParser方法，将待解析的xml元素与解析器绑定。
  4. 在META-INF目录下，创建spring.schemas、spring.handlers文件，建立最高级的映射关系以便Spring进行处理。
 
 ### `<mvc: annotation-driven/>` 与`<context:component-scan/>` 的区别
  
  
1. `<context:component-scan/>`标签是告诉Spring 来扫描指定包下的类，并注册被`@Component，@Controller，@Service，@Repository`等注解标记的组件。
而`<mvc:annotation-driven/>`是告知Spring，我们启用注解驱动。然后Spring会自动为我们注册上面说到的几个Bean到工厂中，来处理我们的请求。
2. `<context:component-scan>` 有一个`use-default-filters` 属性，该属性默认为true,这就意味着会扫描指定包下的全部的标有@Component的类，并注册成bean.也就是@Component的子注解@Service,@Reposity等。
3. `<context:incluce-filter>`  和 `<context:exclude-filter>` 标签
* 在SpringMVC的配置文件中
```xml
<context:component-scan base-package="tv.huan.weisp.web .controller"> 
    <context:include-filter 
    type="annotation" expression="org.springframework.stereotype.Controller"/>   
</context:component-scan>  
```
use-default-filter` 为true
此时只会扫描`@Controller`注解，但需要指定 ` base-package` 只为controller包，如果还包含了别的包,则include-filter不起作用，也会扫描其他包的其他注解。需要指定 `Use-dafault-filters=”false”` 。

* 在Spring的配置文件中

```xml
<context:component-scan base-package="com.ssm.user">
        <context:exclude-filter type="annotation"
        expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```
指定不扫描哪些注解标识的类，此时不用再使用use-default-filters指定，以为该属性默认为true，即该扫描器相关的注解@Controller、@Service等标识的类都会被扫描到，所以不用显示指定，只需使用子标签context:exclude-filter指定不扫描哪些注解标识的类即可。

### `<mvc: annotation-driven/>` 与`<context:annotation-config/>` 的区别


当我们需要使用注解模式时，直接在Spring配置文件中定义这些Bean显得比较笨拙，例如：使用@Autowired注解，必须事先在Spring容器中声明AutowiredAnnotationBeanPostProcessor的Bean：
```xml

<bean class="org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor "/>
```

使用 @Required注解，就必须声明RequiredAnnotationBeanPostProcessor的Bean：
```xml

<bean class="org.springframework.beans.factory.annotation.RequiredAnnotationBeanPostProcessor"/>
```


简单的说，用什么注解，就需要声明对应的BeanPostProcessor。
这样的声明未免太不优雅，而Spring为我们提供了一种极为方便注册这些`BeanPostProcessor`的方式，即使用`<context:annotation-config/>`隐式地向 Spring容器注册
`AutowiredAnnotationBeanPostProcessor、
RequiredAnnotationBeanPostProcessor、
CommonAnnotationBeanPostProcessor、
PersistenceAnnotationBeanPostProcessor这4个BeanPostProcessor`。

另外，在我们使用注解时一般都会配置扫描包路径选项，即`<context:component-scan/>`。该配置项其实也包含了自动注入上述processor的功能，**因此当使用`<context:component-scan/>`后，即可将`<context:annotation-config/>`省去。**





