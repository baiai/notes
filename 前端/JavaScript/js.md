#js中的事件机制

由于js是单线程的，所以在遇到事件绑定、ajax请求的时候回造成阻塞，因此就产生了异步操作。可以让js的主线程持续工作。在这个过程中主要有三个东西，一是主线程、二是任务队列、三是回调函数。

`主线程`就是不断地从任务队列里面取出一个事件去执行，在执行完这个时间之后再去任务队列取事件。如果任务队列没事件就会处于等待状态。

主线程取到任务队列里的事件之后会将事件的上下文放在`调用栈`中，然后一步一步执行里面的操作。如果遇到异步操作就会交给浏览器内核的其他模块进行处理，等处理完之后，再将回调函数放入任务队列汇总。

`任务队列`是用来存放事件的，当主线程完成当前操作之后，就会从这里面取事件执行。任务队列分为`macro-task`和`micro-task`（micro-task只有一个）两种队列，主线程会先调用marco-task任务队列，直到函数调用栈中当前macro-task执行上下文结束，再执行micro-task中的任务，执行完micro-task中的事件后又去执行macro-task中的事件，如此反复。

macro-task：script(整体代码), setTimeout, setInterval, setImmediate, I/O, UI rendering。

micro-task： process.nextTick, Promises, Object.observe, MutationObserver 

`回调函数`是在异步操作执行完之后，放入任务队列让主线程执行的函数。

示例：

```javascript
// demo
console.log('golb1');

setTimeout(function() {
    console.log('timeout1');
    process.nextTick(function() {
        console.log('timeout1_nextTick');
    })
    new Promise(function(resolve) {
        console.log('timeout1_promise');
        resolve();
    }).then(function() {
        console.log('timeout1_then')
    })
})

setImmediate(function() {
    console.log('immediate1');
    process.nextTick(function() {
        console.log('immediate1_nextTick');
    })
    new Promise(function(resolve) {
        console.log('immediate1_promise');
        resolve();
    }).then(function() {
        console.log('immediate1_then')
    })
})

process.nextTick(function() {
    console.log('glob1_nextTick');
})
new Promise(function(resolve) {
    console.log('glob1_promise');
    resolve();
}).then(function() {
    console.log('glob1_then')
})

setTimeout(function() {
    console.log('timeout2');
    process.nextTick(function() {
        console.log('timeout2_nextTick');
    })
    new Promise(function(resolve) {
        console.log('timeout2_promise');
        resolve();
    }).then(function() {
        console.log('timeout2_then')
    })
})

process.nextTick(function() {
    console.log('glob2_nextTick');
})
new Promise(function(resolve) {
    console.log('glob2_promise');
    resolve();
}).then(function() {
    console.log('glob2_then')
})

setImmediate(function() {
    console.log('immediate2');
    process.nextTick(function() {
        console.log('immediate2_nextTick');
    })
    new Promise(function(resolve) {
        console.log('immediate2_promise');
        resolve();
    }).then(function() {
        console.log('immediate2_then')
    })
})
```

这个例子看上去有点复杂，乱七八糟的代码一大堆，不过不用担心，我们一步一步来分析一下。

第一步：宏任务script首先执行。全局入栈。glob1输出。![script首先执行](http://upload-images.jianshu.io/upload_images/599584-5ae0b593167e499b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第二步，执行过程遇到setTimeout。setTimeout作为任务分发器，将任务分发到对应的宏任务队列中。 ![timeout1进入对应队列](http://upload-images.jianshu.io/upload_images/599584-afded6f26c106326.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第三步：执行过程遇到setImmediate。setImmediate也是一个宏任务分发器，将任务分发到对应的任务队列中。setImmediate的任务队列会在setTimeout队列的后面执行。 ![进入setImmediate队列](http://upload-images.jianshu.io/upload_images/599584-c22a5e6567ec25d3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第四步：执行遇到nextTick，process.nextTick是一个微任务分发器，它会将任务分发到对应的微任务队列中去。 ![nextTick](http://upload-images.jianshu.io/upload_images/599584-8d16de95f6a12b25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第五步：执行遇到Promise。Promise的then方法会将任务分发到对应的微任务队列中，但是它构造函数中的方法会直接执行。因此，glob1_promise会第二个输出。 ![先是函数调用栈的变化](http://upload-images.jianshu.io/upload_images/599584-792877853f338494.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)![然后glob1_then任务进入队列](http://upload-images.jianshu.io/upload_images/599584-b5c548ec48521c87.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第六步：执行遇到第二个setTimeout。 ![timeout2进入对应队列](http://upload-images.jianshu.io/upload_images/599584-0392b96fd8fd2281.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第七步：先后遇到nextTick与Promise ![glob2_nextTick与Promise任务分别进入各自的队列](http://upload-images.jianshu.io/upload_images/599584-7001e3438df47eb0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

第八步：再次遇到setImmediate。 ![nextTick](http://upload-images.jianshu.io/upload_images/599584-eb6742e93ff577cd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这个时候，script中的代码就执行完毕了，执行过程中，遇到不同的任务分发器，就将任务分发到各自对应的队列中去。接下来，将会执行所有的微任务队列中的任务。

其中，nextTick队列会比Promie先执行。nextTick中的可执行任务执行完毕之后，才会开始执行Promise队列中的任务。

当所有可执行的微任务执行完毕之后，这一轮循环就表示结束了。下一轮循环继续从宏任务队列开始执行。

这个时候，script已经执行完毕，所以就从setTimeout队列开始执行。![第二轮循环初始状态](http://upload-images.jianshu.io/upload_images/599584-48cfccebbff92e97.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

setTimeout任务的执行，也依然是借助函数调用栈来完成，并且遇到任务分发器的时候也会将任务分发到对应的队列中去。

只有当setTimeout中所有的任务执行完毕之后，才会再次开始执行微任务队列。并且清空所有的可执行微任务。

setTiemout队列产生的微任务执行完毕之后，循环则回过头来开始执行setImmediate队列。仍然是先将setImmediate队列中的任务执行完毕，再执行所产生的微任务。

当setImmediate队列执行产生的微任务全部执行之后，第二轮循环也就结束了。



#垃圾回收机制（5-25）

​	现在(2018)的浏览器的垃圾回收机制都是采用的标记-清除的方式，每次垃圾回收器运行的时候会先将保存在内存中的所有变量进行标记，然后再从根元素开始寻找，找到的变量和引用就会去掉标记，最后没有找到的变量和引用的标记还存在，这些被标记的变量就会被垃圾回收器回收[1]。

​	js中的垃圾回收是不需要开发者自己去管理的，但是开发者应该注意一下内存管理。尤其全局变量，当一个全局变量不再使用的时候，可以通过赋值为null的方式将其回收[2]。

参考资料：

[1]	JavaScript高级程序设计--P79

[2]	JavaScript高级程序设计--P81



# 闭包

闭包的概念[1]：闭包就是能够读取其他函数内部变量的函数 。

先来个栗子1：

```javascript
function makeFunc () {
    var name = "Chrome";
    function display () {
        console.log(name);
    }
    return display;
}

var func = makeFunc();
func(); // Chrome，而不是undefined
func = null; // or undefined，释放内存避免内存泄漏
```

### 词法作用域[2]

​	根据上面的栗子，首先需要了解一个概念---词法作用域，词法作用域是指一个变量在源码声明中的位置作为它的作用域，同时嵌套的函数可以访问到其外层作用域中声明的位置。

​	而栗子1中的display的词法作用域是在makeFunc内，所以他可以访问name，当它作为结果输出的时候，输出的其实是一个闭包，`闭包由函数和它的词法环境组成`，这个环境是指`函数创建时`，他可以访问的所有变量。



### 闭包的应用场景

#### 回调

栗子2[2]

```html
<html>
    <body>
        <p>Some Paragraph Text</p>
        <h1>Some Heading 1 Text</h1>
        <h2>Some Heading 2 Text</h2>
        
        <a href="#" id="size-12">12</a>
        <a href="#" id="size-14">14</a>
        <a href="#" id="size-16">16</a>
    </body>
    <script>
        function makeSizer (size) {
            return function () {
                document.body.style.fontSize = size + 'px';
            }
        }
        
        document.getElementById('size-12').onclick = makeSizer(12);
        document.getElementById('size-14').onclick = makeSizer(14);
        document.getElementById('size-16').onclick = makeSizer(16);
    </script>
</html>

```

​	在这个栗子2中，每个<a>绑定的是一个makeSizer返回的闭包，这个闭包包括一个赋值的size和function。当click事件发生时就会去触发调用这个闭包函数。



#### 模拟私有方法

栗子3[2]：

```javascript
var counter = (function () {
    var privateCounter = 0;
    function changeBy (val) {
        privateCounter += val;
    }
    return {
        increment: function () {
            changeBy(1);
        },
        decrement: function () {
            changeBy(-1);
        },
        value: function () {
            return privateCounter;
        }
    };
})();

console.log(counter.value()); // 0
counter.increment()
console.log(counter.value()); // 1
counter.decrement();
console.log(counter.value()); // 0

```

​	在栗子3中，return返回的闭包函数可以访问变量privateCounter和方法changeBy()，但是counter却只能访问返回的三个闭包函数。这样就达到了私有变量和私有方法的效果。这种方法也称为模块。



#### 和this调用的区别[1]

```javascript
// code 1
var name = "The Window";
var object = {
	name : "My Object",
	getNameFunc : function(){
		return function(){
　　　　　　　　return this.name;
　　　　　};
	}
};
alert(object.getNameFunc()());

// code 2
var name = "The Window";
var object = {
	name : "My Object",
	getNameFunc : function(){
        var that = this;
		return function(){
　　　　　　　　return that.name;
　　　　　};
	}
};
alert(object.getNameFunc()());
```

​	结果是code1输出"The Window"，code2输出"My Object"。

但是如果指向闭包的引用没有消除的话，闭包会一直在内存中，如果过多使用闭包，会造成内存不足。从而引发内存泄漏的问题。解决的办法是将闭包的引用指向null或者undefined。

### 参考资料

[1]	http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html

[2]	https://juejin.im/entry/58f424d5570c3500563d7541



# 原型链

提到了闭包就不得不提作用域，为什么闭包能够访问作用域外的变量就是因为原型链的存在。关于原型链必须记住以下几点：

1. 每个引用都有_ _ proto _ _属性，它们都指向他们的构造函数的prototype属性上
2. 每个函数都有prototype属性
3. 当试图得到一个没有对象的属性时，如果没有找到，就会从他的_ _ proto _ _属性上即它的构造函数的prototype属性上找，直到找到或者遇到null为止。



### 继承

继承方式主要讲三种方式，一是组合式继承，二是寄生组合式继承，三是ES6的Class。

1. 组合继承的优点是融合了原型链继承和构造函数的优点，缺点是会执行两次Parent函数。

   ```javascript
   function Parent (name, age) {
     this.name = name;
     this.age = 23;
   }
   
   Parent.prototype.say = function () {
     console.log(`Name: ${this.name}, Age: ${this.age}`);
   }
   
   function Child (name, age, job) {
     Parent.call(this, name, age); // 1.构造函数继承属性
     this.job = job;
   }
   
   Child.prototype = new Parent(); // 2.原型链继承方法或者说是得到Parent方法的使用权
   Child.prototype.constructor = Child; // 是为了让对象的constructor指向对象的构造函数
   
   Child.prototype.say = function () {
     console.log(`Name: ${this.name}, Age: ${this.age}, Job: ${this.job}`);
   }
   
   var child = new Child('barney', 30, 'business man');
   child.say();
   ```

2. 寄生组合继承，优点就是只会调用一次父级函数。

   ```javascript
   function Parent (name, age) {
     this.name = name;
     this.age = age;
   }
   
   Parent.prototype.say = function () {
     console.log(`Name: ${this.name}, Age: ${this.age}`);
   }
   
   function Child (name, age, job) {
     Parent.call(this, name, age);
     this.job = job;
   }
   
   var Middle = function () {} // 与组合继承不同的是使用了一个函数作为中转
   Middle.prototype = Parent.prototype; 
   // 等价于Middle.prototype = Object.create(Parent.prototype);
   Middle.prototype.constructor = Child; // 是对象的constructor指向它的构造函数
   Child.prototype = new Middle();
   
   Child.prototype.say = function () {
     console.log(`Name: ${this.name}, Age: ${this.age}, Job: ${this.job}`);
   }
   
   var parent = Parent('Ted', 29);
   var child = new Child('barney', 30, 'business man');
   child.say();
   ```

3. Class，这个方法是最好的。可以调用父级的同名函数。

   ```javascript
   class Person {
     constructor (name, age) {
       this.name = name;
       this.age = age;
     }
     say () {
       console.log(`Name: ${this.name}, Age: ${this.age}`);
     }
   }
   
   class Employee extends Person {
     constructor (name, age, job) {
       super(name, age);
       this.job = job;
     }
     say () {
       super.say();
       console.log(`Job: ${this.job}`);
     }
   }
   
   var e = new Employee('Barney', 29, 'Business Man');
   e.say();
   ```

   



# 数组

### 判断对象是否为数组的方法

```javascript
// way 1
arr instanceof Array; // 跨frame或者多个window情况下失效
// way 2
Object.prototype.toString(arr) === '[object Array]';
// way 3
Array.isArray(arr); // 有兼容性问题，可以使用PolyFill，代码如下
function arrayPolyFill (arr) {
    if(!Array.isArray) {
        return Object.prototype.toString(arr) === '[object Array]';
    }else {
        return Array.isArray(arr);
    }
}
```

### 数组去重

```javascript
// way 1
function unique (arr) {
    let arrNew = [];
    arr.forEach(item => {
        if(!arrNew.indexOf(item)+1) {
            arrNew.push(item);
        }
    });
    return arrNew;
}
// way 2
let arrNew = [...new Set(arr)];
```

### 数组排序

```javascript
let arr = [1,11,2,3,4];
// way 1
arr.sort(); // [1,11,2,3,4]，按字符排序

// way 2，从小到大排序
function compare1 (val1,val2) {
    return val1-val2;
}
arr.sort(compare1); // [1,2,3,4,11]

// way 3，从大到小排序
function compare2(val1,val2) {
    return val2-val1;
}
arr.sort(compare2);
```



# this指向

```javascript
// 第一个
var name = 'window'

var person1 = {
  name: 'person1',
  show1: function () {
    console.log(this.name)
  },
  show2: () => console.log(this.name),
  show3: function () {
    return function () {
      console.log(this.name)
    }
  },
  show4: function () {
    return () => console.log(this.name)
  }
}
var person2 = { name: 'person2' }

person1.show1() // 
person1.show1.call(person2) // 

person1.show2() // 
person1.show2.call(person2) // 

person1.show3()() // 
person1.show3().call(person2) // 
person1.show3.call(person2)() // 

person1.show4()() //
person1.show4().call(person2) // 
person1.show4.call(person2)() // 
------------------------------------------------------------------------------------------
// 第二个
var name = 'window'

function Person (name) {
  this.name = name;
  this.show1 = function () {
    console.log(this.name)
  }
  this.show2 = () => console.log(this.name)
  this.show3 = function () {
    return function () {
      console.log(this.name)
    }
  }
  this.show4 = function () {
    return () => console.log(this.name)
  }
}

var personA = new Person('personA')
var personB = new Person('personB')

personA.show1() // 
personA.show1.call(personB) // 

personA.show2() // 
personA.show2.call(personB) // 

personA.show3()() // 
personA.show3().call(personB) // 
personA.show3.call(personB)() // 

personA.show4()() // 
personA.show4().call(personB) // 
personA.show4.call(personB)() // 
```

this是在`运行的时候才决定指向`的。

第一个函数：

person1.show1()说明show()方法是由person1调用的，所以this指向person1。

person1.show1.call(person2)，此时调出person1的show方法指向了person2，所以this指向了person2

person1.show2()，因为字面量表示法不会产生新的作用域和箭头函数的this是继承自父级作用域的。所以person1.show2()中的this指向的是全局环境。箭头函数运行的时候没有生成新的作用域。所以this的指向无法改变。只能指向上一层作用域的this。

person1.show2.call(person2)，由于箭头函数并没有产生新的作用域。所以他的指向并不能通过call改变。所以this仍然指向了全局作用域。

person1.show3()()，person1.show3()返回的是一个函数。然后这个函数的执行是在全局作用域下。所以this指向window。

person1.show3().call(person2)，相当于person2调用了返回的函数。所以this指向person2。

person1.show3.call(person2)()，person2调用了person1里面的show3返回一个函数，这个函数在全局中调用，所以this指向window

person1.show4()()，show4中箭头函数的this是来自show4，而show4是在person1中调用的，所以show4的this指向person1，所以this指向person1

person1.show4(person2).()，show4是由person2调用的，所以，show4指向了person2，所以this指向person2。

person1.show4().call(person2)，show4是由person1调用的，而箭头函数的this是来自show4，所以犹豫this是在执行时就确定了。所以this的指向不能再通过call改变了。

第二个函数：在通过new生成对象的时候就已经将this绑定到生成的对象了，形成了新的作用域。

personA.show1()，personA调出自身的show1执行。所以this指向personA

personA.show1.call(personB)，personA调出自身的show1指向personB，然后再执行，所以this指向personB

personA.show2()，show2的this是指向的Person这个function的，而这个函数在new的时候已经执行了。由于this是在运行时决定指向的。所以箭头函数的this指向在当时已经确定了。就是指向的Person new出来的对象，所以this指向personA

personA.show2.call(personB) ，show2的this来自Person，Person在new的时候就已经运行了，将this指向了personA。所以此时通过call无法改变this的指向了。

personA.show3()()，personA.show3执行返回一个函数引用，这个引用在全局中执行，所以指向了window

personA.show3().call(personB)，这个引用指向了personB，所以this指向了personB。

personA.show3.call(personB)()，show3指向了personB，返回一个函数引用。这个引用在全局中运行。所以this指向了window

personA.show4()()，show4由personA调用并执行。所以show4的this指向personA，返回的箭头函数的this来自show4，所以this也指向personA

personA.show4().call(personB)，因为返回的箭头函数来自show4，show4的this指向在运行时已经确定指向了personA，所以答案是personA

personA.show4.call(personB)() ，show4有personA调用指向personB然后执行。所以show4的this指向了personB，所以箭头函数的this指向了personB。

#深copy、bind、call、apply模拟实现

深copy
```javascript
    function deepClone (obj) {
      if(typeof obj !== 'object'){
        return obj;
      }
      // typeof null 也是 'object'
      if (obj === null) {
        return null;
      }
      let objNew = obj instanceof Array ? [] : {};
      for (item in obj) {
        objNew[item] = typeof obj[item] !== 'object' ? obj[item] : deepClone(obj[item]);
      }
      return objNew;
    }
```
bind实现
```javascript
    Function.prototype.bind2 = function (context) {
    
        if (typeof this !== "function") {
          throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
        }
    	// self 指向了调用bind的函数
        var self = this;
        // args表示的调用bind函数时所传入的参数
        var args = Array.prototype.slice.call(arguments, 1);
    	// 作为中转的函数，
        var fNOP = function () {};
    
        var fBound = function () {
            // bindArgs是指作为返回值的fBound函数在外面的引用函数所传入的参数
            var bindArgs = Array.prototype.slice.call(arguments);
            // args.concat(bindArgs)表示最终调用函数时所传入的参数
            return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
        }
        fNOP.prototype = this.prototype; 
        fBound.prototype = new fNOP();
        return fBound;
    }
    var Foo = function (name,age) {
        console.log(this.value);
        console.log(name, age);
    }
    var obj = { value: 1 };
    var f = Foo.bind(obj, 'Barney'); // args === ['Barney']
    f(30); // bindArgs === [30], args.concat(bindArgs) === ['Barney', 30];
```
call实现
```javascript
    Function.prototype.call = function (context) {
      var context = context || window;
      context.fn = this;
      var args = [];
      for (var i = 1; i < arguments.length; i++) {
        args.push('arguments[' + i + ']');
      }
      var result = eval('context.fn(' + args +')');
      delete context.fn;
      return result;
    }
```
apply实现
```javascript
    Function.prototype.apply = function (context, arr) {
        var context = Object(context) || window;
        context.fn = this;
    
        var result;
        if (!arr) {
            result = context.fn();
        }
        else {
            var args = [];
            for (var i = 0, len = arr.length; i < len; i++) {
                args.push('arr[' + i + ']');
            }
            result = eval('context.fn(' + args + ')')
        }
    
        delete context.fn
        return result;
    }
```
参考资料：https://github.com/mqyqingfeng/Blog/issues/11

#类型判断和类型转换

####类型判断：类型判断typeof、instanceof、Object.prototype.toString()

typeof：返回一个string，能够识别string、number、boolean、undefined、object、function六种类型。

```javascript
typeof 'abc'; // 'string'
typeof 123; // 'number'
typeof undefined; // 'undefined'
typeof true; // 'boolean'
typeof null; // 'object'
typeof {value: 'a'}; // 'object'
var foo = function () {}
typeof foo; // 'function'
```

instaceof：用来测试一个对象在其原型链中是否存在一个构造函数的 prototype 属性。

Object.prototype.toString()可以用来判断大概14种类型

```javascript
// Object.prototype.toString.call(number)
var number = 1;          // [object Number]
var string = '123';      // [object String]
var boolean = true;      // [object Boolean]
var und = undefined;     // [object Undefined]
var nul = null;          // [object Null]
var obj = {a: 1}         // [object Object]
var array = [1, 2, 3];   // [object Array]
var date = new Date();   // [object Date]
var error = new Error(); // [object Error]
var reg = /a/g;          // [object RegExp]
var func = function a(){}; // [object Function]
```



#### 类型转换

强制转换有：Number、String、Boolean

Number:

```javascript
// 基本类型的转换
Number(324) // 324
Number('324') // 324
Number('324abc') // NaN
Number('') // 0
Number(true) // 1
Number(false) // 0
Number(undefined) // NaN
Number(null) // 0

// 对象的转换，除了单个元素的数组外都是NaN
Number({a: 1}) // NaN
Number([1, 2, 3]) // NaN
Number([5]) // 5
```

String：

```javascript
// 基本类型转换
String(123) // "123"
String('abc') // "abc"
String(true) // "true"
String(undefined) // "undefined"
String(null) // "null"

// 对象
var foo = function () {}
String(foo); // 'function () {}'
String({a: 1}); // "[object Object]"
String([1,2,3]); // "1,2,3"
```

Boolean：

```javascript
// 除了以下五个是false之外，其他的都是true
Boolean(undefined) // false
Boolean(null) // false
Boolean(0) // false
Boolean(NaN) // false
Boolean('') // false
```



隐式转换有：==、+、boolean判断，在判断语句中，引用类型的值都是true

boolean转换

```javascript
if(undefined) // false
if(null) // false
if(0) // false
if(NaN) // false
if('') // false
```

字符串转换：

```javascript
'5' + 1 // '51'
'5' + true // "5true"
'5' + false // "5false"
'5' + {} // "5[object Object]"
'5' + [] // "5"
'5' + function (){} // "5function (){}"
'5' + undefined // "5undefined"
'5' + null // "5null"
```

数值转换：

```javascript
'5' - '2' // 3
'5' * '2' // 10
true - 1  // 0
false - 1 // -1
'1' - 1   // 0
'5' * []    // 0
false / '5' // 0
'abc' - 1   // NaN
null + 1 // 1，null转换成数值是0
undefined + 1 // NaN，undefined转换成数值是NaN

// 一元运算符
+'abc' // NaN
-'abc' // NaN
+true // 1
-false // 0
```

x == y转换规律：

1.如果x和y类型不同，那么到14步；
//
//2-13步，为类型相同的比较
//
14.如果x是null，y是undefined，返回true；
15.如果x是undefined，y是null，返回true；
16.如果x是Number，y是String，将y转化成Number，然后再比较；
17.如果x是String，y是Number，将x转化成Number，然后再比较；
18.如果x是Boolean，那么将x转化成Number，然后再比较；
19.如果y是Boolean，那么将y转化成Number，然后再比较；
20。如果x是String或者Number，y是Object，那么将y转化成基本类型，再进行比较；
21.如果x是Object，y是String或者Number，将x转化成基本类型，再进行比较；
22.其他情况均返回false；

总的来说就是:

1. 如果x、y分别null和undefined的话，就返回true
2. 如果x、y分别是String、Boolean、Number的话就都转成Number再比较
3. 如果x、y有Object的话就转化成基本类型再比较
4. 如果x、y中有NaN，则返回false
5. 如果x、y有null，那么另一个如果不是undefined或者null的话就返回false



# 四种前端规范

### CommonJS

​	CommonJS退从一个文件就是一个模块，拥有单独的作用域。普通方式定义的变量、函数、对象都属于该模块内。通过 require 来加载模块，exports 和 modul.exports 来暴露模块中的内容。

​	CommonJS适合于服务端（比如NodeJS），因为CommonJS的模块加载时同步加载，在客户端可能因为网络问题导致迟迟无法加载完成，印象用户体验。而在服务端，由于加载是在本地本地磁盘读取。



###AMD

​	AMD（Asynchronous Module Definition）是一种异步加载模块的规范，RequireJS就是采用的这一规范，先定义所有依赖，加载完成后执行回调钩子。



###CMD

​	CMD (Common Module Definition), 是seajs推崇的规范，CMD则是依赖就近，和AMD一样，CMD也是异步加载。不同的是，AMD依赖前置，js可以方便知道依赖模块是谁，立即加载；而CMD就近依赖，需要使用把模块变为字符串解析一遍才知道依赖了那些模块，这也是很多人诟病CMD的一点，牺牲性能来带来开发的便利性，实际上解析模块用的时间短到可以忽略。 



### ES6

​	ES6推崇一个模块就是一个独立的文件。该文件内部的所有变量，外部无法获取。export 命令用于规定模块的对外接口。import 命令用于输入其他模块提供的功能。ES6 模块的设计思想是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。VueJS采用的就是ES6的模块规范。