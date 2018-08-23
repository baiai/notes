####  Promise的含义

​	Promise是异步编程的一种解决方案

#### 常规用法

```javascript
let promise = new Promise(function(resolve, reject) {
	console.log('Promise');
	resolve();
});

promise.then(function() {
  console.log('resolved.');
}).catch(function(){
	console.log('rejected');
});

console.log('Hi!');
输出结果是：
	Promise
	Hi!
	Resolved.
嵌套：
	const p1 = new Promise(function (resolve, reject) {
	});
	const p2 = new Promise(function (resolve, reject) {
	  // ...
	  resolve(p1);
	});
```

这时p1的状态就会传递给p2，也就是说，p1的状态决定了p2的状态。

如果p1的状态是pending，那么p2的回调函数就会等待p1的状态改变；如果p1的状态已经是resolved或者rejected，那么p2的回调函数将会立刻执行。

#### then and catch 分别捕获resolved状态和rejected状态

#### finally(function(){}) 

​	不管最终promise的状态是什么，都会执行finally指定的函数

#### all(function(){})

```javascript
// 生成一个Promise对象的数组
const promises = [2, 3, 5, 7, 11, 13].map(function (id) {
	return getJSON('/post/' + id + ".json");
});

Promise.all(promises).then(function (posts) {
  // ...
}).catch(function(reason){
  // ...
});

```

只有当all()里面的所有promise对象的状态是resolved的时候，all()的状态
才是resolved，否则就是rejected。