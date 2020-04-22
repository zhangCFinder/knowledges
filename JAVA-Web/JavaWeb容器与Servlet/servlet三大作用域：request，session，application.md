[TOC]
## 1. request
request是表示一个请求，只要发出一个请求就会创建一个request，它的作用域：仅在当前请求中有效。

用处：常用于服务器间同一请求不同页面之间的参数传递，常应用于表单的控件值传递。

方法：request.setAttribute(); request.getAttribute(); request.removeAttribute(); request.getParameter().

## 2. session
服务器会为每个会话创建一个session对象，所以session中的数据可供当前会话中所有servlet共享。

会话：用户打开浏览器会话开始，直到关闭浏览器会话才会结束。一次会话期间只会创建一个session对象。     

用处：常用于web开发中的登陆验证界面（当用户登录成功后浏览器分配其一个session键值对）。

方法：session.setAttribute(); session.getAttribute(); session.removeAttribute();

获得session对象方法：

在Servlet中：HttpSession session = request.getSession();
由于session属于jsp九大内置对象之一，当然可以直接使用。例如：<%session.serAttribute("name","admin")%>。  
session被销毁时间

1. session超时;

2. 客户端关闭后，再也访问不到和该客户端对应的session了，它会在超时之后被销毁;

3. 调用session. invalidate();

备注： session是服务器端对象，保存在服务器端。并且服务器可以将创建session后产生的sessionid通过一个cookie返回给客户端，以便下次验证。（session底层依赖于cookie）

## 3. Application（ServletContext）
作用范围：所有的用户都可以取得此信息，此信息在整个服务器上被保留。Application属性范围值，只要设置一次，则所有的网页窗口都可以取得数据。ServletContext在服务器启动时创建，在服务器关闭时销毁，一个JavaWeb应用只创建一个ServletContext对象，所有的客户端在访问服务器时都共享同一个ServletContext对象;ServletContext对象一般用于在多个客户端间共享数据时使用;

获取Application对象方法（Servlet中）：  
```java
 ServletContext app01 = this.getServletContext();
 app01.setAttribute("name", "kaixuan");    //设置一个值进去
       
 ServletContext app02 = this.getServletContext();
 app02.getAttribute("name");    //获取键值对  

```

ServletContext同属于JSP九大内置对象之一，故可以直接使用

备注：服务器只会创建一个ServletContext 对象，所以app01就是app02，通过app01设置的值当然可以通过app02获取。