[TOC]
##  一、为什么要使用事务？


如果我们一个业务逻辑只执行一次sql，是不需要使用事务的。但如果要**执行多条sql语句才能完成一个业务逻辑**的话，这个时候就要使用事务了。 因为这几条sql可能有的执行成功，有的执行失败。 而**事务就是对一组sql语句进行统一的提交或回滚操作**,为了保证数据执行的一致性，这组sql不是全部成功就是全部失败。

## 二.如何在spring中添加事务


* Spring提供了对事务的声明式事务管理，只需要在配置文件中做一些配置，即可把操作纳入到事务管理当中，解除了和代码的耦合。　　
* Spring声明式事务管理，核心实现就是基于Aop。　　
* Spring声明式事务管理是粗粒度的事务控制，只能给整个方法应用事务，不可以对方法的某几行应用事务。     
* Spring声明式事务管理器类：              
    *  Jdbc技术：DataSourceTransactionManager               
    *  Hibernate技术：HibernateTransactionManager
 
 ### 1.xml方式声明事务
  ```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:tx="http://www.springframework.org/schema/tx"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/aop
        http://www.springframework.org/schema/aop/spring-aop.xsd
        http://www.springframework.org/schema/tx
        http://www.springframework.org/schema/tx/spring-tx.xsd">
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <property name="driverClass" value="com.mysql.jdbc.Driver"></property>
        <property name="jdbcUrl" value="jdbc:mysql:///student"/>
        <property name="user" value="root"/>
        <property name="password" value="juaner767"/>
        <property name="initialPoolSize" value="3"/>
        <property name="maxPoolSize" value="6"/>
        <property name="maxStatements" value="100"/>
        <property name="acquireIncrement" value="2"/>
    </bean>
    <bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
        <constructor-arg ref="dataSource"/>
    </bean>
    <bean id="studentDao" class="com.juaner.spring.tx.StudentDao">
        <property name="jdbcTemplate" ref="jdbcTemplate"/>
    </bean>
    <bean id="studentService" class="com.juaner.spring.tx.StudentService">
        <property name="studentDao" ref="studentDao"/>
    </bean>
    <!--spring声明式事务管理控制-->
    <!--配置事务管理器类-->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--配置事务增强（如何管理事务，只读、读写...）-->
    <tx:advice id="txAdvice" transaction-manager="txManager">
        <tx:attributes>
            <tx:method name="*save*" read-only="false"/>
            <tx:method name="get*" read-only="true"/>
        </tx:attributes>
    </tx:advice>
    <!--aop配置，拦截哪些方法（切入点表达式，拦截上面的事务增强）-->
    <aop:config>
        <aop:pointcut id="pt" expression="execution(* com.juaner.spring.tx.*.*(..))"/>
        <aop:advisor advice-ref="txAdvice" pointcut-ref="pt"/>
    </aop:config>
</beans>
```

### 2.注解方式声明事务
   #### 开启注解扫描
    
```xml
<!--事务管理器类-->
    <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>
    <!--开启注解扫描-->
    <context:component-scan base-package="com.juaner.spring.tx"/>

    <!--注解方式实现事务-->
    <tx:annotation-driven transaction-manager="txManager"/>
```
#### 使用注解

```java
@Transactional
    public void save(Student student){
        this.studentDao.save(student);
        int i = 1/0;
        this.studentDao.save(student);
    }
```

如果@Transactional
应用到方法上，该方法使用事务；
应用到类上，类的方法使用事务，
定义到父类上，执行父类的方法时使用事务；

### 3.事务属性

```java
@Transactional(
            readOnly = false, //读写事务
            timeout = -1 ,     //事务的超时时间，-1为无限制
            noRollbackFor = ArithmeticException.class, //遇到指定的异常不回滚
            isolation = Isolation.DEFAULT, //事务的隔离级别，此处使用后端数据库的默认隔离级别
            propagation = Propagation.REQUIRED //事务的传播行为
    )
```

事务的第一个方面是传播行为（propagation behavior）。当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务，并在自己的事务中运行。Spring定义了七种传播行为：

| 传播行为| 含义 | 
| --- | --- | 
| PROPAGATION_REQUIRED | 表示当前方法必须运行在事务中。如果当前事务存在，方法将会在该事务中运行。否则，会启动一个新的事务 |
|  | | 
|PROPAGATION_SUPPORTS | 表示当前方法不需要事务上下文，但是如果存在当前事务的话，那么该方法会在这个事务中运行 |
| | | 
| PROPAGATION_MANDATORY | 表示该方法必须在事务中运行，如果当前事务不存在，则会抛出一个异常 |
|  | |
| PROPAGATION_REQUIRED_NEW | 表示当前方法必须运行在它自己的事务中。一个新的事务将被启动。如果存在当前事务，在该方法执行期间，当前事务会被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
|  | |
|PROPAGATION_NOT_SUPPORTED |表示该方法不应该运行在事务中。如果存在当前事务，在该方法运行期间，当前事务将被挂起。如果使用JTATransactionManager的话，则需要访问TransactionManager |
|  | |
| PROPAGATION_NEVER | 表示当前方法不应该运行在事务上下文中。如果当前正有一个事务在运行，则会抛出异常 |
|  | |
|PROPAGATION_NESTED | 表示如果当前已经存在一个事务，那么该方法将会在嵌套事务中运行。嵌套的事务可以独立于当前事务进行单独地提交或回滚。如果当前事务不存在，那么其行为与PROPAGATION_REQUIRED一样。注意各厂商对这种传播行为的支持是有所差异的。可以参考资源管理器的文档来确认它们是否支持嵌套事务 |

