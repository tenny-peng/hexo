---
layout: title
title: HttpClient模拟post请求
date: 2017-06-30 14:24:01
tags: [HttpClient]
categoriest: HttpClient
---

因项目需要，使用HttpClient模拟请求，故了解学习，记录在此。
# 简介
HttpClient 是 Apache Jakarta Common 下的子项目，可以用来提供高效的、最新的、功能丰富的支持 HTTP 协议的客户端编程工具包，并且它支持 HTTP 协议最新的版本和建议。

# 使用
使用HttpClient发送请求、接收响应很简单，一般需要如下几步即可：
1. 创建CloseableHttpClient对象。
2. 创建请求方法的实例，并指定请求URL。如果需要发送GET请求，创建HttpGet对象；如果需要发送POST请求，创建HttpPost对象。
3. 如果需要发送请求参数，可调用setEntity(HttpEntity entity)方法来设置请求参数。setParams方法已过时（4.4.1版本）。
4. 调用HttpGet、HttpPost对象的setHeader(String name, String value)方法设置header信息，或者调用setHeaders(Header[] headers)设置一组header信息。
5. 调用CloseableHttpClient对象的execute(HttpUriRequest request)发送请求，该方法返回一个CloseableHttpResponse。
6. 调用HttpResponse的getEntity()方法可获取HttpEntity对象，该对象包装了服务器的响应内容。程序可通过该对象获取服务器的响应内容；调用CloseableHttpResponse的getAllHeaders()、getHeaders(String name)等方法可获取服务器的响应头。
7. 释放连接。无论执行方法是否成功，都必须释放连接。

# 示例
先下载最新的jar包或者maven引入
```
<!-- https://mvnrepository.com/artifact/org.apache.httpcomponents/httpclient -->
<dependency>
  <groupId>org.apache.httpcomponents</groupId>
  <artifactId>httpclient</artifactId>
  <version>4.5.3</version>
</dependency>
```

用HttpClient编写自己的请求工具类
```
package com.demo.utils;

import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.Map.Entry;

import org.apache.http.HttpEntity;
import org.apache.http.NameValuePair;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.entity.UrlEncodedFormEntity;
import org.apache.http.client.methods.CloseableHttpResponse;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.impl.client.CloseableHttpClient;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.message.BasicNameValuePair;
import org.apache.http.util.EntityUtils;

public class HttpClientUtils {

	/**
	 * 通过httpClient发送post请求
	 * @param url 请求地址，非空
	 * @param map 请求参数
	 * @param encoding 编码格式，为空则默认UTF-8
	 */
	public static String sendPost(String url, Map<String,String> map, String encoding){
		String result = "";
		if(url == null){
			result = "url不能为空";
			return result;
		}
		if(encoding == null || encoding.isEmpty()){
			encoding = "UTF-8";
		}

		// 创建httpClient对象
        CloseableHttpClient httpclient = HttpClients.createDefault();  
        // 创建post方式请求对象  
        HttpPost httpPost = new HttpPost(url);

        // 装填参数
        List<NameValuePair> nvps = new ArrayList<NameValuePair>();  
        if(map != null){
        	for(Entry<String, String> entry : map.entrySet()){
        		nvps.add(new BasicNameValuePair(entry.getKey(), entry.getValue()));
        	}
        }

		try {
			// 参数转码
			UrlEncodedFormEntity uefEntity = new UrlEncodedFormEntity(nvps, encoding);

	        //设置参数到请求对象中
	        httpPost.setEntity(uefEntity);

	        System.out.println("--------------------------------------");
	        System.out.println("请求地址： " + httpPost.getURI());  
	        System.out.println("请求参数： " + nvps.toString());
	        System.out.println("--------------------------------------");

	        //设置header信息
	        //指定报文头【Content-type】、【User-Agent】  
	        httpPost.setHeader("Content-type", "application/x-www-form-urlencoded");  
	        httpPost.setHeader("User-Agent", "Mozilla/4.0 (compatible; MSIE 5.0; Windows NT; DigExt)");

	        //执行请求操作，并拿到结果（同步阻塞）
	        CloseableHttpResponse response = httpclient.execute(httpPost);  
	        HttpEntity entity = response.getEntity();  
	        if (entity != null) {
	        	//按指定编码转换结果实体为String类型
	        	result = EntityUtils.toString(entity, encoding);
	        	System.out.println("返回内容：");  
	            System.out.println("--------------------------------------");  
	            System.out.println(result);  
	            System.out.println("--------------------------------------");  
	        }
	        EntityUtils.consume(entity);
	        response.close();

		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		} catch (ClientProtocolException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}  finally{
			try {
				httpclient.close();
			} catch (IOException e) {
				e.printStackTrace();
			}
		}

		return result;
	}
}
```

编写测试
```
package com.demo;

import java.util.HashMap;
import java.util.Map;

import com.demo.utils.HttpClientUtils;

public class Main {
	public static void main(String[] args) {

		String url = "http://php.weather.sina.com.cn/iframe/index/w_cl.php";
		Map<String, String> map = new HashMap<String, String>();
		map.put("city", "深圳");
		map.put("charset", "utf-8");
		HttpClientUtils.sendPost(url, map, null);

	}

}
```

控制台查看结果
```
--------------------------------------
请求地址： http://php.weather.sina.com.cn/iframe/index/w_cl.php
请求参数： [charset=utf-8, city=深圳]
--------------------------------------
返回内容：
--------------------------------------
(function(){var w=[];w['深圳']=[{s1:'阵雨',s2:'多云',f1:'zhenyu',f2:'duoyun',t1:'32',t2:'27',p1:'≤3',p2:'≤3',d1:'无持续风向',d2:'无持续风向'}];var add={now:'2017-06-30 14:45:59',time:'1498805159',update:'北京时间06月30日12:10更新',error:'0',total:'1'};window.SWther={w:w,add:add};})();//0

--------------------------------------
```

# 附录
顺便记录一下java直接发送http请求的例子。

请求工具类
```
package com.demo.utils;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.URL;
import java.net.URLConnection;

public class HttpRequest {
	/**
	 * 向指定URL发送GET方法的请求
	 *
	 * @param url 发送请求的URL
	 * @param param 请求参数，请求参数应该是 name1=value1&name2=value2 的形式。
	 * @return URL 所代表远程资源的响应结果
	 */
	public static String sendGet(String url, String param) {
		String result = "";
		BufferedReader in = null;
		try {
			String urlNameString = url + "?" + param;
			URL realUrl = new URL(urlNameString);
			// 打开和URL之间的连接
			URLConnection connection = realUrl.openConnection();
			// 设置通用的请求属性
			connection.setRequestProperty("accept", "*/*");
			connection.setRequestProperty("connection", "Keep-Alive");
			connection.setRequestProperty("user-agent", "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
			// 建立实际的连接
			connection.connect();
			// 定义 BufferedReader输入流来读取URL的响应
			in = new BufferedReader(new InputStreamReader(connection.getInputStream()));
			String line;
			while ((line = in.readLine()) != null) {
				result += line;
			}
		} catch (Exception e) {
			System.out.println("发送GET请求出现异常！" + e);
			e.printStackTrace();
		}
		// 使用finally块来关闭输入流
		finally {
			try {
				if (in != null) {
					in.close();
				}
			} catch (Exception e2) {
				e2.printStackTrace();
			}
		}
		return result;
	}

	/**
	 * 向指定 URL 发送POST方法的请求
	 *
	 * @param url 发送请求的 URL
	 * @param param 请求参数，请求参数应该是 name1=value1&name2=value2 的形式。
	 * @return 所代表远程资源的响应结果
	 */
	public static String sendPost(String url, String param) {
		PrintWriter out = null;
		BufferedReader in = null;
		String result = "";
		try {
			URL realUrl = new URL(url);
			// 打开和URL之间的连接
			URLConnection conn = realUrl.openConnection();
			// 设置通用的请求属性
			conn.setRequestProperty("accept", "*/*");
			conn.setRequestProperty("connection", "Keep-Alive");
			conn.setRequestProperty("user-agent", "Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1;SV1)");
			// 发送POST请求必须设置如下两行
			conn.setDoOutput(true);
			conn.setDoInput(true);
			// 获取URLConnection对象对应的输出流
			out = new PrintWriter(conn.getOutputStream());
			// 发送请求参数
			out.print(param);
			// flush输出流的缓冲
			out.flush();
			// 定义BufferedReader输入流来读取URL的响应
			in = new BufferedReader(new InputStreamReader(conn.getInputStream()));
			String line;
			while ((line = in.readLine()) != null) {
				result += line;
			}
		} catch (Exception e) {
			System.out.println("发送 POST 请求出现异常！" + e);
			e.printStackTrace();
		}
		// 使用finally块来关闭输出流、输入流
		finally {
			try {
				if (out != null) {
					out.close();
				}
				if (in != null) {
					in.close();
				}
			} catch (IOException ex) {
				ex.printStackTrace();
			}
		}
		return result;
	}
}
```

测试类
```
package com.demo;

import com.demo.utils.HttpRequest;

public class Main {
	public static void main(String[] args) {

    String url = "http://php.weather.sina.com.cn/iframe/index/w_cl.php";
		String result = HttpRequest.sendPost(url, "city=深圳&charset=utf-8");
		System.out.println(result);

	}

}
```

控制台输出
```
(function(){var w=[];w['深圳']=[{s1:'阵雨',s2:'多云',f1:'zhenyu',f2:'duoyun',t1:'32',t2:'27',p1:'≤3',p2:'≤3',d1:'无持续风向',d2:'无持续风向'}];var add={now:'2017-06-30 14:58:03',time:'1498805883',update:'北京时间06月30日12:10更新',error:'0',total:'1'};window.SWther={w:w,add:add};})();//0
```

# 参考
http://www.cnblogs.com/zhuawang/archive/2012/12/08/2809380.html
http://blog.csdn.net/xiaoxian8023/article/details/49863967
