[TOC]
### 配置方式
有一种方案可以方便我们在一个阶段内不需要频繁写一个参数的值，而在不同阶段间又可以方便的切换参数的配置信息。
解决：spring3中提供了一种简便的方式就是 `<content:property-placeholder>`元素只需要在spring配置文件中添加一句：
```xml
<context:property-placeholder location="classpath:jdbc.properties"/>
```
或者：

```xml
<bean id="propertyPlaceholderConfigurer" class="org.springframework,beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="locations">
        <list>
            <value>jdbc.properties<value/>
        </list>
    </property>
</bean>
```

即可，这里的location值为参数配置文件的位置，配置文件通常放到src目录下，参数配置文件的格式即键值对的形式，
```xml
#jdbc配置 
driverClassName=com.mysql.jdbc.Driver 
url=jdbc:mysql://localhost:3306/test 
username=root 
password=root
```

行内#号后面部分为注释

这样一来就可以为spring配置的bean的属性设置值了，比如spring有一个数据源的类
```xml
<bean id="dataSource" class="org.springframework,jdbc,datasource.DriverManagerDataSource">
    <property name="driverClassName" value="${driverClassName}"/>
    <property name="url" value="${url}"/>
    <property name="username" value="${username}"/>
    <property name="password" value="${password}"/>
</bean>
```

甚至可以将 ${} 这种形式的变量用在spring提供的注解当中，为注解的属性提供值

### 配置注意事项

Spring容器仅允许最多定义一个 `PropertyPlaceholderConfigurer` 或 `<content:property-placeholder>`其余的会被Spring忽略。

因为Spring容器采用反射扫描的发现机制，在探测到Spring容器中有一个 org.springframework.beans.config.PropertyPlaceholderConfigurer的Bean就会停止对剩余PropertyPlaceholderConfigurer的扫描，


由于Spring容器只能有一个PropertyPlaceholderConfigurer，如果有多个属性文件，这时就看谁先谁后了，先的保留 ，后的忽略。

### Spring 自动注入 properties文件中的参数值

要自动注入properties文件中的配置，需要在Spring配置文件中添加  
`org.springframework.beans.factory.config.PropertiesFactoryBean` 和`org.springframework.beans.factory.config.PreferencesPlaceholderConfigurer`的实例配置
```xml
<bean id="configProperties" class="org.springframework.beans.factory.config.PropertiesFactoryBean">
    <property name="locations">
        <list>
            <value> classpath*:application.properties</value>
        </list>
    </property>
</bean>
<bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PreferencesPlaceholderConfigurer">
    <property name="properties" ref="configProperties" />
</bean>
```
在这个配置文件中我们配置了注解扫描，和configProperties实例和propertyConfigurer实例，这样我们就可以在java类中自动注入配置了

```java
@Component
public class Test{
    @Value("#{configProperties['userName']}")
    private String userName;

    public String getUserName(){
        return userName;
    }
}
```

自动注入需要使用 @Value 这个注解，这个注解的格式 `#{configProperties['userName']}` 其中configProperties是我们在配置文件中配置的bean的 id， userName 是在配置文件中配置的项

