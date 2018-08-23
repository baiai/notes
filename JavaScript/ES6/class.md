####定义 
类内部定义的所有方法都是不可枚举，this默认指向类的实例但是this的指向是在运行时确定的。所以还是要小心this的指向被改变。
```javascript
class Point(x,y){
	这就是构造函数，初始化值，new命令实例化对象时，会自动调用该方法，
	如果没有显示定义的话也会被默认添加一个空的constructor方法
	constructor(x,y){ 
		this.x = x;
		this.y = y;
	}
	方法之间没有逗号，方法不用加function关键字
	toString(){
		return '(' + this.x + ',' + this.y + ')' ;
	}
}

typeof Point;  // "function" 类本身就是函数，但是必须使用new才能调用

Point === Point.prototype.constructor
```

其实class里面的所有函数都是声明在class的prototype属性上Point等同于
```javascript
Point.prototype = {
	constructor(x,y){ this.x = x; thi.y = y; },
	toString(){ return '(' + this.x + ',' + this.y + ')' ; }
}
```
####创建 创建对象
```javascript
let point = new Point(1,2);
point.hasOwnProperty('x'); // true
point.hasOwnProperty('y'); // true
point.hasOwnProperty('toString'); // false
```
 没通过this显示定义的方法都是定义在prototype属性上的,
 私有变量用this定义，class的实例共享prototype上的方法
```javascript
point.__proto__.hasOwnProperty('toString'); // true
```

class表达式，就和函数的表达式声明是一样的，类本身也是一种函数
```javascript
	let MyClass = class Me(){ // Me才是类名，Me只在class内部使用，没用到的时候可以省略
		getClassName{ return Me.name; }
	}

	let init = new MyClass();
	init.getClassName(); // Me
```
####不存在变量提升，如果使用在声明和定义之前就会报错，
  主要是为了继承时父类在子类之前定义

####私有方法和属性
	方法要通过call()、apply()、和Symbol绑定
	属性可以在constructor里面定义编程私有属性
```javascript
	class Widget () {
	  foo (baz) {
	    bar.call(this, baz);  /*bar绑定到了*/
	  }
	}
	// 将私有方法放在模块外，有class内部的方法调用就相当于是私有方法
	function bar(baz) {
	  return this.snaf = baz;
	}
```
#### class静态方法
在方法名前加上static表明该方法和属性是静态方法和静态属性，会被子类继承，静态方法的this是指的类本身而不是Class new出来的实例。

```javascript
//说白了就是
Class Foo {
    constructor () {
        this.name = 'constructor';
    }
    static classMethod () {
        return 'hello';
    }
    static className () {
        return this.name; // 'Foo' 而不是 'constructor'
    }
}
Foo.name = 'Foo'
// 相当于
Class Foo {
    
}
Foo.classMethod = function () { return 'helo' }
```



####class的继承
通过关键字：class Children extends Parent{ }
Children继承了Parent的所有属性和方法，通过super.调用

```javascript
		class ColorPoint extends Point {
		  constructor(x, y, color) {
		    super(x, y); // 调用父类的constructor(x, y)
		    this.color = color;
		  }

		  toString() {
		    return this.color + ' ' + super.toString(); // 调用父类的toString()
		  }
		}
```

子类在constructor里面"`必须`"调用super方法获取父类的this对象，因为子类没有自己的this对象，而是继承父类的this在对其进行加工，再调用super函数之后this才指向子类。super有两个身份，一个是父类的构造函数只能在constructor中使用，另一个就是作为父类的实例对象在普通函数中使用，但是如果实在静态函数中，super指向的是父类，而不是父类的实例对象。

```javascript	
//a. Object.getPrototypeOf() 获取父类,此方法可以判断一个类是否是另一个类的子类
Object.getPrototypeOf(ColorPoint) === Point // true
	
//b. new.target new的调用点，即使用new创建对象的是哪一个类
class A {
	constructor() {
		console.log(new.target.name);
	}
}
class B extends A {
	constructor() {
		  super();
	}
}
new A() // A
new B() // B
```