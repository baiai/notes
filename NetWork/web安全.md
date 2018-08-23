# XSS攻击

XSS的分类：存储型、反射型、DOM型

### 存储型

比如说，一个博客的留言板功能，如何我们在提交留言的时候写入脚本攻击语言。如果后台没有做处理的话，就会保存到数据库，以后只要访问这个页面就会触发这段脚本攻击。正是因为攻击的脚本代码存储在了数据库里，所以叫做存储型XSS攻击。比如在未经处理的留言板写入以下字段：

```
<script>alert('XSS攻击成功')</script>
```

### 反射型

反射型性对于存储型的区别在于反射型不会保存到数据库里，从而只是一次性的攻击方式。比如下面的语句：

```
http://example.com/?param=<script>alert('这是一个XSS攻击')</script>
```

### DOM型

DOM型是反射型的一种特例，是指通过DOM元素注入一些攻击代码，比如：

```
document.getElementById('x').innerHTML = "<img src=0 onerror = alter('DOM攻击');";
```

### 防御

1 编码：对用户的输入数据进行HTML Entity编码

2 过滤：移出用户上传的DOM属性，如onerror，移出用户上传的style节点、script节点、iframe节点

3 校正：避免直接对HTML Entity编码，使用DOM Parse转换，校正不配对的DOM标签

参考：https://wizardforcel.gitbooks.io/xss-naxienian/content/7.html



# CSRF攻击

CSRF是通过以合法用户的名义对网站发起攻击

攻击过程如下：

　　webA是存在CSRF漏洞的网站，webB是攻击者构建的网站，userC是webA的合法用户。

　　1 C打开浏览器，输入用户名和密码访问webA

　　2 在用户信息通过后，webA产生Cookie并返回给浏览器，此时用户登录成功，可以正常发送请求webA　　

　　3 用户为退出webA之前访问webB

　　4 webB接收到用户请求后返回一些攻击性代码。并发送请求访问第三方webA

　　5 浏览器在接收到这些攻击性代码后，根据webB的请求，在用户不知情的情况下携带Cookie向webA发送请求。webA根据Cookie信息以及C的权限处理请求，导致webB的恶意代码被执行。

防御：

　　1 验证HTTP Referer字段，不过Referer可以被修改并不是很安全。

　　2 在请求地址中添加token并验证，有效但是繁琐，每次都要添加token验证信息。

　　3 HTTP头中并自定义属性并验证。