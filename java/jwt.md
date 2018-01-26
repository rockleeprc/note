
# JWT
* RFC7519标准
* 数字签名保证安全
* 可以通过URL、POST、HTTP Header发送，数据量小
* payload包含用户信息，避免查询数据库

# JWT 结构
* header
* payload
* signature

## header

头部用于描述关于该JWT的最基本的信息，例如其token类型以及签名所用的算法等

	{
	  "typ": "JWT",
	  "alg": "HS256"
	}

Base64编码，之后的字符串就成了JWT的Header（头部）：

	eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9

## payload

* 包含了claim， Claim是一些实体（通常指的用户）的状态和额外的元数据

* 三种claim：
	* Reserved claims: 这些claim是JWT预先定义的，在JWT中并不会强制使用它们，而是推荐使用，常用的有 
		* iss（签发者）
		* exp（过期时间戳）
		* sub（面向的用户）
		* aud（接收方）
		* iat（签发时间）
	* Public claims：根据需要定义自己的字段，注意应该避免冲突
	* Private claims：这些是自定义的字段，可以用来在双方之间交换信息

	
	{
	    "sub": "1",
	    "iss": "http://localhost:8000/auth/login",
	    "iat": 1451888119,
	    "exp": 1454516119,
	    "nbf": 1451888119,
	    "jti": "37c107e4609ddbcc9c096ea5ee76c667"
	}
	
	sub: 该JWT所面向的用户
	iss: 该JWT的签发者
	iat(issued at): 在什么时候签发的token
	exp(expires): token什么时候过期
	nbf(not before)：token在此时间之前不能被接收处理
	jti：JWT ID为web token提供唯一标识

将上面的JSON对象进行base64编码可以得到下面的字符串：

	eyJzdWIiOiIxIiwiaXNzIjoiaHR0cDpcL1wvbG9jYWxob3N0OjgwMDFcL2F1dGhcL2xvZ2luIiwiaWF0IjoxNDUxODg4MTE5LCJleHAiOjE0NTQ1MTYxMTksIm5iZiI6MTQ1MTg4ODExOSwianRpIjoiMzdjMTA3ZTQ2MDlkZGJjYzljMDk2ZWE1ZWU3NmM2NjcifQ

这个字符串称作JWT的payload

## signature

将payload、header base64后的字符串用.拼接，head.payload

	eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIiwiaXNzIjoiaHR0cDpcL1wvbG9jYWxob3N0OjgwMDFcL2F1dGhcL2xvZ2luIiwiaWF0IjoxNDUxODg4MTE5LCJleHAiOjE0NTQ1MTYxMTksIm5iZiI6MTQ1MTg4ODExOSwianRpIjoiMzdjMTA3ZTQ2MDlkZGJjYzljMDk2ZWE1ZWU3NmM2NjcifQ

将上面拼接完的字符串用HS256算法进行加密。在加密的时候，我们还需要提供一个密钥（secret）:

	HMACSHA256(
	    base64UrlEncode(header) + "." +
	    base64UrlEncode(payload),
	    secret
	)

得到加密后的内容，这一部分又叫做签名。

	wyoQ95RjAyQ2FF3aj8EvCSaUmeP0KUqcCJDENNfnaT4

JWT格式的输出是以.分隔的三段Base64编码，与SAML等基于XML的标准相比，JWT在HTTP和HTML环境中更容易传递。下列的JWT展示了一个完整的JWT格式，它拼接了之前的Header， Payload以及秘钥签名

	eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiIxIiwiaXNzIjoiaHR0cDpcL1wvbG9jYWxob3N0OjgwMDFcL2F1dGhcL2xvZ2luIiwiaWF0IjoxNDUxODg4MTE5LCJleHAiOjE0NTQ1MTYxMTksIm5iZiI6MTQ1MTg4ODExOSwianRpIjoiMzdjMTA3ZTQ2MDlkZGJjYzljMDk2ZWE1ZWU3NmM2NjcifQ.wyoQ95RjAyQ2FF3aj8EvCSaUmeP0KUqcCJDENNfnaT4


* header.payload.signature

# JWT使用

当用户希望访问一个受保护的路由或者资源的时候，通常应该在Authorization头部使用Bearer模式添加JWT，其内容看起来是下面这样：

Authorization: Bearer <token>


# 注

* 跨站请求伪造Cross-site request forgery（简称CSRF, 读作 [sea-surf]）