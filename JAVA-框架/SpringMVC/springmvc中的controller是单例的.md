1. 如果你给controller中定义很多的属性，那么单例肯定会出现竞争访问了。  因此，只要controller中不定义属性，那么单例就是是安全的
2. `@controller`之前增加`@Scope("prototype")`，就可以改变单例模式为多例模式