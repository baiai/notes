# Script资源加载的方式

script标签加载资源的方式有以下三种方式

```html
<script src='./tempCodeRunnerFile.js' async></script>
<script src='./tempCodeRunnerFile.js' defer></script>
<script src='./tempCodeRunnerFile.js'></script>
```

他们的区别如下：

![wfL82.png](https://segmentfault.com/img/bVWhRl?w=801&h=814)



# srcset响应式加载图片

响应式页面中，经常用到根据屏幕密度设置不同的图片，img标签的srcset属性就是用来设置不同屏幕密度下，加载不同的图片的。

```html
<img src="./image-128.jpg"
     srcset="./image-128.jpg 128w, image-256.jpg 256w, image-512.jpg 512w"
     sizes="(max-width: 360px) 340px, 128px"/>
```

srcset中的属性值表示的含义是当`img`元素的宽度规格为128的时候加载image-128.jpg这张图片，当`img`元素的宽度规格为256的时候加载image-256.jpg这张图片，当`img`元素的宽度规格为512的时候加载image-512.jpg这张图片。

这里的宽度规格就是`w`描述符的另外一种理解，其与sizes属性设定和屏幕密度密切相关。举个例子，假设屏幕密度是2的iPhone6手机，sizes属性计算值是128px，则此时`img`实际的宽度规格应该是128*2也就是256w，因此会加载256px.jpg这张图。



# data-*属性

data-*是HTML5的新特性，可以允许用户在标签中`存储自定义的信息`。

```html
<img src="javascript:;" data-origin="./src/image/1.jpg" id="tu"/>
```

自定义信息的读写可以通过JavaScript的`getAttribute()`和`dataset`。

```js
let img = document.getElementById('tu');
// 读
img.getAttribute('data-origin'); // ./src/image/1.jpg
img.dataset.origin; // ./src/image/1.jpg
// 写
img.setAttribute('data-origin', 'undefined')
img.dataset.origin = undefined
```

此外CSS也可以使用`attr`和`属性选择器`获取data-*

```css
img{
    background: url(attr(data-origin))
}
img[data-origin="./src/image/1.jpg"]{
	background-color:red;
}
```

