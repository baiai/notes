#### 作用：Generator函数是一种异步编程解决方案

#### 用法：
形式上，Generator 函数是一个普通函数，但是有两个特征。

一是，function关键字与函数名之间有一个星号；

二是，函数体内部使用yield表达式，定义不同的内部状态例如：

```javascript
		function* helloWorldGenerator() {
		  yield 'hello';
		  yield 'world';
		  return 'ending';
		}
		var hw = helloWorldGenerator();
```
它内部有两个yield表达式（hello和world），即该函数有三个状态：hello，world 和 return 语句Generator 函数的调用方法与普通函数一样，也是在函数名后面加上一对圆括号。不同的是，调用 Generator 函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象，也就是上一章介绍的遍历器对象。下一步，必须调用遍历器对象的next方法，使得指针移向下一个状态。也就是说，每次调用next方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个yield表达式（或return语句）为止。换言之，Generator 函数是分段执行的，yield表达式是暂停执行的标记，而next方法可以恢复执行。例如：

```javascript
		hw.next()
		// { value: 'hello', done: false }
		hw.next()
		// { value: 'world', done: false }
		hw.next()
		// { value: 'ending', done: true }
		hw.next()
		// { value: undefined, done: true }
```
yield在另一个表达式中必须放在圆括号内，yield本身没有返回值，或者返回的是undefined当next()带参数true是就会把指针重新指到-1，下一次调用next()会调用0，相当于重新遍历Generator如果next()带的其他的参数

```javascript
		function* foo(x) {
		  var y = 2 * (yield (x + 1));
		  var z = yield (y / 3);
		  return (x + y + z);
		}
	
		var a = foo(5);
		a.next() // Object{value:6, done:false}
		a.next() // Object{value:NaN, done:false}
		a.next() // Object{value:NaN, done:true}
	
		var b = foo(5);
		b.next() // { value:6, done:false }
		b.next(12) // { value:8, done:false }
		b.next(13) // { value:42, done:true }
```
上面代码第一次调用b的next方法时，返回x+1的值6；

第二次调用next方法，将上一次yield表达式的值设为12，因此y等于24，返回y / 3的值8；

第三次调用next方法，将上一次yield表达式的值设为13，因此z等于13，这时x等于5，y等于24，所以return语句的值等于42。由于next方法的参数表示上一个yield表达式的返回值，所以在第一次使用next方法时，传递参数是无效的。V8 引擎直接忽略第一次使用next方法时的参数，只有从第二次使用next方法开始，参数才是有效的。

```javascript
	Generator.prototype.throw()
		Generator函数返回的遍历器对象都有一个throw方法，可以在函数体外抛出错误
		然后再Generator函数体内捕获
			var g = function* () {
			  try {
			    yield;
			  } catch (e) {
			    console.log('内部捕获', e);
			  }
			};
	
			var i = g();
			i.next();
	
			try {
			  i.throw('a');
			  i.throw('b');
			} catch (e) {
			  console.log('外部捕获', e);
			}
			// 内部捕获 a
			// 外部捕获 b
```

上面代码中，遍历器对象i连续抛出两个错误。第一个错误被 Generator 函数体内的catch语句捕获。i第二次抛出错误，由于 Generator 函数内部的catch语句已经执行过了，不会再捕捉到这个错误了，所以这个错误就被抛出了 Generator 函数体，被函数体外的catch语句捕获。

如果 Generator 函数内部没有部署try...catch代码块，那么throw方法抛出的错误，将被外部try...catch代码块捕获。

如果 Generator 函数内部和外部，都没有部署try...catch代码块，那么程序将报错，直接中断执行。
`*`throw方法被捕获以后，会附带执行下一条yield表达式。也就是说，会附带执行一次next方法。

Generator 函数体外抛出的错误，可以在函数体内捕获；反过来，Generator 函数体内抛出的错误，也可以被函数体外的catch捕获。


```javascript
				function* foo() {
				  var x = yield 3;
				  var y = x.toUpperCase();
				  yield y;
				}
	
				var it = foo();
	
				it.next(); // { value:3, done:false }
	
				try {
				  it.next(42);
				} catch (err) {
				  console.log(err);
				}
```

上面代码中，第二个next方法向函数体内传入一个参数 42，数值是没有toUpperCase方法的，所以会抛出一个 TypeError 错误，被函数体外的catch捕获。

一旦 Generator 执行过程中抛出错误，且没有被内部捕获，就不会再执行下去了。如果此后还调用next方法，将返回一个value属性等于undefined、done属性等于true的对象，即 JavaScript 引擎认为这个 Generator 已经运行结束了。

```javascript
				function* g() {
				  yield 1;
				  console.log('throwing an exception');
				  throw new Error('generator broke!');
				  yield 2;
				  yield 3;
				}
	
				function log(generator) {
				  var v;
				  console.log('starting generator');
				  try {
				    v = generator.next();
				    console.log('第一次运行next方法', v);
				  } catch (err) {
				    console.log('捕捉错误', v);
				  }
				  try {
				    v = generator.next();
				    console.log('第二次运行next方法', v);
				  } catch (err) {
				    console.log('捕捉错误', v);
				  }
				  try {
				    v = generator.next();
				    console.log('第三次运行next方法', v);
				  } catch (err) {
				    console.log('捕捉错误', v);
				  }
				  console.log('caller done');
				}
	
				log(g());
				// starting generator
				// 第一次运行next方法 { value: 1, done: false }
				// throwing an exception
				// 捕捉错误 { value: 1, done: false }
				// 第三次运行next方法 { value: undefined, done: true }
				// caller done
```
上面代码一共三次运行next方法，第二次运行的时候会抛出错误，然后第三次运行的时候，Generator 函数就已经结束了，不再执行下去了。

Generator.prototype.return()Generator 函数返回的遍历器对象，还有一个return方法，可以返回给定的值，并且终结遍历 Generator 函数。

```javascript
			function* gen() {
			  yield 1;
			  yield 2;
			  yield 3;
			}
	
			var g = gen();
	
			g.next()        // { value: 1, done: false }
			g.return('foo') // { value: "foo", done: true }
			g.next()        // { value: undefined, done: true }
```
上面代码中，遍历器对象g调用return方法后，返回值的value属性就是return方法的参数foo。并且，Generator 函数的遍历就终止了，返回值的done属性为true，以后再调用next方法，done属性总是返回true。

如果return方法调用时，不提供参数，则返回值的value属性为undefined。

如果 Generator 函数内部有try...finally代码块，那么return方法会推迟到finally代码块执行完再执行。

```javascript
			function* numbers () {
			  yield 1;
			  try {
			    yield 2;
			    yield 3;
			  } finally {
			    yield 4;
			    yield 5;
			  }
			  yield 6;
			}
			var g = numbers();
			g.next() // { value: 1, done: false }
			g.next() // { value: 2, done: false }
			g.return(7) // { value: 4, done: false }
			g.next() // { value: 5, done: false }
			g.next() // { value: 7, done: true }
```
上面代码中，调用return方法后，就开始执行finally代码块，然后等到finally代码块执行完，再执行return方法。yield*表达式 任何数据结构只要有 Iterator 接口，就可以被yield*遍历。如果在 Generator 函数内部，调用另一个 Generator 函数，默认情况下是没有效果的。

```javascript
			function* foo() {
			  yield 'a';
			  yield 'b';
			}
	
			function* bar() {
			  yield 'x';
			  foo();
			  yield 'y';
			}
	
			for (let v of bar()){
			  console.log(v);
			}
			// "x"
			// "y"
```
上面代码中，foo和bar都是 Generator 函数，在bar里面调用foo，是不会有效果的。这个就需要用到yield*表达式，用来在一个 Generator 函数里面执行另一个 Generator 函数。

```javascript
			function* bar() {
			  yield 'x';
			  yield* foo();
			  yield 'y';
			}
	
			// 等同于
			function* bar() {
			  yield 'x';
			  yield 'a';
			  yield 'b';
			  yield 'y';
			}
	
			// 等同于
			function* bar() {
			  yield 'x';
			  for (let v of foo()) {
			    yield v;
			  }
			  yield 'y';
			}
	
			for (let v of bar()){
			  console.log(v);
			}
			// "x"
			// "a"
			// "b"
			// "y"
```
任何数据结构只要有 Iterator 接口，就可以被yield*遍历。

```javascript
			let read = (function* () {
			  yield 'hello';
			  yield* 'hello';
			})();
	
			read.next().value // "hello"
			read.next().value // "h"
```
如果被代理的 Generator 函数有return语句，那么就可以向代理它的 Generator 函数返回数据。

```javascript
			function* foo() {
			  yield 2;
			  yield 3;
			  return "foo";
			}
	
			function* bar() {
			  yield 1;
			  var v = yield* foo();
			  console.log("v: " + v);
			  yield 4;
			}
	
			var it = bar();
	
			it.next()
			// {value: 1, done: false}
			it.next()
			// {value: 2, done: false}
			it.next()
			// {value: 3, done: false}
			it.next();
			// "v: foo"
			// {value: 4, done: false}
			it.next()
			// {value: undefined, done: true}
```
Generator函数中的this

Generator函数不能跟new一起用，会报错，而且也无法拿到this对象

```javascript
			function* g() {
			  this.a = 11;
			}
	
			let obj = g();
			obj.next();
			obj.a // undefined
```
解决办法：

```javascript
			function* F() {
			  this.a = 1;
			  yield this.b = 2;
			  yield this.c = 3;
			}
			var obj = {};
			var f = F.call(obj);
	
			f.next();  // Object {value: 2, done: false}
			f.next();  // Object {value: 3, done: false}
			f.next();  // Object {value: undefined, done: true}
	
			obj.a // 1
			obj.b // 2
			obj.c // 3
```
上面代码中，首先是F内部的this对象绑定obj对象，然后调用它，返回一个 Iterator 对象。这个对象执行三次next方法（因为F内部有两个yield表达式），完成 F 内部所有代码的运行。这时，所有内部属性都绑定在obj对象上了，因此obj对象也就成了F的实例。上面代码中，执行的是遍历器对象f，但是生成的对象实例是obj，有没有办法将这两个对象统一呢？一个办法就是将obj换成F.prototype。

```javascript
			function* F() {
			  this.a = 1;
			  yield this.b = 2;
			  yield this.c = 3;
			}
			var f = F.call(F.prototype);
	
			f.next();  // Object {value: 2, done: false}
			f.next();  // Object {value: 3, done: false}
			f.next();  // Object {value: undefined, done: true}
	
			f.a // 1
			f.b // 2
			f.c // 3
```
再将F改成构造函数，就可以对它执行new命令了。

```javascript
			function* gen() {
			  this.a = 1;
			  yield this.b = 2;
			  yield this.c = 3;
			}
	
			function F() {
			  return gen.call(gen.prototype);
			}
	
			var f = new F();
	
			f.next();//Object {value: 2, done: false}将b部署到了gen.prototype上
			f.next();  // Object {value: 3, done: false}
			f.next();  // Object {value: undefined, done: true}
	
			f.a // 1
			f.b // 2
			f.c // 3
```
`*`new F()的对象不会共享yield的进程，共享gen.prototype上的属性
Generator应用：
	异步操作同步化表达
	Ajax 是典型的异步操作，通过 Generator 函数部署 Ajax 操作，
	可以用同步的方式表达。

```javascript
			function* main() {
			  var result = yield request("http://some.url");
			  var resp = JSON.parse(result);
			    console.log(resp.value);
			}
	
			function request(url) {
			  makeAjaxCall(url, function(response){
			    it.next(response);
			  });
			}
	
			var it = main();
			it.next();
```
控制流管理
部署iterator接口
作为数据结构

#####Generator函数的异步应用
	ES6以前，异步编程有四种：
		回调函数：解决异步编程，问题就是嵌套太多
		事件监听：
		发布/订阅
		Promise对象：解决回调的横向编程问题，问题是代码冗余
	协程的Generator函数实现
		整个 Generator 函数就是一个封装的异步任务，或者说是异步任务的容器。
		异步操作需要暂停的地方，都用yield语句注明。Generator 函数的执行方法如下。
```javascript
			function* gen(x) {
			  var y = yield x + 2;
			  return y;
			}

			var g = gen(1);
			g.next() // { value: 3, done: false }
			g.next() // { value: undefined, done: true }
```
Generator 函数的数据交换和错误处理next返回值的 value 属性，是 Generator 函数向外输出数据；next方法还可以接受参数，向 Generator 函数体内输入数据。

```javascript
			function* gen(x){
			  var y = yield x + 2;
			  return y;
			}
	
			var g = gen(1);
			g.next() // { value: 3, done: false }
			g.next(2) // { value: 2, done: true }
```
上面代码中，第一个next方法的value属性，返回表达式x + 2的值3。第二个next方法带有参数2，这个参数可以传入 Generator 函数，作为上个阶段异步任务的返回结果，被函数体内的变量y接收。因此，这一步的value属性，返回的就是2（变量y的值）。

Generator 函数内部还可以部署错误处理代码，捕获函数体外抛出的错误。

```javascript

function* gen(x){
	try {
		var y = yield x + 2;
	} catch (e){
		console.log(e);
	}
	return y;
}
var g = gen(1);
g.next();
g.throw('出错了');
// 出错了
```
上面代码的最后一行，Generator 函数体外，使用指针对象的throw方法抛出的错误，可以被函数体内的try...catch代码块捕获。这意味着，出错的代码与处理错误的代码，实现了时间和空间上的分离，这对于异步编程无疑是很重要的。	

异步任务的封装

```javascript
		var fetch = require('node-fetch');
		function* gen(){
		  var url = 'https://api.github.com/users/github';
		  var result = yield fetch(url);
		  console.log(result.bio);
		}
	
		var g = gen();
		var result = g.next();
		result.value.then(function(data){
		  return data.json();
		}).then(function(data){
		  g.next(data);
		});
```
上面代码中，首先执行 Generator 函数，获取遍历器对象，然后使用next方法（第二行），执行异步任务的第一阶段。由于Fetch模块返回的是一个 Promise 对象，因此要用then方法调用下一个next方法。可以看到，虽然 Generator 函数将异步操作表示得很简洁，但是流程管理却不方便（即何时执行第一阶段、何时执行第二阶段）。Thunk函数 是自动执行Generator函数的一种方法



