---
layout: title
title: spring整合redis
date: 2017-04-21 10:02:06
tags: [spring, redis]
categories: redis
---

# 概念简介：

* Redis：一款开源的Key-Value数据库。
* Jedis：Redis官方推出的一款面向Java的客户端，提供了很多接口供Java语言调用。
* Spring Data Redis：SDR是Spring官方推出，可以算是Spring框架集成Redis操作的一个子框架，封装了Redis的很多命令，可以很方便的使用Spring操作Redis数据库。

这三个究竟有什么区别呢？可以简单的这么理解，Redis是用ANSI C写的一个基于内存的Key-Value数据库，而Jedis是Redis官方推出的面向Java的Client，提供了很多接口和方法，可以让Java操作使用Redis，而Spring Data Redis是对Jedis进行了封装，集成了Jedis的一些命令和方法，可以与Spring整合。在后面的配置文件（spring-redis.xml）中可以看到，Spring是通过Jedis类来初始化connectionFactory的。

# spring整合redis

1. maven添加依赖配置
```
  <dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
  	<version>1.4.1.RELEASE</version>
  </dependency>

  <dependency>
  	<groupId>redis.clients</groupId>
  	<artifactId>jedis</artifactId>
  	<version>2.6.1</version>
  </dependency>
```

2. redis.properties
```
# Redis settings
redis.host=localhost
redis.port=6379
redis.pass=tenny
redis.maxTotal=200
redis.maxIdle=50
redis.minIdle=300
redis.maxWaitMillis=1000
redis.testOnBorrow=true
```

3. spring-redis.xml
```
  <?xml version="1.0" encoding="UTF-8"?>  
  <beans xmlns="http://www.springframework.org/schema/beans"
  	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  	xmlns:context="http://www.springframework.org/schema/context"
  	xmlns:mvc="http://www.springframework.org/schema/mvc"
  	xmlns:p="http://www.springframework.org/schema/p"
  	xmlns:c="http://www.springframework.org/schema/c"
  	xsi:schemaLocation="http://www.springframework.org/schema/beans  
                          http://www.springframework.org/schema/beans/spring-beans.xsd  
                          http://www.springframework.org/schema/context  
                          http://www.springframework.org/schema/context/spring-context.xsd  
                          http://www.springframework.org/schema/mvc  
                          http://www.springframework.org/schema/mvc/spring-mvc.xsd">

      <context:property-placeholder location="classpath:redis.properties" />  

      <bean id="poolConfig" class="redis.clients.jedis.JedisPoolConfig">  
          <property name="maxTotal" value="${redis.maxTotal}" />
          <property name="maxIdle" value="${redis.maxIdle}" />
          <property name="minIdle" value="${redis.minIdle}" />
          <property name="maxWaitMillis" value="${redis.maxWaitMillis}" />
          <property name="testOnBorrow" value="${redis.testOnBorrow}" />
      </bean>

      <bean id="connectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory"
  		p:hostName="${redis.host}" p:port="${redis.port}" p:password="${redis.pass}" c:poolConfig-ref="poolConfig">
      </bean>  

      <bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">  
          <property name="connectionFactory" ref="connectionFactory" />
      </bean>

      <bean id="cacheManager" class="org.springframework.data.redis.cache.RedisCacheManager" c:template-ref="redisTemplate">
          <property name="usePrefix" value="true" />
          <property name="cacheNames">
              <set>
                  <value>t</value>
                  <value>c</value>
              </set>
          </property>
      </bean>

  </beans>  
```

4. 定义User实体
```
  import java.io.Serializable;
  import java.util.Date;

  public class User implements Serializable {

  	private static final long serialVersionUID = -6683628971480535063L;

  	private Integer id;

  	private String username;

  	private String password;

  	private Integer type;

  	private Date createTime;

  	public Integer getId() {
  		return id;
  	}

  	public void setId(Integer id) {
  		this.id = id;
  	}

  	public String getUsername() {
  		return username;
  	}

  	public void setUsername(String username) {
  		this.username = username == null ? null : username.trim();
  	}

  	public String getPassword() {
  		return password;
  	}

  	public void setPassword(String password) {
  		this.password = password == null ? null : password.trim();
  	}

  	public Integer getType() {
  		return type;
  	}

  	public void setType(Integer type) {
  		this.type = type;
  	}

  	public Date getCreateTime() {
  		return createTime;
  	}

  	public void setCreateTime(Date createTime) {
  		this.createTime = createTime;
  	}

  }
```

5. 测试
```
  import java.util.Date;
  import org.springframework.cache.Cache;
  import org.springframework.cache.Cache.ValueWrapper;
  import org.springframework.context.support.ClassPathXmlApplicationContext;
  import org.springframework.data.redis.cache.RedisCacheManager;
  import com.news.pojo.User;

  public class TestRedis {

  	@SuppressWarnings("resource")
  	public static void main(String[] args) {
  		ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring-redis.xml");
  		RedisCacheManager cacheManager = (RedisCacheManager) context.getBean("cacheManager");
  		System.out.println("cacheNames: " + cacheManager.getCacheNames());
  		Cache cacheT = cacheManager.getCache("t");
  		Cache cacheC = cacheManager.getCache("c");

  		User user1 = new User();
  		user1.setId(1);
  		user1.setUsername("tenny");
  		user1.setPassword("admin");
  		user1.setType(1);
  		user1.setCreateTime(new Date());
  		User user2 = new User();
  		user2.setId(2);
  		user2.setUsername("panda");
  		user2.setPassword("xiaobai");
  		user2.setType(2);
  		user2.setCreateTime(new Date());

  		System.out.println("put two user into cacheT...");
  		cacheT.put("user1", user1);
  		cacheT.put("user2", user2);
  		System.out.println("put name and age into cacheC...");
  		cacheC.put("name", "tenny");
  		cacheC.put("age", 25);

  		System.out.println("get two user from cacheT");
  		User value1 = cacheT.get("user1", User.class);
  		System.out.println(value1.toString());
  		ValueWrapper value2 = cacheT.get("user2");
  		System.out.println(value2.get());

  		System.out.println("get two user from cacheC");
  		ValueWrapper value3 = cacheC.get("name");
  		System.out.println(value3.get());
  		ValueWrapper value4 = cacheC.get("age");
  		System.out.println(value4.get());

  	}
  }
```

6. 测试结果
```
  cacheNames: [t, c]
  put two user into cacheT...
  put name and age into cacheC...
  get two user from cacheT
  com.news.pojo.User@2ea227af
  com.news.pojo.User@4386f16
  get two field from cacheC
  tenny
  25
```

7. 直接使用redisTemplate
```

  @SuppressWarnings("unchecked")
  RedisTemplate<String, User> redisTemplate = (RedisTemplate<String, User>) context.getBean("redisTemplate");
  System.out.println("put two user into redisTemplate...");
  redisTemplate.opsForHash().put("user", "user1", user1);
  redisTemplate.opsForHash().put("user", "user2", user2);
  System.out.println("gut two user from redisTemplate...");
  User redisUser1 = (User) redisTemplate.opsForHash().get("user", "user1");
  System.out.println(redisUser1);
  User redisUser2 = (User) redisTemplate.opsForHash().get("user", "user2");
  System.out.println(redisUser2);

  @SuppressWarnings("unchecked")
  RedisTemplate<String, String> redisTemplate2 = (RedisTemplate<String, String>) context.getBean("redisTemplate");
  System.out.println("put color list into redisTemplate2...");
  redisTemplate2.opsForList().leftPush("color", "blue");
  redisTemplate2.opsForList().leftPush("color", "red");
  redisTemplate2.opsForList().rightPush("color", "yellow");
  System.out.println("gut color list from redisTemplate2...");
  List<String> colorList = redisTemplate2.opsForList().range("color", 0, -1);
  System.out.println(colorList);
```

8. redisTemplate测试结果
```
put two user into redisTemplate...
gut two user from redisTemplate...
com.news.pojo.User@4313f5bc
com.news.pojo.User@7f010382
put color list into redisTemplate2...
gut color list from redisTemplate2...
[red, blue, yellow]
```
