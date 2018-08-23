CORS需要浏览器和服务器同时支持，浏览器将CORS分为两类：简单请求和非简单请求。

只要满足以下两个请求的都称为简单请求：

```
（1）请求的方法是：HEAD、POST、GET中的一种。
（2）HTTP的头部信息不超出以下几种字段：
	Accept
	Accept-Language
	Content-Language
	Last-Event-ID
	Content-Type的值是：
		application/x-www-form-urlencoded
		multipart/form-data
		text/plain中的一种
```

无法满足以上条件的就是非简单请求。非简单请求会在正式请求之前发送一个OPTIONS请求。

###CORS简单请求跨域的流程：

在浏览器发起跨域访问请求的时候，会在请求头中加入Origin这个字段表示请求来自哪个源。如果服务器允许Origin中指定的源访问的话，就会在响应头重加入Access-Control-Allow-Origin这个字段。如果浏览器没有检测到这个字段就会认为跨域请求失败，从而被XMLHttpRequest的onerror检测到。

```http
// 请求头
GET /cors HTTP/1.1
Origin: http://localhost:8080
Host: api.alice.com
Accept-Language: en-US
Connection: keep-alive
User-Agent: Mozilla/5.0...
// 响应头
Access-Control-Allow-Origin:http://localhost:8080
Access-Control-Allow-Credentials: true
Access-Control-Expose-Headers: FooBar
Content-Type: text/html; charset=utf-8
```

其中响应头中的各个属性解释如下：

（1）Access-Control-Allow-Origin：

该字段是`必须`的。它的值要么是请求时Origin字段的值，要么是一个*，表示接受任意域名的请求。

（2）Access-Control-Allow-Credentials：

该字段可选。它的值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在CORS请求之中。设为true，即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器。这个值也只能设为true，如果服务器不要浏览器发送Cookie，删除该字段即可。

此外除了后台的设置之外，要发送还必须在前端设置XMLHTTPRequest对象上添加一个widthCredentials的属性，这样才能发送Cookie和Http认证信息。同时Access-Control-Allow-Origin的值`不能`设置为*，必须是和原请求网页一样的地址。

（3）Access-Control-Expose-Headers

该字段可选。CORS请求时，XMLHttpRequest对象的getResponseHeader()方法只能拿到6个基本字段：Cache-Control、Content-Language、Content-Type、Expires、Last-Modified、Pragma。如果想拿到其他字段，就必须在Access-Control-Expose-Headers里面指定。上面的例子指定，getResponseHeader('FooBar')可以返回FooBar字段的值。

###CORS非简单请求跨域的流程：

非简单请求其实就是在简单请求中间添加了一个“预检”过程，使用的是OPTIONS的请求方法，请求头中有以下属性：

```http
// 请求头
OPTIONS /cors HTTP/1.1
Origin: http://api.bob.com // 必须的
Access-Control-Request-Method: PUT // 必须的
Access-Control-Request-Headers: X-Custom-Header
// 响应头
Access-Control-Allow-Methods: GET, POST, PUT
Access-Control-Allow-Headers: X-Custom-Header
Access-Control-Allow-Credentials: true
Access-Control-Max-Age: 1728000
```

（1）Access-Control-Request-Method: 

表明浏览器的CORS请求会用到的方法，请求的方法都必须响应头的Access-Control-Request-Method的值中

（2）Access-Control-Allow-Headers: 

如果请求头中有这个字段的话，那么这个字段就是必须的。同样请求头的包含的值是响应头中包含的值的子集

（3）Access-Control-Allow-Credentials: 

与简单请求中一样

（4）Access-Control-Max-Age：

指定本次“预检”请求的有效期，单位是s，例子中的有效期是20天，表示允许缓存该回应20天。

### 与JSONP的比较

相较而言，CROS是大势所趋，但是不代表JSONP就一无是处了，因为CROS还存在兼容性的问题。那么不兼容CROS的浏览器就只能使用JSONP的方式来进行跨域了。

参考资料：

http://www.ruanyifeng.com/blog/2016/04/cors.html



