---
layout: title
title: spring加载多个xml遇到的问题
date: 2017-04-27 09:36:25
tags: [spring]
categories: spring
---

## 问题出现
之前学习spring整合redis，spring整合activemq，单独测试没有问题。后来想把他们一起部署启动，结果报错
```
Could not resolve placeholder 'redis.maxTotal' in string value "${redis.maxTotal}"
```

## 查找原因
查了一会找到了原因。因为我的spring-redis.xml和spring-activemq.xml都写了一个
```
<context:property-placeholder location="classpath:conf/xxx.properties" />
```
而Spring容器采用反射扫描的发现机制，在探测到Spring容器中有一个org.springframework.beans.factory.config.PropertyPlaceholderConfigurer的Bean就会停止对剩余PropertyPlaceholderConfigurer的扫描。

而<context:property-placeholder/>这个基于命名空间的配置，其实内部就是创建一个PropertyPlaceholderConfigurer Bean而已。换句话说，即Spring容器仅允许**最多定义一个**PropertyPlaceholderConfigurer(或<context:property-placeholder/>)，其余的会被Spring**忽略**掉（其实Spring如果提供一个警告就好了）。

这样的话，其实就只加载了第一个properties文件，后面的并没有加载，自然也就找不到'redis.maxTotal'了。

## 尝试解决
按照网上的办法，去掉每个xml单独的context:property-placeholder，再写一个xml文件一次性加载所有资源文件，并引入之前单独的所有xml文件。

先将web文件的
```
context-param>
	<param-name>contextConfigLocation</param-name>
	<param-value>
        classpath:spring-activemq.xml,
        classpath:spring-redis.xml
	</param-value>
</context-param>
```
改为
```
context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>
			classpath:applicationContext.xml
		</param-value>
	</context-param>
```
编写这个applicationContext.xml如下：
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:p="http://www.springframework.org/schema/p"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
						http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
						http://www.springframework.org/schema/context
						http://www.springframework.org/schema/context/spring-context.xsd">

    <context:property-placeholder location="classpath:conf/*.properties" />

	<import resource="activemq/spring-activemq.xml" />

	<import resource="redis/spring-redis.xml" />

</beans>
```
这样部署启动应该就可以了。

## 新的问题
按道理应该启动成功，不过我这里又遇到另一个问题
```
Cannot convert value of type [org.springframework.data.redis.connection.jedis.JedisConnectionFactory] to required type [javax.jms.ConnectionFactory] for property 'connectionFactory': no matching editors or conversion strategy found
```

## 再查原因
查看自己的spring-redis.xml
```
<bean id="connectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
p:hostName="${redis.host}" p:port="${redis.port}" p:password="${redis.pass}" c:poolConfig-ref="poolConfig">
</bean>

<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">  
    <property name="connectionFactory" ref="connectionFactory" />
    <property name="keySerializer" ref="keySerializer" />
</bean>
```
并没有需要'javax.jms.ConnectionFactory'，根据问题在网上搜索，在一篇博客看到了'redis也有个bean叫connectionFactory'的字眼。于是想到自己应该也是bean name重复了。查看spring-activemq.xml
```
<bean id="connectionFactory" class="org.springframework.jms.connection.SingleConnectionFactory">  
    <property name="targetConnectionFactory" ref="pooledConnectionFactory"/>  
</bean>

<!-- Spring提供的JMS工具类，它可以进行消息发送、接收等 -->  
<bean id="jmsTemplate" class="org.springframework.jms.core.JmsTemplate">  
    <property name="connectionFactory" ref="connectionFactory"/>  
    <property name="defaultDestinationName" value="${activemq.queue.name}"/>  
</bean>
```

## 解决问题
问题就很明显了，spring-redis.xml和spring-activemq.xml都有connectionFactory这个bean。于是修改了spring-redis.xml中的bean name
```
<bean id="redisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
p:hostName="${redis.host}" p:port="${redis.port}" p:password="${redis.pass}" c:poolConfig-ref="poolConfig">
</bean>

<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">  
    <property name="connectionFactory" ref="redisConnectionFactory" />
    <property name="keySerializer" ref="keySerializer" />
</bean>
```
再次部署启动，OK。

参考博客：http://www.iteye.com/topic/1131688  
　　　　　http://blog.csdn.net/AlbertFly/article/details/51503079
