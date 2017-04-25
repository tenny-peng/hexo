---
layout: title
title: spring的MessageSource
date: 2017-04-25 14:17:54
tags: [spring, MessageSource]
categories: MessageSource
---

## spring-message.xml
配置messageSource路径。
```
<bean id="messageSource"
    class="org.springframework.context.support.ReloadableResourceBundleMessageSource">
    <property name="basenames">
        <list>
            <value>classpath:message/message</value>
        </list>
    </property>
    <property name="defaultEncoding" value="UTF-8"/>
</bean>

<bean id="messageHelper" class="com.news.common.utils.MessageHelper">
    <property name="messageSource" ref="messageSource"/>
</bean>
```

## MessageHelper
信息工具类，通过spring注入。核心是上面配置的messageSource，可针对不同地区/国家加载不同的信息文件(message.properties)。
```
import java.util.Locale;

import org.springframework.context.MessageSource;

public class MessageHelper {

	private static MessageSource messageSource;

	public static void setMessageSource(MessageSource messageSource) {
		MessageHelper.messageSource = messageSource;
	}

	public static String getMessage(String code) {
		return getMessage(code, null);
	}

	public static String getMessage(String code, Object[] args) {
		return messageSource.getMessage(code, args, Locale.getDefault());
	}

	public static String getMessage(String code, Object[] args, Locale locale) {
		return messageSource.getMessage(code, args, locale);
	}

}
```

## message_zh_CN.properties & message_en_US.properties
message_zh_CN，针对中文语言环境。
```
#用户提示
u0001=用户名或密码不能为空！
u0002=用户名"{0}"已存在！
u0003=用户名或密码错误！
```
message_en_US.properties，针对英文(国际)语言环境。
```
#user tips
u0001=username or password cannot be null!
u0002=username "{0}" is already exist！
u0003=username or password is error!
```

## TestMeaage.java
使用默认本地语言环境(中文)和指定语言环境(英文)分别测试。
```
import java.util.Locale;

import org.springframework.context.support.ClassPathXmlApplicationContext;

import com.news.common.utils.MessageHelper;

/**
 * TODO
 *
 * @author tenny.peng
 */
public class TestMessage {

	public static void main(String[] args) {
		ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("spring-message.xml");

		System.out.println(MessageHelper.getMessage("u0001"));
		System.out.println(MessageHelper.getMessage("u0002", new String[] { "tenny" }));
		System.out.println(MessageHelper.getMessage("u0003"));

		System.out.println(MessageHelper.getMessage("u0001", null, Locale.US));
		System.out.println(MessageHelper.getMessage("u0002", new String[] { "tenny" }, Locale.US));
		System.out.println(MessageHelper.getMessage("u0003", null, Locale.US));
	}

}
```
输出
```
用户名或密码不能为空！
用户名"tenny"已存在！
用户名或密码错误！
username or password cannot be null!
username "tenny" is already exist!
username or password is error!
```

## 方法说明
messageSource.getMessage(code, args, locale)有三个参数：

1. 消息的编码值；

2. 对应消息的参数，没有就传null；

3. java.util.Locale参数。locale为null时，根据使用者的语言环境决定Locale，从而决定要加载的message文件。  
    上面的测试先后加载了messages_zh_CN.properties和message_en_US.properties资源文件。  
    这其中还有一个控制点在JVM，JVM会根据当前操作系统的语言环境进行相应处理，我们可以通过在JVM启动参数中追加“-Duser.lang ge=zh_TW”来设定当前JVM语言类型，通过JVM级的设定，也可以实现自动切换所使用的资源文件类型。
    所以这里面的控制语言的方式有三种：从最低层的操作系统的Locale设定，到更上一层的JVM的Locale设定，再到程序一级的Locale设定。

参考博客：http://lixiaorong223.blog.163.com/blog/static/4401162920110106305224/
