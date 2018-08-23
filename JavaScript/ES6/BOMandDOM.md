###DOM
####获取节点
```javascript
document.getElementById();
document.getElementsByName();
document.getElementsByTagName();
document.getElementsByClassName();
document.querySelectorAll('tagName'); //document.querySelectorAll('p');
document.querySelectorAll('*');// 获取所有节点
```

####property
DOM节点就是一个js对象，符合对象的特性--可扩展性，所以有style、nodeName、className、nodeType等属性
####attribute
attribute是直接修改HTML标签的属性主要有getAttribute和setAttribute两种方法
```html
<div style='width:300px;height:300px;' data-name></div>
```
```javascript
divs = document.getElementsByTagName('div');
div = divs[0];
div.getAttribute('data-name');
div.setAttribute('data-name','juejin'); //
```
####DOM树操作
```javascript
//新增节点：
let p1 = document.createElement('p1');

//移动节点：
let p2 = document.getElementsById('p2');
p2.appendChild(p1); // 移动已有节点而非复制，appendChild是添加节点
p2.append(p1); // append是添加元素

//获取父节点：
let div = document.getElementById('div')[0];
parent = div.parentElement

//获取子节点：
let div = document.getElementById('div')[0];
children1 = div.childNodes // 包含注释，文本等节点
children2 = div.children;

//删除节点：
div.removeChild(children[0]);  // 只能一个一个的删除，可以用forEach
	
//事件绑定
obj.addEventListener('click',fn);
```

BOM
	navigator screen location history