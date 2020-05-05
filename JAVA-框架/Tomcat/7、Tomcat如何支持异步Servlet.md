[TOC]

>Servlet 3.0 中引入的异步 Servlet。主要是在 Web 应用里启动一个单独的线程来执行这些比较耗时的请求，而 Tomcat 线程立即返回，不再等待 Web 应用将请求处理完，这样 Tomcat 线程可以立即被回收到线程池，用来响应其他请求，降低了系统的资源消耗，同时还能提高系统的吞吐量。

# 一、异步 Servlet 示例

```java

@WebServlet(urlPatterns = {"/async"}, asyncSupported = true)
public class AsyncServlet extends HttpServlet {

    //Web应用线程池，用来处理异步Servlet
    ExecutorService executor = Executors.newSingleThreadExecutor();

    public void service(HttpServletRequest req, HttpServletResponse resp) {
        //1. 调用startAsync或者异步上下文
        final AsyncContext ctx = req.startAsync();

       //用线程池来执行耗时操作
        executor.execute(new Runnable() {

            @Override
            public void run() {

                //在这里做耗时的操作
                try {
                    ctx.getResponse().getWriter().println("Handling Async Servlet");
                } catch (IOException e) {}

                //3. 异步Servlet处理完了调用异步上下文的complete方法
                ctx.complete();
            }

        });
    }
}
```

上面的代码有三个要点：

1. 通过注解的方式来注册 Servlet，除了 @WebServlet 注解，还需要加上asyncSupported=true的属性，表明当前的 Servlet 是一个异步 Servlet。

2. Web 应用程序需要调用 Request 对象的 startAsync 方法来拿到一个异步上下文 AsyncContext。这个上下文保存了请求和响应对象。

3. Web 应用需要开启一个新线程来处理耗时的操作，处理完成后需要调用 AsyncContext 的 complete 方法。目的是告诉 Tomcat，请求已经处理完成。

注意，虽然异步 Servlet 允许用更长的时间来处理请求，但是也有超时限制的，默认是 30 秒，如果 30 秒内请求还没处理完，Tomcat 会触发超时机制，向浏览器返回超时错误，如果这个时候你的 Web 应用再调用ctx.complete方法，会得到一个 IllegalStateException 异常。

# 二、异步 Servlet 原理

关键就是要弄清楚req.startAsync方法和ctx.complete方法都做了什么。

## startAsync 方法

startAsync 方法其实就是创建了一个异步上下文 AsyncContext 对象，AsyncContext 对象的作用是保存请求的中间信息，比如 Request 和 Response 对象等上下文信息。

你来思考一下为什么需要保存这些信息呢？

这是因为 Tomcat 的工作线程在request.startAsync调用之后，就直接结束回到线程池中了，线程本身不会保存任何信息。也就是说一个请求到服务端，执行到一半，你的 Web 应用正在处理，这个时候 Tomcat 的工作线程没了，这就需要有个缓存能够保存原始的 Request 和 Response 对象，而这个缓存就是 AsyncContext。

有了 AsyncContext，你的 Web 应用通过它拿到 Request 和 Response 对象，拿到 Request 对象后就可以读取请求信息，请求处理完了还需要通过 Response 对象将 HTTP 响应发送给浏览器。

除了创建 AsyncContext 对象，startAsync 还需要完成一个关键任务，那就是告诉 Tomcat 当前的 Servlet 处理方法返回时，不要把响应发到浏览器，因为这个时候，响应还没生成呢；并且不能把 Request 对象和 Response 对象销毁，因为后面 Web 应用还要用呢。

在 Tomcat 中，负责 flush 响应数据的是 CoyoteAdapter，它还会销毁 Request 对象和 Response 对象，因此需要通过某种机制通知 CoyoteAdapter，具体来说是通过下面这行代码：

```java
this.request.getCoyoteRequest().action(ActionCode.ASYNC_START, this);
```
你可以把它理解为一个 Callback，在这个 action 方法里设置了 Request 对象的状态，设置它为一个异步 Servlet 请求。

连接器是调用 CoyoteAdapter 的 service 方法来处理请求的，而 CoyoteAdapter 会调用容器的 service 方法，当容器的 service 方法返回时，CoyoteAdapter 判断当前的请求是不是异步 Servlet 请求，

如果是，就不会销毁 Request 和 Response 对象，也不会把响应信息发到浏览器。 CoyoteAdapter 的 service 方法:

```java

public void service(org.apache.coyote.Request req, org.apache.coyote.Response res) {
    
   //调用容器的service方法处理请求
    connector.getService().getContainer().getPipeline().
           getFirst().invoke(request, response);
   
   //如果是异步Servlet请求，仅仅设置一个标志，
   //否则说明是同步Servlet请求，就将响应数据刷到浏览器
    if (request.isAsync()) {
        async = true;
    } else {
        request.finishRequest();
        response.finishResponse();
    }
   
   //如果不是异步Servlet请求，就销毁Request对象和Response对象
    if (!async) {
        request.recycle();
        response.recycle();
    }
}
```

当 CoyoteAdapter 的 service 方法返回到 ProtocolHandler 组件时，

ProtocolHandler 判断返回值，如果当前请求是一个异步 Servlet 请求，它会把当前 Socket 的协议处理者 Processor 缓存起来，

将 SocketWrapper 对象和相应的 Processor 存到一个 Map 数据结构里。

```java
private final Map<S,Processor> connections = new ConcurrentHashMap<>();
```
之所以要缓存是因为这个请求接下来还要接着处理，还是由原来的 Processor 来处理，通过 SocketWrapper 就能从 Map 里找到相应的 Processor。

## complete 方法
当请求处理完成时，Web 应用调用ctx.complete方法。最重要的就是把响应数据发送到浏览器。

这件事情不能由 Web 应用线程来做，也就是说ctx.complete方法不能直接把响应数据发送到浏览器，

因为这件事情应该由 Tomcat 线程来做，但具体怎么做呢？


连接器中的 Endpoint 组件检测到有请求数据达到时，会创建一个 SocketProcessor 对象交给线程池去处理，因此 Endpoint 的通信处理和具体请求处理在两个线程里运行。

在异步 Servlet 的场景里，Web 应用通过调用ctx.complete方法时，也可以生成一个新的 SocketProcessor 任务类，交给线程池处理。对于异步 Servlet 请求来说，相应的 Socket 和协议处理组件 Processor 都被缓存起来了，并且这些对象都可以通过 Request 对象拿到。

```java

public void complete() {
    //检查状态合法性，我们先忽略这句
    check();
    
    //调用Request对象的action方法，其实就是通知连接器，这个异步请求处理完了
request.getCoyoteRequest().action(ActionCode.ASYNC_COMPLETE, null);
    
}
```

complete 方法调用了 Request 对象的 action 方法。而在 action 方法里，则是调用了 Processor 的 processSocketEvent 方法，并且传入了操作码 OPEN_READ。

```java

case ASYNC_COMPLETE: {
    clearDispatches();
    if (asyncStateMachine.asyncComplete()) {
        processSocketEvent(SocketEvent.OPEN_READ, true);
    }
    break;
}
```
继续看processSocketEvent 方法，它调用 SocketWrapper 的 processSocket 方法：

```java

protected void processSocketEvent(SocketEvent event, boolean dispatch) {
    SocketWrapperBase<?> socketWrapper = getSocketWrapper();
    if (socketWrapper != null) {
        socketWrapper.processSocket(event, dispatch);
    }
}
```

而 SocketWrapper 的 processSocket 方法会创建 SocketProcessor 任务类，并通过 Tomcat 线程池来处理：

```java
public boolean processSocket(SocketWrapperBase<S> socketWrapper,
        SocketEvent event, boolean dispatch) {
        
      if (socketWrapper == null) {
          return false;
      }
      
      SocketProcessorBase<S> sc = processorCache.pop();
      if (sc == null) {
          sc = createSocketProcessor(socketWrapper, event);
      } else {
          sc.reset(socketWrapper, event);
      }
      //线程池运行
      Executor executor = getExecutor();
      if (dispatch && executor != null) {
          executor.execute(sc);
      } else {
          sc.run();
      }
}
```

请你注意 createSocketProcessor 函数的第二个参数是 SocketEvent，这里我们传入的是 OPEN_READ。通过这个参数，我们就能控制 SocketProcessor 的行为，因为我们不需要再把请求发送到容器进行处理，只需要向浏览器端发送数据，并且重新在这个 Socket 上监听新的请求就行了。

最后我通过一张在帮你理解一下整个过程：

![d2d96b7450dff9735989005958fa13ae](7、Tomcat如何支持异步Servlet？.resources/890BD8E3-4A90-47DC-AA97-3AF5F98A4483.jpg)
