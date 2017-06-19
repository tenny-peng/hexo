---
layout: title
title: hessian入门使用
date: 2017-06-19 16:11:24
tags: [hessian]
categories: hessian
---

# 简介
Hessian是caucho公司开发的一种基于二进制RPC协议（Remote Procedure Call protocol）的轻量级远程调用框架。

在Java中使用Hessian：

服务器端：
* 包含Hessian.jar包
* 设计一个接口，用来给客户端调用
* 实现该接口的功能
* 配置web.xml，配好相应的servlet
* 由于使用二进制RPC协议传输数据，对象必须进行序列化，实现Serializable 接口
* 对于复杂对象可以使用Map的方法传递

客户端：
* 包含Hessian.jar包
* 具有和服务器端结构一样的接口。包括命名空间都最好一样
* 利用HessianProxyFactory调用远程接口

# 入门例子

到 http://hessian.caucho.com/ 下载hessian.jar包，我这里使用的是hessian-4.0.51.jar。

## 服务器端
1. Eclipse新建一个Web工程，命名为HseeionService。将hessian.jar包放入WEB-INF/lib中，并引入之；
2. 创建pojo类；
    ```
    package app.demo;

    import java.io.Serializable;

    public class User implements Serializable {

        private static final long serialVersionUID = 153519254199840035L;

        String userName = "soopy";

        String password = "showme";

        public User(String user, String pwd){
            this.userName = user;
            this.password = pwd;
        }

        public String getUserName(){
            return userName;
        }

        public String getPassword(){
            return password;
        }
    }
    ```

3. 创建接口
	```
    package app.demo;

    public interface BasicAPI {
        public void setGreeting(String greeting);
        public String hello();
        public User getUser();
    }
    ```

4. 实现接口
	```
    package app.demo;

    public class BasicService implements BasicAPI {

        private String _greeting = "Hello, world";

        @Override
        public void setGreeting(String greeting) {
            _greeting = greeting;
        }

        @Override
        public String hello() {
            return _greeting;
        }

        @Override
        public User getUser() {
            return new User("prance", "meshow");
        }

    }
    ```

5. 配置web.xml
    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xmlns="http://java.sun.com/xml/ns/javaee"
             xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                                 http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
             id="WebApp_ID"
             version="3.0">
      <servlet>
        <servlet-name>hello</servlet-name>
        <servlet-class>com.caucho.hessian.server.HessianServlet</servlet-class>
        <init-param>
          <param-name>service-class</param-name>
          <param-value>app.demo.BasicService</param-value>
        </init-param>
      </servlet>
      <servlet-mapping>
        <servlet-name>hello</servlet-name>
        <url-pattern>/hello</url-pattern>
      </servlet-mapping>
    </web-app>
    ```

6. 服务器测试test.jsp
    ```
    <%@ page import="com.caucho.hessian.client.HessianProxyFactory,app.demo.BasicAPI"%>
    <%@page language="java"%>
    <%
    HessianProxyFactory factory = new HessianProxyFactory();
    String url = ("http://" +request.getServerName() + ":" +request.getServerPort() +
    request.getContextPath() + "/hello");
    out.println(url);
    out.println("<br>");
    BasicAPI basic = (BasicAPI) factory.create(BasicAPI.class,url);
    out.println("Hello: " + basic.hello());
    out.println("<br>");
    out.println("Hello: " + basic.getUser() .getUserName() );
    out.println("<br>");
    out.println("Hello: " +basic.getUser().getPassword() );
    %>
    ```

7. 将HessianService部署到Tomcat等服务器上，访问 http://localhost:8080/HessianService/test.jsp ，浏览器结果：
    ```
    http://localhost:8080/HessianService/hello
    Hello: Hello, world
    Hello: prance
    Hello: meshow
    ```

## 客户端
1. 创建一个工程，命名为HessianClient，同样引入hessian.jar包；
2. 创建和服务端一样的pojo类；
3. 创建和服务端一样的接口；
4. 创建客户端
    ```
    package app.demo;

    import java.net.MalformedURLException;

    import com.caucho.hessian.client.HessianProxyFactory;

    public class BasicClient {

        public static void main(String[] args) throws MalformedURLException {
            String url ="http://127.0.0.1:8080/HessianService/hello";
            HessianProxyFactory factory = new HessianProxyFactory();
            BasicAPI basic = (BasicAPI) factory.create(BasicAPI.class, url);
            System.out.println("Hello:" + basic.hello());
            System.out.println("Hello:" + basic.getUser().getUserName());
            System.out.println("Hello:" + basic.getUser().getPassword());
            basic.setGreeting("HelloGreeting");
            System.out.println("Hello:" + basic.hello());
        }

    }
    ```

5. 运行客户端代码，控制台结果：
    ```
    Hello:Hello, world
    Hello:prance
    Hello:meshow
    Hello:HelloGreeting
    ```

## 说明
这里服务端和客户端都编写了同样的接口和基础类，只是为了演示简单例子。在实际使用中，应该是服务端将自己的接口及所需类打成jar包给客户端引入调用。