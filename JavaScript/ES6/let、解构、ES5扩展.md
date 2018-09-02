# let和const命令

###let的特点

`let`是声明变量用的，但是它所声明的变量只在let命令所在的代码块中有效，同时`let`会将它所在的代码声明为块级作用域：

```js
{
    let a = 10;
    var b = 20;
}
console.log(a); // ReferenceError: a is not defined
console.log(b); // 20
```

`let`声明的变量不存在`变量提升`，即：

```js
console.log(a); // Reference Error: a is not defined
let a = 10;

console.log(b); // 20
var b = 20;
```

在代码块中，在`let`声明变量之前，该变量都是不可用的，即使是在外部作用域已经定义了，这在语法上称为`暂时性死区(temproal dead zone)`。比如：

```js
var tmp = 123;

if (true) {
  // TDZ开始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}
```

所以，为了避免`变量提升`和`暂时性死区`带来的一些不必要的报错，建议将`let`声明放在作用域的头部。



### const

`const`是在`let`的基础上，又添加另外两个属性：

1. 声明时，必须赋值；
2. 赋值之后，`const`变量所指向的地址不得改变。

同时`const`、`let`和`class`声明的全局变量不再属于顶层对象，比如：

```js
var a = 1;
window.a // 1

let b = 1;
window.b // undefined
```



# 变量解构赋值

### 数组解构

```js
// 完全解构，两边的样式一样
let [a, b, c] = [1, 2, 3]; // a === 1, b === 2, c === 3
let [a, [[b], c]] = [1, [[2], 3]]; // a === 1, b === 2, c === 3
let [ , , c] = [1, 2, 3]; // c === 3

// 不完全解构，有一边是另一边的一部分而不是全部
let [a, b, c] = [1, 2]; // a === 1, b === 2, c === undefined
let [a, b] = [1, 2, 3]; // a === 1, b === 2

// 默认值，默认值生效的条件是对象的属性值严格等于undefined
let [a = 1, b] = [, 2]; // a === 1, b === 2
```

注意，右边的表达式不一定非要是数组，只要有`Iterator`接口就可以。



### 对象解构

```js
// foo === 'aaa', bar === 'bbb', baz === undefined
let {foo, bar, baz} = {foo:'aaa', bar: 'bbb'}; 
// 相当于 let {foo: foo, bar: bar, baz: baz} = {foo:'aaa', bar: 'bbb'}; 

// 变量名和属性名不一样时
let { foo: baz } = { foo: 'aaa', bar: 'bbb' }; // baz === 'aaa'

// 默认值，默认值生效的条件是对象的属性值严格等于undefined
let {foo = 'bbb', bar = 'bbb'} = {foo: 'aaa', bar:undefined}; // foo->'aaa', bar->'bbb'

// 嵌套结构
const node = {
  loc: {
    start: {
      line: 1,
      column: 5
    }
  }
};

let { loc, loc: { start }, loc: { start: { line }} } = node;
// loc->{start: {line: 1, column: 5}}, start->{line: 1, olumn: 5}, line->1
```



### 函数参数解构

函数的参数也可以使用解构赋值：

```js
function add([x, y]){
  return x + y;
}

add([1, 2]); // 3
```

函数参数的解构也可以使用默认值：

```js
function move({x = 0, y = 0}) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, 0]
move({}); // [0, 0]
move(); // [0, 0]
```

函数参数设置默认值：

```js
function move({x, y} = { x: 0, y: 0 }) {
  return [x, y];
}

move({x: 3, y: 8}); // [3, 8]
move({x: 3}); // [3, undefined]
move({}); // [undefined, undefined]
move(); // [0, 0]
```

上面代码是为函数`move`的参数指定默认值，而不是为变量`x`和`y`指定默认值，所以会得到与前一种写法不同的结果。 



### 用途

变量的解构赋值用途很多。

**（1）交换变量的值**

```js
let x = 1;
let y = 2;

[x, y] = [y, x];
```

上面代码交换变量`x`和`y`的值，这样的写法不仅简洁，而且易读，语义非常清晰。

**（2）从函数返回多个值**

函数只能返回一个值，如果要返回多个值，只能将它们放在数组或对象里返回。有了解构赋值，取出这些值就非常方便。

```js
// 返回一个数组

function example() {
  return [1, 2, 3];
}
let [a, b, c] = example();

// 返回一个对象

function example() {
  return {
    foo: 1,
    bar: 2
  };
}
let { foo, bar } = example();
```

**（3）函数参数的定义**

解构赋值可以方便地将一组参数与变量名对应起来。

```js
// 参数是一组有次序的值
function f([x, y, z]) { ... }
f([1, 2, 3]);

// 参数是一组无次序的值
function f({x, y, z}) { ... }
f({z: 3, y: 2, x: 1});
```

**（4）提取 JSON 数据**

解构赋值对提取 JSON 对象中的数据，尤其有用。

```js
let jsonData = {
  id: 42,
  status: "OK",
  data: [867, 5309]
};

let { id, status, data: number } = jsonData;

console.log(id, status, number);
// 42, "OK", [867, 5309]
```

上面代码可以快速提取 JSON 数据的值。

**（5）函数参数的默认值**

```js
jQuery.ajax = function (url, {
  async = true,
  beforeSend = function () {},
  cache = true,
  complete = function () {},
  crossDomain = false,
  global = true,
  // ... more config
} = {}) {
  // ... do stuff
};
```

指定参数的默认值，就避免了在函数体内部再写`var foo = config.foo || 'default foo';`这样的语句。

**（6）遍历 Map 结构**

任何部署了 Iterator 接口的对象，都可以用`for...of`循环遍历。Map 结构原生支持 Iterator 接口，配合变量的解构赋值，获取键名和键值就非常方便。

```js
const map = new Map();
map.set('first', 'hello');
map.set('second', 'world');

for (let [key, value] of map) {
  console.log(key + " is " + value);
}
// first is hello
// second is world
```

如果只想获取键名，或者只想获取键值，可以写成下面这样。

```js
// 获取键名
for (let [key] of map) {
  // ...
}

// 获取键值
for (let [,value] of map) {
  // ...
}
```

**（7）输入模块的指定方法**

加载模块时，往往需要指定输入哪些方法。解构赋值使得输入语句非常清晰。

```js
const { SourceMapConsumer, SourceNode } = require("source-map");
```



# 相对ES5的一些扩展

### 字符串

1. 字符串遍历接口：

   ```js
   for (let codePoint of 'foo') {
     console.log(codePoint) // f o o
   }
   ```

2. includes、startsWith、endsWith：

   ```js
   let str = “今天吃的是apple”;
   
   str.startWith(‘今天’); //返回true，判断str是否是以今天开头
   str.endsWith(‘le’); //true,判断str是否是以le结尾
   str.includes(‘apple’); //true，判断str是否含有apple这个字符串
   ```

3. repeat、matchAll（见正则）

   ```js
   'hello'.repeat(2) // "hellohello"
   'na'.repeat(0) // ""
   ```

4. 模板字符串



### 正则扩展

1. `y`修饰符：与`g`大致相同，区别是`y`要求匹配的位置必须是从剩余的第一个位置开始

   ```js
   var s = 'aaa_aa_a';
   var r1 = /a+/g;
   var r2 = /a+/y;
   
   r1.exec(s) // ["aaa"]
   r2.exec(s) // ["aaa"]
   
   r1.exec(s) // ["aa"]
   r2.exec(s) // null，匹配的位置没有从下一个位置即`_`开始
   ```

2. 具名组匹配，可以直接匹配具名，就不用担心序号变了，引用的数组也要跟着变。

   ```js
   // 不用具名
   const RE_DATE = /(\d{4})-(\d{2})-(\d{2})/;
   
   const matchObj = RE_DATE.exec('1999-12-31');
   const year = matchObj[1]; // 1999
   const month = matchObj[2]; // 12
   const day = matchObj[3]; // 31
   
   // 具名
   const RE_DATE = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
   
   const matchObj = RE_DATE.exec('1999-12-31');
   const year = matchObj.groups.year; // 1999
   const month = matchObj.groups.month; // 12
   const day = matchObj.groups.day; // 31
   ```

3. matchAll

   ```js
   const string = 'test1test2test3';
   
   // g 修饰符加不加都可以
   const regex = /t(e)(st(\d?))/g;
   
   for (const match of string.matchAll(regex)) {
     console.log(match);
   }
   // ["test1", "e", "st1", "1", index: 0, input: "test1test2test3"]
   // ["test2", "e", "st2", "2", index: 5, input: "test1test2test3"]
   // ["test3", "e", "st3", "3", index: 10, input: "test1test2test3"]
   ```

   上面代码中，由于`string.matchAll(regex)`返回的是遍历器，所以可以用`for...of`循环取出。相对于返回数组，返回遍历器的好处在于，如果匹配结果是一个很大的数组，那么遍历器比较节省资源。 



### 数值扩展

1. Number.isFinite()，Number.isNaN()，Number.isInteger()，Number.isSafeInteger()
2. Number.parseInt()，Number.parseFloat()
3. Math对象的扩展



### 函数扩展（*）

1. 箭头函数：

   箭头函数有几个使用注意点：

   （1）函数体内的`this`对象，就是定义时所在的对象，而不是使用时所在的对象。

   （2）不可以当作构造函数，也就是说，不可以使用`new`命令，否则会抛出一个错误。

   （3）不可以使用`arguments`对象，该对象在函数体内不存在。如果要用，可以用 rest 参数代替。

   （4）不可以使用`yield`命令，因此箭头函数不能用作 Generator 函数。

   箭头函数转换成ES5：

   ```js
   // ES6
   function foo() {
     setTimeout(() => {
       console.log('id:', this.id);
     }, 100);
   }
   
   // ES5
   function foo() {
     var _this = this;
   
     setTimeout(function () {
       console.log('id:', _this.id);
     }, 100);
   ```

   上面代码中，转换后的 ES5 版本清楚地说明了，箭头函数里面根本没有自己的`this`，而是引用外层的`this`。 

2. 尾递归及其优化

   `尾调用`：指某个函数的最后一步是调用另一个函数 

   ```js
   function f(x){
     return g(x);
   }
   // 以下三种都不是尾调用
   // 情况一
   function f(x){
     let y = g(x);
     return y;
   }
   
   // 情况二
   function f(x){
     return g(x) + 1;
   }
   
   // 情况三
   function f(x){
     g(x); // 相当于：g(x); return undefined;
   }
   ```

   ​	函数调用会在内存形成一个“调用记录”，又称“调用帧”（call frame），保存调用位置和内部变量等信息。如果在函数`A`的内部调用函数`B`，那么在`A`的调用帧上方，还会形成一个`B`的调用帧。等到`B`运行结束，将结果返回到`A`，`B`的调用帧才会消失。如果函数`B`内部还调用函数`C`，那就还有一个`C`的调用帧，以此类推。所有的调用帧，就形成一个“调用栈”（call stack）。

   ​	尾调用由于是函数的最后一步操作，所以不需要保留外层函数的调用帧，因为调用位置、内部变量等信息都不会再用到了，只要直接用内层函数的调用帧，取代外层函数的调用帧就可以了。

   

   `尾递归`：函数调用自身，称为递归。如果尾调用自身，就称为尾递归。 递归非常耗费内存，因为需要同时保存成千上百个调用帧，很容易发生“栈溢出”错误（stack overflow）。但对于尾递归来说，由于只存在一个调用帧，所以永远不会发生“栈溢出”错误。 比如下面非尾递归实现的斐波那契数列：

   ```js
   function Fibonacci (n) {
     if ( n <= 1 ) {return 1};
   
     return Fibonacci(n - 1) + Fibonacci(n - 2);
   }
   
   Fibonacci(10) // 89
   Fibonacci(100) // 堆栈溢出
   Fibonacci(500) // 堆栈溢出
   ```

   上面代码是一个阶乘函数，计算`n`的斐波那契的值，最多需要保存`n`个调用记录，复杂度 O(n)  

   尾递归优化过的 Fibonacci 数列只保留一个调用记录，复杂度为O(1)，实现如下：

   ```js
   function Fibonacci2 (n , ac1, ac2) {
     if( n <= 1 ) {return ac2};
   
     return Fibonacci2 (n - 1, ac2, ac1 + ac2);
   }
   
   Fibonacci2(100, 1, 1) // 573147844013817200000
   Fibonacci2(1000, 1, 1) // 7.0330367711422765e+208
   Fibonacci2(10000, 1, 1) // Infinity
   ```

   `递归函数的改写`：尾递归的实现，往往需要改写递归函数，确保最后一步只调用自身。做到这一点的方法，就是把所有用到的内部变量改写成函数的参数。 但是这样就有一个缺点，就是函数调用不够直观，比如上面的菲波那切数列其实只需要输入一个`n`即可，不必输入`ac1`和`ac2`的。两个办法可以解决，方法一是在尾递归函数之外，再提供一个正常形式的函数：

   ```js
   function Fibonacci2 (n , ac1 = 1 , ac2 = 1) {
     if( n <= 1 ) {return ac2};
   
     return Fibonacci2 (n - 1, ac2, ac1 + ac2);
   }
   
   function factorial(n) {
     return Fibonacci2(n, 1, 1);
   }
   
   factorial(5) // 1
   ```

   二是函数柯里化：

   ```js
   function Fibonacci2 (n , ac1 = 1, ac2 = 1) { // 使用了默认值
     if( n <= 1 ) {return ac2};
   
     return Fibonacci2 (n - 1, ac2, ac1 + ac2);
   }
   
   Fibonacci2(100) // 573147844013817200000
   Fibonacci2(1000) // 7.0330367711422765e+208
   Fibonacci2(10000) // Infinity
   ```

   

### 数组的扩展

1. 扩展运算符（...）

   ```js
   console.log(...[1, 2, 3]); // 1 2 3
   function restFunc (...rest) {
       console.log(rest)
   }
   restFunc(1,2,3); // [1, 2, 3]
   ```

2. Array.from、Array.of

   `Array.from`方法用于将两类对象转为真正的数组：类似数组的对象（array-like object）和可遍历（iterable）的对象（包括 ES6 新增的数据结构 Set 和 Map）。 

   ```js
   // 类数组
   let arrayLike = {
       '0': 'a',
       '1': 'b',
       '2': 'c',
       length: 3
   };
   let arr2 = Array.from(arrayLike); // ['a', 'b', 'c']
   
   // 可遍历
   Array.from('hello') // ['h', 'e', 'l', 'l', 'o']
   
   let namesSet = Array.from(new Set(['a', 'b'])); // ['a', 'b']
   ```

   `Array.of`方法用于将一组值，转换为数组。 

   ```js
   Array.of(3, 11, 8) // [3,11,8]
   Array.of(3) // [3]
   Array.of(3).length // 1
   ```

3. Array.prototype.includes

   方法返回一个布尔值，表示某个数组是否包含给定的值，与字符串的`includes`方法类似。ES2016 引入了该方法。

   ```js
   [1, 2, 3].includes(2)     // true
   [1, 2, 3].includes(4)     // false
   [1, 2, NaN].includes(NaN) // true
   ```

   该方法的第二个参数表示搜索的起始位置，默认为`0`。如果第二个参数为负数，则表示倒数的位置，如果这时它大于数组长度（比如第二个参数为`-4`，但数组长度为`3`），则会重置为从`0`开始。

   ```js
   [1, 2, 3].includes(3, 3);  // false
   [1, 2, 3].includes(3, -1); // true
   ```

### 对象扩展

1. Object.is():它用来比较两个值是否严格相等，与严格比较运算符（===）的行为基本一致。 

   ```js
   Object.is('foo', 'foo') // true
   Object.is({}, {}) // false
   
   // 不同之处
   +0 === -0 //true
   NaN === NaN // false
   
   Object.is(+0, -0) // false
   Object.is(NaN, NaN) // true
   ```

2. Object.assign()：用于对象合并

   ```js
   const target = { a: 1 };
   
   const source1 = { b: 2 };
   const source2 = { c: 3 };
   
   Object.assign(target, source1, source2); // target: {a:1, b:2, c:3}
   
   ```

   注意事项：

   - 只拷贝源对象自身`可枚举`属性，不拷贝继承属性

   - 拷贝是浅拷贝，如果拷贝变量是对象时，只是赋值一个指向地址的引用

     ```js
     const obj1 = {a: {b: 1}};
     const obj2 = Object.assign({}, obj1);
     
     obj1.a.b = 2;
     obj2.a.b // 2
     ```

   - 同名属性，后面的参数会覆盖前面的参数

   - 第一个参数不能使`undefined`和`null`

3. Object.getOwnPropertyDescriptors()：指定对象所有自身属性（非继承属性）的描述对象。 

   ```js
   const obj = {
     foo: 123,
     get bar() { return 'abc' }
   };
   
   Object.getOwnPropertyDescriptors(obj)
   // { foo:
   //    { value: 123,
   //      writable: true,
   //      enumerable: true,
   //      configurable: true },
   //   bar:
   //    { get: [Function: get bar],
   //      set: undefined,
   //      enumerable: true,
   //      configurable: true } }
   ```

   