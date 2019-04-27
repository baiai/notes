####比较Generator
就是Generator的语法糖，只是将*换成async，yeild换成await

`优点`：

​	1	内置执行器，可以执行asyncFuntion()就可以执行了

​	2	更好的语义

​	3	更广的适用性，await后面可以是Peomise也可以是原始类型的值

​	4	返回值是Promise，可以使用then方法进行下一步操作

####基本用法

#####声明

```javascript
// 函数声明
async function foo(){}

// 函数表达式
const foo = async function(){};

// 对象的方法
let obj = { async foo(){} };
```

#####语法

async函数返回一个Promise对象

async函数内部return语句返回的值会成为then方法回调函数的参数

```javascript
async function f(){
    return 'hello world';
}
f().then( v => console.log(v) );
```
函数内部跑出错误，async会返回reject状态，抛出的错误对象会被catah方法回调函数接收到

```javascript
async function foo(){
	throw new Error('出错了！');
}
foo().then(
	v => console.log( v ),
    err => console.log( err )
)
```

async函数必须等到所有的await命令后的Promise对象执行完才会返回resolve对象，除非遇到return语句或者抛出错误。也就是说，只有里面的所有异步操作执行完才能调用then指定的回调函数

#####await

`await`命令会返回一个`Promise`的对象，如果是`reject`，即使没有`return`也可以被外面的`catch`捕捉到。而一旦await返回一个reject对象，那么后面的都不用执行了。

```javascript
async function foo(){
    await throw new Error('出错了！');
    await Promise.resolve('hello');
}
foo().then(
	e => console.log(e),
    err => console.log( err )
);
```
如果不希望因为一个`await`返回`reject`导致后面的函数无法执行的话，就必须给可能出错的函数放在一个`try...catch`结构里面或者是在await的Promise后面加上一个catch。

```javascript
// way 1
async function foo(){
    try{
        await throw new Error('出错了！');
    }catch{
   	}
    return await Promise.resolve('hello');
}
foo().then(
	v => console.log( v ),
    err => console.log( err )
); // hello

// way 2
async function foo(){
    await Promise.reject('error').catch( err => console.log( err ));
    return await Promise.resolve('hello');
}
foo().then(
	v => console.log( v ),
    err => console.log( err )
); 
// error
// hello
```
`*`如果多个异步函数并没有继发（即先后）关系的话，可以让他们同时出发

```javascript
// way 1
let [foo,bar] = await Promise.all( [ getFoo(), getBar() ] );

// way 2
let fooPromise = getFoo();
let barPromise = getBar();
let foo = await fooPromise;
let bar = await barPromise;
```


#### 实例

Promise封装的ajax请求

```javascript
function ajax (method,url,data) {
  return new Promise(function (resolve,reject) {
    let xhr = new XMLHttpRequest();
    xhr.open(method,url);
    xhr.onreadystatechange = function () {
      if(xhr.readyState === 4 &&
        (xhr.status === 200 || xhr.status === 304)) {
        resolve(xhr.responseText,xhr.responseXML);
      }
    };
    if(method === 'GET') {
      xhr.send();
    }else if (method === 'POST') {
      xhr.send(data);
    }

    xhr.onerror = function () {
      rejetct("Error has ocurred");
    };
    xhr.onabort = function () {
      reject("request has been aborted");
    };
    timeout = 1000;
    xhr.ontimeout = function () {
      reject("请求超时");
    };

  });
}
```

当需要同时请求两个资源的时候，Promise和Async的写法差异。

Promise写法：

```javascript
let ajax1 = ajax('GET',"https://www.google.com");
let ajax2 = ajax('GET',"Https://www.baidu.com");
Promise.all([ajax1,ajax2]).then(arr => {
    console.log(arr[0]);
    console.log(arr[1]);
});
```

Async写法：

```javascript
async function solution () {
    let ajax1 = ajax('GET',"https://www.google.com");
    let ajax2 = ajax('GET',"Https://www.baidu.com");
    await Promise.all([ajax1,ajax2]);
}
```



参考资料：

[1]	阮一峰ES6

[2]	https://segmentfault.com/a/1190000011498280