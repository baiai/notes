为何会出现SSR，SSR主要解决了什么问题？

在当前的前端环境下，使用MV*框架开发成为主流，但是并不意味这这些框架就是完美无瑕的。比如Vue，Vue是客户端应用程序的框架，所以他在客户端会有大量的计算，从而影响性能。那么如果我们把这部分计算放在后台服务器做会不会更好呢？答案是会，而SSR就是解决这个问题的，如何让Vue中的大量的计算在服务器端运行。



# 基本用法

以下是Vue官方给出的例子，给出SSR的基本用法：

```javascript
const Vue = require('vue');
const server = require('express')();
const fs = require('fs');
const renderer = require('vue-server-renderer').createRenderer({
  // 引入的HTML文件模板，支持插值
  template: fs.readFileSync('./index.template.html', 'utf-8')
});

server.get('*', function (req, res) {
  const app = new Vue({
    data: {
      url: req.url
    },
    template: `<div>URL: {{ url }}</div>`
  });
  context = {
    title: 'BaiAi'
  };
  // 对Vue实例进行渲染，渲染成html文件
  renderer.renderToString(app, context, (err, html) => {
    if (err) {
      res.status(500).end('Internal Server Error');
      return;
    }
    console.log(html);
    res.end(html);
  });
});
server.listen(8080);
```

```html
<!-- index.template.html -->
<!DOCTYPE html>
<html lang="en">
  <head><title>{{ title }}</title></head>
  <body>
    <!--vue-ssr-outlet-->
    <!--这里存放的就是Vue实例中的template的内容-->
  </body>
</html>
```

如上代码所示，Node服务器监听8080端口，一旦有访问就会返回一个在后台已经渲染好的html文件，这样就可以减少Vue在前端的计算时间。从而提升效率。



