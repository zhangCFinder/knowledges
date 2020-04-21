[TOC]
# 1. 第一种方式是spring2时代的产物，也就是每个json视图controller配置一个JsonView。
如：`<bean id="defaultJsonView" class="org.springframework.web.servlet.view.json.MappingJacksonJsonView"/> `

或者`<bean id="defaultJsonView" class="org.springframework.web.servlet.view.json.MappingJackson2JsonView"/>`

同样要用jackson的jar包。

# 2. 第二种使用JSON工具将对象序列化成json，常用工具Jackson，fastjson，gson。

利用HttpServletResponse，然后获取`response.getOutputStream()`或`response.getWriter()` 直接输出。

示例：
```java
public class JsonUtil
{
    
    private static Gson gson=new Gson();
 
 
    /**
     * @MethodName : toJson
     * @Description : 将对象转为JSON串，此方法能够满足大部分需求
     * @param src  :将要被转化的对象
     * @return :转化后的JSON串
     */
    public static String toJson(Object src) {
        if (src == null) {
            return gson.toJson(JsonNull.INSTANCE);
        }
        return gson.toJson(src);
    }
}
```

# 3. 利用springMvc3的注解@ResponseBody

```java
@ResponseBody
@RequestMapping("/list")
public List<String> list(ModelMap modelMap) {
    String hql = "select c from Clothing c ";
    Page<Clothing> page = new Page<Clothing>();
    page.setPageSize(6);
    page  = clothingServiceImpl.queryForPageByHql(page, hql);

    return page.getResult();
}
```
然后使用spring mvc的默认配置就可以返回json了，不过需要jackson的jar包哦。

注意：当springMVC-servlet.xml中使用<mvc:annotation-driven />时，如果是3.1之前已经默认注入AnnotationMethodHandlerAdapter，3.1之后默认注入RequestMappingHandlerAdapter