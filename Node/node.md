# 第一章   简介

### node的特点

异步I/O	单线程	事件与回调函数（event loop）	跨平台

### Node的应用场景

由于Node的单线程和异步I/O，CPU会一直处于工作状态。所以Node很适合用于I/O密集型和CPU密集型的应用。

# 第二章   模块机制

### 模块加载

1. **模块的导出和引入**

   ​	模块由module.exports或者exports导出，require导入。

   ```javascript
   // 导入
   var math = require('math');
   var a = 1,b = 2;
   math.add(a,b)
   
   //math.js
   exports.add = function (var a, var b) {
       return a + b;
   }
   ```
   ​	`module.exports`和`exports`指向的是同一个。但是一旦其中一个改变了就只能使用module.exports上的数据和方法。如果是exports指向改变，module.exports导出的就是一个空对象。如果是module.exports改变，导出的就只有改变后的module.exports上的方法和成员。

2. **模块的加载顺序**`*`

   ​	Node对引入过的模块会进行缓存，以减少二次引入的开销。同名模块引入的优先级是：缓存 > 核心模块 > 文件模块，在没有扩展名的情况会优先加载 .js > .node > .json文件。

   ​	核心模块就是Node自带的模块。而文件模块则是用户自定义的模块

3. **模块的编译**

   ​	在编译过程中，Node对获取的js文件内容进行了头尾包装。

   ```javascript
   (function (exports, require, module, __filename, __dirname) {
       // js文件的内容
   })
   ```

   ​	这样每个模块之间都进行了作用域隔离。包装之后的代码会通过原生的vm模块的runInThisContext()方法执行。返回一个具体的function对象。

4. **模块的导出**

   ​	文件模块依赖核心模块(JS)，核心模块依赖内建模块(C/C++)，使用C/C++内建模块的原因是静态语言的性能是优于JS的。所以，为了性能的考虑，最底层采用的是C++编写。

   ![](C:\Users\baiai\Pictures\应用\模块依赖图.png)

5. **总结**

   ​	Node的模块机制是采用的CommonJS规范。适用于后台开发，而前端采用的是CMD或者AMD规范。模块的意义在于可以让各个文件之间形成独立的作用域从而方便代码管理和不造成全局污染。一个文件就是一个模块。重点掌握module.exports和exports导出的区别。

# 第三章 异步IO

1. ###**为什么要用异步I/O**

   ​	从在资源加载和用户体验角度上看，在浏览器中，JavaScript和UI渲染使用的是同一个单线程。这就导致资源加载时，浏览器什么都不能做。JS需要等待资源加载完成才能执行，这期间UI停顿，不响应用户的交互行为。所以，这个时候，如果采用异步I/O，在加载资源的时候，JS和UI执行都不会处于等待状态，可以继续响应用户的操作。

   ​	同时，如果加载两个不相关的资源时。采用同步的代码是：

   ```javascript
   getData('from_db'); // 消耗时长M
   getData('from_remote_api'); // 消耗时长N
   ```

   ​	此时消耗的时间是M+N。而如果采用异步的方式的话，代码如下：

   ```javascript
   getData('from_db', function (result) {
       // 处理返回结果
   });
   getData('from_remote_api', function (result) {
       // 处理返回结果
   })；
   ```

   ​	此时消耗的时间是max(M, N)。

   ​	从资源分配的角度上看，单线程同步编程会导致因为I/O阻塞造成的硬件资源得不到更好的利用，而如果采用多线程编程又会因为死锁、状态同步等问题烦恼。所以，Node通过单线程避免死锁、状态同步的问题，通过异步I/O避免I/O阻塞的问题。

   ![](C:\Users\baiai\Pictures\应用\异步I_O调用示意图.png)

2. ###**Node的异步I/O**

   1. ####阻塞I/O与非阻塞I/O的区别

      ​	阻塞I/O会立即调用阻塞I/O，然后处于等待时间，直到所有操作完成再继续。而非阻塞I/O会在调用之后立即返回部分资源，之后通过轮询的方式查看操作是否完成。轮询主要是通过epoll的方式。非阻塞I/O虽然减少了CPU的耗用，但是等待期间的CPU几乎是闲置的。

   2. ####Node的异步I/O

      ![](C:\Users\baiai\Pictures\应用\异步I_O流程.png)

      ​	以上是整个异步I/O的流程，从图中可以看见。当程序发起异步调用之后，Node会将请求封装到一个请求对象中，一个请求对象中可能有多个异步调用请求。封装好请求对象之后，将请求对象放入线程池，等待系统操作完成之后，会调用回调函数执行请求结果。

      ​	异步I/O过程中，有几个关键词：单线程、事件循环、观察者、线程池。单线程指的是JavaScript的执行，线程池则是因为Node自身是多线程的。所以，除了用户代码无法并行执行之外，其他的异步操作是可以并行执行的。

   3. ####**事件循环机制**

      事件循环机制是异步实现的核心。具体参考：[C:\Users\baiai\Desktop\常用\js.md](C:\Users\baiai\Desktop\常用\js.md)




# 第四章 异步编程

1. ### 函数式编程

   ​	在Node中，函数作为一等公民，可以作为参数和返回值的形式出现。而这也是`高阶函数`和普通函数的区别。其次是`偏函数`的概念。偏函数是通过传入的参数或者预置的函数生成另一个函数的函数。通过以下具体了解一下什么是偏函数。

   ```javascript
   var toString  = Object.prototype.toString;
   var isString  = function (obj){
       return toString.call(obj) == "[Object String]";
   };
   var isFunction = function (obj) {
       return toString.call(obj) == '[Object Function]';
   }
   ```

   ​	通过上面的栗子可以看出，如果我们需要更多的判别方法，就需要建立更多的方法实例。这重复的代码会导致代码冗余。那如果我们能够通过一个函数工厂来定制我们需要的判别方法就科一这个问题了。

   ```javascript
   var isType = function (type) {
       return function (type) {
           return  toString.call(obj) == `[Object ${type}]`;
       }
   }
   var isString = isType('String');
   var isFunction = isType('Function');
   ```

   ​	如上所示，通过isType函数可以创造许多和isString和isFunction一样的判别函数去判别其他的类型。这个就是偏函数。

2. ### Node异步编程的优难点

   异步编程的优点就是资源利用率高，难点主要有以下几点：

   1. #### 异常处理：可以用try-catch-finnlly语句块解决

   2. #### 嵌套过深-回调地狱：Promise可以解决

   3. #### 阻塞代码：有些代码需要延后执行：目前较好的是setTimeOut等时间戳方法

   4. #### 多线程编程：还得再查资料

   5. #### 异步转同步：还得查资料

3. ### 异步编程

   1. 事件回调（发布-订阅模式）
   2. Promise/Deferred模式
   3. 流程控制库

4. ### 异步并发控制

   继续查资料

5. ### 总结

   ​	对于异步编程和异步并发控制的知识点还要继续深入了解。



# 第五章 内存控制

1. ### V8引擎的垃圾回收机制与内存限制

   ​	Node是基于V8引擎的语音，但是V8引擎本身具有一些内存限制。因为V8一开始是为浏览器开发的，在Node中通过JavaScript使用内存的时候，只能使用部分内存。（64bit可以使用1.4GB，32bit可以使用0.7GB）。为什么要限制V8的内存的原因是因为V8引擎在做垃圾回收的时候，一次小的垃圾回收需要50ms+的时间，做一次非增量式的垃圾回收甚至需要1000ms+的时间。

   ​	垃圾回收机制将内存分为了老生代和新生代两个内存区域。老生代表示的是长期存活在内存中的对象。而新生代则是刚进入内存不久的对象。当新生代中的对象存活的时间到了一定程度的时候，就会晋升到老生代。之所以分为新、老两代是因为有些对象只会在内存中存在一小会儿的时间，有些对象会在内存中长期存在。所以，需要使用不同的算法来清除。

   ​	新生代的垃圾回收机制采用的是Scavenge算法，其中最主要的算法是Cheney算法。Cheney算法将新生代内存一分为二，一个是处于使用中的From，一个是处于闲置的To。Cheney算法会检测From中的还存活对象，还存活着的对象复制到To，非存活对象占用的空间就会被释放。完成复制后，两个空间兑换。Scavenge是牺牲了空间来换取时间的算法，因为To空间始终不会用。而新生代中的大多数对象存活时间短。所以可以使用这种算法。但是老生代不行。老生代大多对象都可以长期存活，直接少了一半的内存会导致内存不足。

   ​	老生代采用的是Mark-Sweep和Mark-Compact。Mark-Sweep分标记和清除两个部分，在标记阶段，将遍历堆中还存活的对象进行标记，清除阶段会清除未标记的对象，释放空间。但是Mark-Sweep有一个缺点是每次清除对象后，留下的空间会造成内存碎片现象，导致晋升到老生代的对象没有足够的空间。此时就需要采用另一种方案--Mark-Compact。Mark-Compact是基于Mark-Sweep的。只不过，在清除阶段会将标记的对象往内存一端移动，从而得到一个一边是存活对象，一边是可用内存空间的现象。消除了内存碎片，不过这种方法因为有内存移动，会导致效率不够高。所以只有内存不够用的时候，才会采用这种回收机制。

   ​	在垃圾回收机制中，闭包和全局变量引用都不能立即回收。不过全局变量可以通过delete和赋值解决内存问题。如下所示：

   ```javascript
   global.foo = 'global';
   delete global.foo; // way 1，没有2 3好。delete可能会干扰V8的优化。
   global.foo = null; // way 2
   global.foo = undefined; // way 3
   ```

   ​	但是闭包则不行，闭包是指实现外部作用域访问内部作用域中的变量的方法。如下：

   ```javascript
   function wrapper(){
       var inner = function (){
           let innerVar = 1;
           return function (){
               return innerVar;
           };
       };
       var baz = inner();
       console.log(baz());
   }
   ```

   ​	函数inner返回一个匿名函数，这个匿名函数可以访问内部的变量innerVar。当inner执行结束后，本来会被回收了，但是由于baz指向了返回的匿名函数，导致这个匿名函数并没有被回收，导致inner这个作用域也一直存在。只有当匿名函数不在被引用，才会逐步释放。否则会一直存在于老生代，占用内存。

2. ### 内存泄漏

   ​	内存泄漏的是指就是应该被回收的对象没有被及时回收，变成了常驻在老生代内存中的对象。通常造成内存泄漏的原因有三点：缓存、队列消费不及时、作用域未释放

   1. #### 缓存

      ​	在Node中，由于内存有限，如果讲一个对象放置在内存中当做缓存对象的话，那就意味缓存对象将会常驻老生代，在前端可能没有什么问题，但是在Node作为后台，由于执行量大、参数多样性等情况下，会造成内存占用不释放的问题。所以一定要`慎用`内存做缓存。

      ​	如何使用大量缓存，目前较好的解决方案是采用进程外缓存，优点是：

      ​	a.     将缓存转移到外部，减少常驻内存对象的数量，让垃圾回收更高效。

      ​	b.     进程之间可以共享缓存

      ​	目前较好的缓存有Redis和Memcached。

   2. #### 队列消费不及时

      ​	通常情况下，生产速度是赶不上消费速度的。但是一旦生产速度大于了消费速度，消息队列中就会存储大量的生产记录，从而导致内存泄漏。

      ​	比如，做日志收集的时候，如果用数据库来记录日志，由于日志是海量的，而数据库是构建在文件系统之上的。写入的效率远远低于文件直接写入。于是就会导致数据库写入操作的堆积，而JS相关的作用域也不会得到释放，内存占用不会回落，从而出现内存泄漏。

      ​	要解决上面的问题，可以将数据库写入改为文件写入。但是这个方法治标不治本，万一文件写入也赶不上生产速度怎么办。这个时候就需要监控消息队列的长度，一旦超过阈值就返回警告。或者是采用超时机制，一旦在规定的时间内没有完成响应，就调用回调函数传递超时异常。或者当消息队列满了的时候，就直接拒绝其他请求。

   3. 内存泄漏排查工具

      V8-profiler、node-heapdump、node-mtrace、dtrace、node-memwatch




# 第六章 Buffer

​	buffer是什么？buffer就像一个Array对象，但是它主要操作的对象是字节。Buffer主要应用于二进制数据的处理，比如文件流的读写、网络请求数据的处理。buffer的性能模块都是有C++完成的，因为C++的从能远高于JS，而非性能模块则是由JS完成的。Buffer所占用的内存不是通过V8分配的，属于堆外内存。由于Buffer是一个十分常见应用场景。所以Node将他放在了全局对象上。不用引入也可以直接使用。

​	Buffer对象的一些基本操作：

```javascript
var str = "Node笔记"；
var buf = new Buffer(str, 'utf-8'); // var buf = new Buffer(100);分配了100个元素长度的buffer
console.log(buf); // <Buffer 4e 6f 64 65 e7 ac 94 e8 ae b0>其中一个中文字符占三个元素。
console.log(buf.length); // 10
buf[0] = 'n'; // 赋值、修改
console.log(buf); // <Buffer 00 6f 64 65 e7 ac 94 e8 ae b0>

// 字符串转Buffer
var buf = new Buffer(str, [encoding]);
buf.write(string, [offset], [length], [encoding]);

// Buffer转字符串
buf.toString([encoding], [start], [end]);

// Buffer拼接
var buf1 = new Buffer('buffer1 ');
var buf2 = new Buffer('buffer2');
var buf3 = Buffer.concat([buf1, buf2]);
```



​	总结：Buffer是二进制数据。和字符串存在编码关系。但是二进制数据在传输和读取上都比字符串要快很多。除了在Buffer和字符串之间转换需要一定的性能消耗之外。它的性能远高于读取和传输字符串。所以，处于效率考虑很多时候传输都会采用Buffer。所以要理解Buffer一定要从以下四点入手：

1. 字符串转Buffer、Buffer转字符串
2. Buffer拼接
3. Buffer的读取
4. Buffer性能相关的操作

更多Buffer操作比如Buffer实例的比较、拷贝、查找、遍历、类型转换、截取、编码转换等参考官方文档：http://nodejs.cn/api/buffer.html



# 第七章 进程

###单线程事件驱动服务器模型的缺点:

1. 无法利用现在计算机多核的优点，利用率低。
2. 一旦单线程的异常未被捕获，整个进程会崩溃。所以健壮性和稳定性不好。

###解决多核利用的问题：

1. child_process模块去创建子进程，利用子进程去完成其他的任务。父子进程之间可以通过onmessage和PostMessage传递信息。也可以通过回调函数。如下面这个栗子：

   ```javascript
   // worker.js
   var http = require('http');
   http.createServer(function (req, res){
       res.writeHead(200, {
           'Content-Type': 'text/plain'
       });
       res.end('hello world');
   }).listen(Math.ceil(1+Math.round()*1000), 'localhost');
   
   // master.js
   var fork = require('child_process').fork;
   var cpus = require('os').cpus();
   for(var i = 0; i < cpus.length; i++){
       fork('./worker.js');
   }
   ```

   从上面的代码可以看出。主进程master.js在每个CPU上都开启了一个worker.js的子进程，而这些子进程都在完成一个创建http服务器并传输一个Hello world的字符串。可以看出，master.js启动了多个进程去完成其他的事情。然后他自身依然还在工作。从而达到多和利用的目的。

2. 子进程的创建

   模块child_process是专门用来创建和管理子进程的，其中子进程的创建有四种方式。

   spawn()：启动一个子进程来执行命令；

   exec()：启动一个子进程来执行命令，但是它可以设置超时时间，一旦超过时间，进程将会被杀死；

   execFile()：启动一个子进程来执行可执行文件，可设置超时时间。

   fork()：创建一个Node的子进程执行js文件模块。

   这四个方法都会返回一个子进程对象，具体用法如下所示：

   ```javascript
   var cp = require('child_process');
   cp.spawn('node', ['worker.js']);
   cp.exec('node worker.js', function (err, stdout, stderr){
       // callback method
   });
   cp.execFile('worker.js', function (err, stdout, stderr){
       //callback method
   });
   cp.fork('./worker.js');
   ```

   这四个方法之间的细微差别如下图所示：

   ![](C:\Users\baiai\Pictures\应用\创建子进程.png)

3. 进程间的通信

4. 句柄传递

   
