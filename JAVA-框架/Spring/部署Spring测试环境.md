[TOC]
# JUnit4
引入依赖：
```xml
<!-- https://mvnrepository.com/artifact/junit/junit -->
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13</version>
    <scope>test</scope>
</dependency>

```
在测试类加上以下注解即可
```java
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
@RunWith(SpringJUnit4ClassRunner.class)
public class TestSpringTestEnvi {
@Autowired
OrbCommDaoMapping orbCommDaoMappingMapper;

@Test
public void testMapper() {
    System.out.println(orbCommDaoMappingMapper);
}
}

```

# JUnit5
引入依赖：
```xml
 <dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-api</artifactId>
    <version>5.6.2</version>
    <scope>test</scope>
</dependency>
```
在测试类加上以下注解即可
```java
@ContextConfiguration(locations = {"classpath:applicationContext.xml"})
@ExtendWith(SpringExtension.class)
public class TestSpringTestEnvi {
@Autowired
OrbCommDaoMapping orbCommDaoMappingMapper;

@Test
public void testMapper() {
    System.out.println(orbCommDaoMappingMapper);
}
}

```