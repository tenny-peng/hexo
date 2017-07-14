---
title: rsa之js加密java解密
date: 2017-07-14 8:37:54
tags: [rsa]
categories: rsa
---

# 需求
有一个需求，登录密码不能明文传输，需要前端加密，后端接收后再解密。这里使用rsa加密，在此记录简单的demo。

# RSA简介
RSA算法是一种非对称密码算法，所谓非对称，就是指该算法需要一对密钥，使用其中一个加密，则需要用另一个才能解密。使用时，先生成一对密钥——公钥+私钥，公钥对外公开，供加密使用，私钥则需妥善保存，用于解密已加密数据。

# 使用
在这个demo中，最终实现前端加密密码，后端解密密码。

1. 引入依赖包，，除spring mvc等常用包外，这里需要commons-codec
```
	<!-- https://mvnrepository.com/artifact/commons-codec/commons-codec -->
	<dependency>
	    <groupId>commons-codec</groupId>
	    <artifactId>commons-codec</artifactId>
	    <version>1.10</version>
	</dependency>
```

2. 编写RSAUtils，工具类，实现生成密钥，加密解密等方法
```
  package com.rsa.utils;

  import java.security.Key;
  import java.security.KeyFactory;
  import java.security.KeyPair;
  import java.security.KeyPairGenerator;
  import java.security.NoSuchAlgorithmException;
  import java.security.spec.PKCS8EncodedKeySpec;
  import java.security.spec.X509EncodedKeySpec;
  import java.util.HashMap;
  import java.util.Map;

  import javax.crypto.Cipher;

  import org.apache.commons.codec.binary.Base64;

  public class RSAUtils {

  	public static final String KEY_ALGORITHM = "RSA";

  	public static final String PUBLIC_KEY = "RSAPublicKey";

  	public static final String PRIVATE_KEY = "RSAPrivateKey";

  	public static byte[] decryptBASE64(String key) {
          return Base64.decodeBase64(key);
      }

      public static String encryptBASE64(byte[] bytes) {
          return Base64.encodeBase64String(bytes);
      }

  	/**
  	 * 生成密钥
  	 * @return
  	 * @throws Exception
  	 */
  	public static Map<String, Key> generateKey(){
  		Map<String, Key> keyMap = new HashMap<String,Key>(2);

  		try {
  			KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance(KEY_ALGORITHM);
  			keyPairGenerator.initialize(1024);
  			KeyPair keyPair = keyPairGenerator.generateKeyPair();
  		 	keyMap.put(PUBLIC_KEY, keyPair.getPublic());// 公钥
  	        keyMap.put(PRIVATE_KEY, keyPair.getPrivate());// 私钥
  		}catch (NoSuchAlgorithmException e) {
  			e.printStackTrace();
  		}

  		return keyMap;
  	}

      /**
       * 取得公钥
       * @param keyMap
       * @return
       * @throws Exception
       */
      public static String getPublicKey(Map<String, Key> keyMap) {
          Key key = keyMap.get(PUBLIC_KEY);
          return encryptBASE64(key.getEncoded());
      }

      /**
       * 取得私钥
       * @param keyMap
       * @return
       * @throws Exception
       */
      public static String getPrivateKey(Map<String, Key> keyMap) {
          Key key = (Key) keyMap.get(PRIVATE_KEY);
          return encryptBASE64(key.getEncoded());
      }

      /**
       * 解密<br>
       * @param data
       * @param key
       * @return
       * @throws Exception
       */
      public static String decrypt(String data, String key) {
          return decrypt(decryptBASE64(data), key);
      }

  	/**
  	 * 解密
  	 * @param data
  	 * @param key
  	 * @return
  	 * @throws Exception
  	 */
  	public static String decrypt(byte[] data, String key){
  		String result = null;

  		byte[] keyBytes = decryptBASE64(key);
  		try {
  			PKCS8EncodedKeySpec pkcs8KeySpec = new PKCS8EncodedKeySpec(keyBytes);
  			KeyFactory keyFactory= KeyFactory.getInstance(KEY_ALGORITHM);
  			Key privateKey = keyFactory.generatePrivate(pkcs8KeySpec);

  			Cipher cipher = Cipher.getInstance(keyFactory.getAlgorithm());
  			cipher.init(Cipher.DECRYPT_MODE, privateKey);
  			byte[] decryptData = cipher.doFinal(data);
  			result = new String(decryptData);

  		} catch (Exception e) {
  			e.printStackTrace();
  		}

  		return result;
  	}

  	/**
       * 加密
       * @param data
       * @param key
       * @return
       * @throws Exception
       */
      public static byte[] encrypt(String data, String key) {
      	byte[] result = null;

          byte[] keyBytes = decryptBASE64(key);
          try{
          	X509EncodedKeySpec x509KeySpec = new X509EncodedKeySpec(keyBytes);
          	KeyFactory keyFactory = KeyFactory.getInstance(KEY_ALGORITHM);
          	Key publicKey = keyFactory.generatePublic(x509KeySpec);

          	Cipher cipher = Cipher.getInstance(keyFactory.getAlgorithm());
          	cipher.init(Cipher.ENCRYPT_MODE, publicKey);
          	result =  cipher.doFinal(data.getBytes());
          } catch(Exception e){
          	e.printStackTrace();
          }

          return result;
      }

  }
```

3. 后端测试
```
  import java.security.Key;
  import java.util.Map;

  import com.rsa.utils.RSAUtils;

  public class Test {
  	public static void main(String[] args) {
  		Map<String, Key> keyMap = RSAUtils.generateKey();
  		String publicKey = RSAUtils.getPublicKey(keyMap);
  	    String privateKey = RSAUtils.getPrivateKey(keyMap);
          System.err.println("公钥: \n" + publicKey);
          System.err.println("私钥： \n" + privateKey);

          String data = "123456";
          byte[] decryptDate = RSAUtils.encrypt(data, publicKey);
          String decryptData = RSAUtils.decrypt(decryptDate, privateKey);
          System.out.println("解密数据： \n" + decryptData);
  	}
  }
```

4. 后端测试结果，验证后端工具类正确。
```
公钥:
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCV38Y7LUhAjrE2zdXlPhtvGdqeiZXKLQwkjEyO3xH/jIUy+6NgXAgam99HtMpUQATbvsVT1oRAIkVFkqiDgIZDxLrlKdESoRLyaThGsUlmRLkUTT1Ofuzc8l8FEYGonb9A0uHAmqsXrwSZjNG0M812cbvt4Hjts6TgbzoYLsLHDQIDAQAB
私钥：
MIICdwIBADANBgkqhkiG9w0BAQEFAASCAmEwggJdAgEAAoGBAJXfxjstSECOsTbN1eU+G28Z2p6JlcotDCSMTI7fEf+MhTL7o2BcCBqb30e0ylRABNu+xVPWhEAiRUWSqIOAhkPEuuUp0RKhEvJpOEaxSWZEuRRNPU5+7NzyXwURgaidv0DS4cCaqxevBJmM0bQzzXZxu+3geO2zpOBvOhguwscNAgMBAAECgYBFgvAx6iKkronK3VTjahbXRKp89Vsf1hzXpqqraRKz77ynlMaFnqmzja/VViixQq/+K1DiPZBBHqP6TLcTpryeZBM5OeDoA3eYawOFuM/lyUe6H91mCygkM2FDWQGxEiXkgpyui786MtyWeI53xHjSWOPa5zQYWuH9fHG3mivOAQJBAMjN2d1AFPNe/La73DHi7RcA6CK4eZmoKZQqawYZ2oWd1MMOFRl8dkmneOfFaGDUZ2mShXzFZk7oKFdTJG1FB2ECQQC/Ehc3MJd/awZZ4qejxTe2LetaQnuMclqreZK3ClXTwFe+Sk/ETwXFQHsfblPRN1NFScAoA3Eq6GBhnp9MoVstAkAI/7iowqttsK8QnWCj17CaXE8K50uDyFZ8rl33ewcg/86+Iw5tAvfmGxw+/sjLthkgURGsYshP9vV/3FkAkJxhAkEAqWBwFAyPP/Sv/J5f3V3GtUifibPFsgrtNXTgCkKvMrcfESDu9SbYBrPScVpsEtrohlOKc+4ZM+ArEF58+IFRQQJBAMKBOO2cUWDwmanz26KEUAxPNi5PsvQcZIvreBbRdNsC3zRsNJwaF3WVe9K/Yn53jkrxhj5UC8sk6/A6JsNehkk=
解密数据：
123456
```

5. 前端加密，依赖jsencrypt.js。这里登录时才获取公钥，实际可以在访问此页面就获取公钥，并且密码改变时实时加密
```
function login() {
    $.ajax({
		type: "post",
		url: "/rsa-demo/user/getkey",
		data: {},
		dataType: 'json',
		success: function(result) {
			var publicKey = result.key;
			console.log(publicKey);

			var username = $('#username').val();
			var password = $('#password').val();

			var encrypt = new JSEncrypt();
		    encrypt.setPublicKey(publicKey);
		    var encrypted = encrypt.encrypt(password);

			$.ajax({
				type: "post",
				url: "/rsa-demo/user/login",
				data: {'username': username, 'password': encrypted},
				dataType: 'json',
				success: function(result) {
					alert(result.message);
				}
			});
		}
	});
}
```

6. 前端ajax请求对应的后端控制器，返回公钥和验证密码
```
  package com.rsa.controller;

  import java.security.Key;
  import java.util.HashMap;
  import java.util.Map;

  import javax.servlet.http.HttpServletRequest;
  import javax.servlet.http.HttpServletResponse;
  import javax.servlet.http.HttpSession;

  import org.springframework.web.bind.annotation.RequestMapping;
  import org.springframework.web.bind.annotation.RequestMethod;
  import org.springframework.web.bind.annotation.RestController;

  import com.rsa.utils.RSAUtils;

  @RestController
  @RequestMapping("/user")
  public class UserController {
  	@RequestMapping(value = "/login", method = RequestMethod.POST)
  	public Map<String, Object> login(HttpServletRequest requset, HttpServletResponse response){
  		Map<String, Object> result = new HashMap<String, Object>();

  		HttpSession session = requset.getSession();
  		// 没有密钥非法登录
  		if(session.getAttribute(RSAUtils.PUBLIC_KEY) == null || session.getAttribute(RSAUtils.PRIVATE_KEY) == null){
  			result.put("message", "登录状态异常，请刷新重试");
  			return result;
  		}

  		String username = requset.getParameter("username");
  		String password = requset.getParameter("password");

  		String privateKey = (String) session.getAttribute(RSAUtils.PRIVATE_KEY);
  		long startTime = System.currentTimeMillis();
  		String decryptData = RSAUtils.decrypt(password, privateKey);
  		long endTime = System.currentTimeMillis();
  		System.err.println("解密所用时间：" + (endTime - startTime) + "ms");

  		// 模拟数据验证
  		if(username.equals("test") && decryptData.equals("123456")){
  			// 登录成功删除密钥
  			session.removeAttribute(RSAUtils.PUBLIC_KEY);
  			session.removeAttribute(RSAUtils.PRIVATE_KEY);
  			result.put("message", "登录成功");
  		}else{
  			result.put("message", "用户名或密码错误");
  		}

  		return result;
  	};

  	@RequestMapping(value = "/getkey")
  	public Map<String, Object> getKey(HttpServletRequest requset){
  		Map<String, Object> result = new HashMap<String, Object>();

  		HttpSession session = requset.getSession();
  		long startTime = System.currentTimeMillis();
  		// 访问当前页面设置密钥
  		Map<String, Key> keyMap = RSAUtils.generateKey();
  		String publicKey = RSAUtils.getPublicKey(keyMap);
  		String privateKey = RSAUtils.getPrivateKey(keyMap);
  		long endTime = System.currentTimeMillis();
  		System.err.println("生成key所用时间：" + (endTime - startTime) + "ms");
  		session.setAttribute(RSAUtils.PUBLIC_KEY, publicKey);
  		session.setAttribute(RSAUtils.PRIVATE_KEY, privateKey);

  		result.put("key", publicKey);
  		return result;
  	}
  }
```

# 小结
按照正常流程，用户访问到登陆页面，后端给前端发送一个公钥，在用户输入密码登录时，需要先对密码加密，再传输。后端接受到已加密数据后，使用此公钥对应的私钥解密数据。后续验证数据库密码等省略。

需要注意的是，为保证安全，避免重放攻击和直接post请求，在登录前生成新的密钥对，登录后销毁该密钥对。在用户访问登录页面时，生成密钥对放入session，验证登录成功后，销毁该密钥对。因为每次生成新的公钥，所以重复发送同一请求无法解密通过。直接使用post请求也会因为密钥过期或不存在不予通过。

另外，RSA还可以生成数字签名。上述加密数据是用公钥加密，私钥解密。而数字签名用私钥加密，公钥解密，用于验证消息发送来源及消息完整性。因为私钥只有我（服务端）知道，假如客户端收到数字签名后用公钥无法解开，说明此服务端不可信。

参考链接：http://blog.csdn.net/cut001/article/details/53189645
　　　　　https://baike.baidu.com/item/RSA%E7%AE%97%E6%B3%95
