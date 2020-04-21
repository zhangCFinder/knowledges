[TOC]
## 实例
1. 判断String是否为空
```xml
<if test="stringParam != null and stringParam != ''"></if>
```
2. 判断Integer是否大于0

```xml
<if test="idParam !=null and idParam gt 0"></if>
```
3. 判断List是否不为空
```xml
<if test="listParam !=null and listParam.size >0"></if>
```
4. 判断String是否以某特定字符(比如此处的"user")开头
```xml
<if test="stringParam.indexOf('user') != -1"></if>
```
5. 判断字符串是否等于特定字符(比如此处的user)
```xml
<if test='stringParam != null and stringParam == "user"'></if>
```
注意不能使用此写法 
```xml
<if test="stringParam != null and stringParam != 'user'"></if>
```
即最外边用双引号，里边用单引号，此写法会抱java.lang.NumberFormatException异常

如果要用这个写法要
```xml
<if test="stringParam != null and stringParam != 'user'.toString()"></if>
```
6. 判断MAP中是否包含某key,用_parameter.containsKey(变量名)来判断map中是否包含有些变量
```xml
<if test="_parameter.containsKey('msgType')"></if>
```

## 源码分析

```xml
<if test="type=='y'">  
    and status = 0   
</if> 
```
当传入的type的值为y的时候，if判断内的sql也不会执行，抱着这个疑问就去看了mybatis是怎么解析sql的。
下面我们一起来看一下mybatis 的执行过程。

* DefaultSqlSession.class  121行
```java
public void select(String statement, Object parameter, RowBounds rowBounds, ResultHandler handler) {  
    try {  
      MappedStatement ms = configuration.getMappedStatement(statement);  
      executor.query(ms, wrapCollection(parameter), rowBounds, handler);  
    } catch (Exception e) {  
      throw ExceptionFactory.wrapException("Error querying database.  Cause: " + e, e);  
    } finally {  
      ErrorContext.instance().reset();  
    }  
  }  
```
在 executor.query(ms, wrapCollection(parameter), rowBounds, handler);
执行到BaseExecutor.class执行器中的query方法

```java
public <E> List<E> query(MappedStatement ms, Object parameter, RowBounds rowBounds, ResultHandler resultHandler) throws SQLException {  
    BoundSql boundSql = ms.getBoundSql(parameter);  
    CacheKey key = createCacheKey(ms, parameter, rowBounds, boundSql);  
    return query(ms, parameter, rowBounds, resultHandler, key, boundSql);  
 } 
```
在query的方法中看到boundSql，是通过 ms.getBoundSql(parameter);获取的。
再点进去可以看到MappedStatement.class类中的getBoundSql方法

```java
public BoundSql getBoundSql(Object parameterObject) {  
    BoundSql boundSql = sqlSource.getBoundSql(parameterObject);  
    List<ParameterMapping> parameterMappings = boundSql.getParameterMappings();  
    if (parameterMappings == null || parameterMappings.size() <= 0) {  
      boundSql = new BoundSql(configuration, boundSql.getSql(), parameterMap.getParameterMappings(), parameterObject);  
    }  
    // check for nested result maps in parameter mappings (issue #30)  
    for (ParameterMapping pm : boundSql.getParameterMappings()) {  
      String rmId = pm.getResultMapId();  
      if (rmId != null) {  
        ResultMap rm = configuration.getResultMap(rmId);  
        if (rm != null) {  
          hasNestedResultMaps |= rm.hasNestedResultMaps();  
        }  
      }  
    }  
  
    return boundSql;  
  }  
```

看到其中有sqlSource.getBoundSql(parameterObject); sqlsource是一个接口。
```java
/** 
 *  
 * This bean represets the content of a mapped statement read from an XML file 
 * or an annotation. It creates the SQL that will be passed to the database out 
 * of the input parameter received from the user. 
 *  
 */  
public interface SqlSource {  
  
  BoundSql getBoundSql(Object parameterObject);  
  
}  
```
类中getBoundSql是一个核心方法，mybatis 也是通过这个方法来为我们构建sql。
BoundSql 对象其中保存了经过参数解析，以及判断解析完成sql语句。
比如<if> <choose> <when> 都会在这一层完成，
具体的完成方法往下看，那最常用sqlSource的实现类是DynamicSqlSource.class

```java
public class DynamicSqlSource implements SqlSource {  
  
  private Configuration configuration;  
  private SqlNode rootSqlNode;  
  
  public DynamicSqlSource(Configuration configuration, SqlNode rootSqlNode) {  
    this.configuration = configuration;  
    this.rootSqlNode = rootSqlNode;  
  }  
  
  public BoundSql getBoundSql(Object parameterObject) {  
    DynamicContext context = new DynamicContext(configuration, parameterObject);  
    rootSqlNode.apply(context);  
    SqlSourceBuilder sqlSourceParser = new SqlSourceBuilder(configuration);  
    Class<?> parameterType = parameterObject == null ? Object.class : parameterObject.getClass();  
    SqlSource sqlSource = sqlSourceParser.parse(context.getSql(), parameterType, context.getBindings());  
    BoundSql boundSql = sqlSource.getBoundSql(parameterObject);  
    for (Map.Entry<String, Object> entry : context.getBindings().entrySet()) {  
      boundSql.setAdditionalParameter(entry.getKey(), entry.getValue());  
    }  
    return boundSql;  
  } 
}  
```
核心方法是调用了rootSqlNode.apply(context); rootSqlNode是一个接口
```java

public interface SqlNode {  
  boolean apply(DynamicContext context);  
}  
```
可以看到类中 rootSqlNode.apply(context); 的方法执行就是一个递归的调用，通过不同的
实现类执行不同的标签，每一次appll是完成了我们<></>一次标签中的sql创建，计算出标签中的那一段sql，mybatis通过不停的递归调用，来为我们完成了整个sql的拼接。那我们主要来看IF的实现类IfSqlNode.class

```java
public class IfSqlNode implements SqlNode {  
  private ExpressionEvaluator evaluator;  
  private String test;  
  private SqlNode contents;  
  
  public IfSqlNode(SqlNode contents, String test) {  
    this.test = test;  
    this.contents = contents;  
    this.evaluator = new ExpressionEvaluator();  
  }  
  
  public boolean apply(DynamicContext context) {  
    if (evaluator.evaluateBoolean(test, context.getBindings())) {  
      contents.apply(context);  
      return true;  
    }  
    return false;  
  }  
  
}  
```

可以看到IF的实现中，执行了 if (evaluator.evaluateBoolean(test, context.getBindings()))  如果返回是false的话直接返回，否则继续递归解析IF标签以下的标签，并且返回true。那继续来看 evaluator.evaluateBoolean 的方法

```java
public class ExpressionEvaluator {  
  public boolean evaluateBoolean(String expression, Object parameterObject) {  
    Object value = OgnlCache.getValue(expression, parameterObject);  
    if (value instanceof Boolean) return (Boolean) value;  
    if (value instanceof Number) return !new BigDecimal(String.valueOf(value)).equals(BigDecimal.ZERO);  
    return value != null;  
  }  
```
关键点就在于这里，在OgnlCache.getValue中调用了Ognl.getValue，看到这里恍然大悟，mybatis是使用的OGNL表达式来进行解析的，在OGNL的表达式中，'y'会被解析成字符，因为java是强类型的，char 和 一个string 会导致不等。所以if标签中的sql不会被解析。具体的请参照 OGNL 表达式的语法。到这里，上面的问题终于解决了，只需要把代码修改成：

```xml

<if test='type=="y"'>  
    and status = 0   
</if>  
```
就可以执行了，这样"y"解析出来是一个字符串，两者相等！





  




