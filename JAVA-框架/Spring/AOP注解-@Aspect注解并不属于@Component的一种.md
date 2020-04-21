

* 也就是一个类单纯如果只添加了`@Aspect`注解，那么它并不能被`context:component-scan`标签扫描到。 

* 想要被扫描到的话，需要追加一个`@Component` 注解
