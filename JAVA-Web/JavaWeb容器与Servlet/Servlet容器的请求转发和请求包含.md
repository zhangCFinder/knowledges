[TOC]
# 1. Servlet 容器

编程中的容器我们可以理解为程序运行时需要的环境，那么Tomcat 就是Servlet 的运行环境，就是一个Servlet 容器。Servlet 容器的作用是负责处理客户请求，当Servlet 容器获取到用户请求后，调用某个Servlet，并把Servlet 的执行结果返回给用户。

**Servlet 容器的工作流程：**

* 当用户请求某个资源时，Servlet容器使用ServletRequest对象将用户的请求信息封装起来，然后调用 `java Servlet API` 中定义的Servlet的生命周期方法，完成Servlet的运行。

* Servlet容器将Servlet执行后需要返回用户的结果封装到 `ServletResponse` 对象中，最后由`Servlet` 容器发送给客户，完成对客户的一次服务过程。

* 每一个Servlet都会执行 `init()、service()、destory()` 三个方法，在启动时调用一次`init() `方法对参数进行初始化，在该Servlet 生存期间每当收到对其的请求时都会调用`Service()` 方法对请求进行处理，当容器销毁时自动调用 `destory() `方法对Servlet 进行销毁。

# 2. 请求转发和请求包含
正是因为因为Servlet 中的`service()` 方法由Servlet 容器调用，所以一个 Servlet 的对象是无法调用另一个 Servlet 的方法的，但是在实际项目中，对于客户端请求做出的响应可能会复杂，需要多个Servlet 来协作完成，这就需要请求转发和请求包含技术了。但是，要注意，无论是请求转发还是请求包含，都是表示由多个Servlet 共同处理同一个请求。
## （1）请求转发定义：
Servlet（源组件）先对客户请求做一些预处理操作（一般是对响应头进行处理），然后把请求转发给其他Servlet（目标组件）来完成包括生成响应结果在内的后续操作。

实现方法：`request.getRequestDispatcher(“接收请求的Servlet 路径”). forward(request,response)`

## （2）请求包含定义：

Servlet（源组件）把其他Servlet（目标组件）生成的响应结果包含到自身的响应结果中。

实现方式：`request.getRequestDispatcher(“接收请求的Servlet 路径”). include(request,response)`

**注意：** 

1. 当Servlet 源组件调用 RequestDispatcher 的 forward 或 include 方法时，都要把当前的 ServletRequest 对象和ServletResponse 对象作为参数传给 forward 或 include 方法，这就使得源组件和目标组件共享同一个ServletRequest 对象和ServletResponse 对象，就实现了**多个Servlet 协同处理同一个请求**。

2. RequestDispatcher 接口中定义了两个方法:：`forward()` 方法和 `include()` 方法,它们分别用于将请求转发到 RequestDispatcher 对象封装的资源和将 RequestDispatcher 对象封装的资源作为当前响应内容的一部分包含进来.

