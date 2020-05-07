* `@GetMapping`用于将HTTP get请求映射到特定处理程序的方法注解。具体来说，`@GetMapping`是一个组合注解，是`@RequestMapping(method = RequestMethod.GET)`的缩写。

* `@PostMapping`用于将HTTP post请求映射到特定处理程序的方法注解。具体来说，@PostMapping是一个组合注解，是`@RequestMapping(method = RequestMethod.POST)`的缩写。



下面我们来看下@GetMapping的源码，可以对上面的两句释义给予充分的支撑。
```java
/**
 * Annotation for mapping HTTP {@code GET} requests onto specific handler
 * methods.
 *
 * <p>Specifically, {@code @GetMapping} is a <em>composed annotation</em> that
 * acts as a shortcut for {@code @RequestMapping(method = RequestMethod.GET)}.
 *
 *
 * @author Sam Brannen
 * @since 4.3
 * @see PostMapping
 * @see PutMapping
 * @see DeleteMapping
 * @see PatchMapping
 * @see RequestMapping
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@RequestMapping(method = RequestMethod.GET)
public @interface GetMapping {
 
	/**
	 * Alias for {@link RequestMapping#name}.
	 */
	@AliasFor(annotation = RequestMapping.class)
	String name() default "";
 
    ...
 
}
```

上面代码中，最关键的是 `@RequestMapping(method = RequestMethod.GET)`，
这行代码即说明`@GetMapping`就是`@RequestMapping`附加了请求方法。
同时，可以看到`@GetMapping`这个注解 是spring4.3版本引入，
同时引入的还有`@PostMapping、@PutMapping、@DeleteMapping和@PatchMapping`，一共5个注解。


所以，**一般情况下用`@RequestMapping(value = "/**.action",method = RequestMethod. XXXX)`即可。**
