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

# Spring整合Hessian

## 服务端项目
1. 新建一个Maven项目，编辑pom.xml
    ```
    <project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.tenny</groupId>
    <artifactId>SpringHessianService</artifactId>
    <packaging>war</packaging>
    <version>0.0.1-SNAPSHOT</version>
    <name>SpringHessianService Maven Webapp</name>
    <url>http://maven.apache.org</url>
    <dependencies>
        <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>
        
        <dependency>
            <groupId>com.caucho</groupId>
            <artifactId>hessian</artifactId>
            <version>4.0.7</version>
        </dependency>
        
    </dependencies>
    <build>
        <finalName>SpringHessianService</finalName>
        <plugins>  
            <plugin>  
                <groupId>org.apache.maven.plugins</groupId>  
                <artifactId>maven-compiler-plugin</artifactId> 
                <version>3.1</version>
                <configuration>  
                    <source>1.8</source>  
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>  
            </plugin>  
        </plugins>
    </build>
    </project>
    ```

2. 编辑web.xml
    ```
    <?xml version="1.0" encoding="UTF-8"?>  
    <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
            xmlns="http://java.sun.com/xml/ns/javaee"  
            xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
                                http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"  
            version="3.0">

    <display-name>Spring Hessian Service</display-name>
    
    <servlet>
            <servlet-name>HessianServlet</servlet-name>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
            <init-param>
                <param-name>contextConfigLocation</param-name>
                <param-value>classpath:hessian-server.xml</param-value>
            </init-param>
            <load-on-startup>1</load-on-startup>
        </servlet>
        <servlet-mapping>
            <servlet-name>HessianServlet</servlet-name>
            <url-pattern>/</url-pattern>
        </servlet-mapping>
    
    </web-app>
    ```

3. 编辑hessian-server.xml
    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
        xmlns:mvc="http://www.springframework.org/schema/mvc"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/mvc 
                            http://www.springframework.org/schema/mvc/spring-mvc.xsd
                            http://www.springframework.org/schema/beans 
                            http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context 
                            http://www.springframework.org/schema/context/spring-context.xsd">

        <context:component-scan base-package="com.tenny" />

        <bean name="/hello" class="org.springframework.remoting.caucho.HessianServiceExporter">
            <property name="service" ref="helloImpl" />
            <property name="serviceInterface" value="com.tenny.IHello" />
        </bean>

    </beans>
    ```

4. 接口代码
    ```
    package com.tenny;

    /**
    * @author tenny
    *
    */
    public interface IHello {
        
        public String hello(String name);

    }
    ```

5. 接口实现代码
    ```
    package com.tenny;

    import org.springframework.stereotype.Service;

    /**
    * @author tenny
    *
    */
    @Service
    public class HelloImpl implements IHello {

        @Override
        public String hello(String name) {
            String result = "hello " + name;
            System.out.println("service result: " + result);
            return result;
        }

    }
    ```

## 客户端项目
1. 新建Maven工程，编辑pom.xml
    ```
    <project xmlns="http://maven.apache.org/POM/4.0.0" 
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/maven-v4_0_0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.tenny</groupId>
    <artifactId>SpringHessianClient</artifactId>
    <packaging>war</packaging>
    <version>0.0.1-SNAPSHOT</version>
    <name>SpringHessianClient Maven Webapp</name>
    <url>http://maven.apache.org</url>
    <dependencies>
        <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
        <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>4.3.16.RELEASE</version>
        </dependency>
        
        <dependency>
            <groupId>com.caucho</groupId>
            <artifactId>hessian</artifactId>
            <version>4.0.7</version>
        </dependency>
        
    </dependencies>
    <build>
        <finalName>SpringHessianClient</finalName>
        <plugins>  
            <plugin>  
                <groupId>org.apache.maven.plugins</groupId>  
                <artifactId>maven-compiler-plugin</artifactId> 
                <version>3.1</version>
                <configuration>  
                    <source>1.8</source>  
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>  
            </plugin>  
        </plugins>
    </build>
    </project>
    ```

2. 编辑web.xml
    ```
    <?xml version="1.0" encoding="UTF-8"?>  
    <web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
            xmlns="http://java.sun.com/xml/ns/javaee"  
            xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
                                http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"  
            version="3.0">
        
    <display-name>Spring Hessian Client</display-name>
    
    <servlet>
            <servlet-name>HessianClient</servlet-name>
            <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
            <init-param>
                <param-name>contextConfigLocation</param-name>
                <param-value>classpath:hessian-client.xml</param-value>
            </init-param>
            <load-on-startup>1</load-on-startup>
        </servlet>
        <servlet-mapping>
            <servlet-name>HessianClient</servlet-name>
            <url-pattern>/</url-pattern>
        </servlet-mapping>
    
    </web-app>
    ```

3. 编辑hessian-client.xml
    ```
    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
        xmlns:mvc="http://www.springframework.org/schema/mvc"
        xmlns:context="http://www.springframework.org/schema/context"
        xsi:schemaLocation="http://www.springframework.org/schema/mvc 
                            http://www.springframework.org/schema/mvc/spring-mvc.xsd
                            http://www.springframework.org/schema/beans 
                            http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context 
                            http://www.springframework.org/schema/context/spring-context.xsd">

        <mvc:annotation-driven  />

        <context:component-scan base-package="com.tenny" />

        <bean id="hello" class="org.springframework.remoting.caucho.HessianProxyFactoryBean">
            <property name="serviceUrl" value="http://localhost:8080/SpringHessianService/hello"></property>
            <property name="serviceInterface" value="com.tenny.IHello"></property>
        </bean>

    </beans>
    ```

4. 接口代码（实际项目中引入服务端的接口jar包）
    ```
    package com.tenny;

    /**
    * @author tenny
    *
    */
    public interface IHello {
        
        public String hello(String name);

    }
    ```

5. Controller代码
    ```
    package com.tenny;

    import org.springframework.beans.factory.annotation.Autowired;
    import org.springframework.web.bind.annotation.RequestMapping;
    import org.springframework.web.bind.annotation.RequestMethod;
    import org.springframework.web.bind.annotation.RestController;

    /**
    * @author tenny
    *
    */
    @RestController
    @RequestMapping("/hello")
    public class HelloController {

        @Autowired
        private IHello helloHess;
        
        @RequestMapping(value = "/sayHello", method=RequestMethod.GET)
        public String sayHello(){
            String result =  helloHess.hello("tenny");
            System.out.println("client result: " + result);
            return result;
        }
        
    }
    ```

## 运行
分别启动服务端（端口为8080）和客户端（端口为8081）项目，浏览器地址访问   http://localhost:8081/SpringHessianClient/hello/sayHello 就能看到结果了。
其中页面显示`hello tenny`，服务端控制台打印`service result: hello tenny`，客户端控制台打印`client result: hello tenny`。