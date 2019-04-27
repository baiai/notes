# 块级作用域let

```javascript
if(true){

       let fruit = ‘apple’;

}

consoloe.log(fruit);//会报错，因为let只在if{ }的作用域有效，也就是块级作用域

```



# 常量const

```javascript
const fruit = ‘apple’;//其实就是fruit这个变量已经指定了apple这个位置，二次赋值其实就是改变他的引用，所以会报错，
console.log(fruit);

const fruit = ‘lemo’;
console.log(fruit);//会报错，const定义的变量不能二次定义，而且const也是块级作用域，不会变量提升
const arr = [ ];
arr.push(‘apple’);
arr.push(‘lemo’);
arr = [ ];//会报错
```

# 解构数组

```javascript
function breakfast(){
   return [‘cake’,’coffee’,’apple’];
}

let [dessert,drink,fruit] = breakfast();//会将dessert = ‘cake’,drink = ‘coffee’,fruit = ‘apple’,这就是数组解构，会将对应下标的值赋值

console.log(dessert,drink,fruit);
```
# 解构对象
```javascript
function breakfast(){
	return  { dessert:’cake’,drink:’coffee’,fruit:’apple’ };
}

//解析对象主要也是用于赋值，对象之间的赋值
let { dessert1:dessert, drink1:drink, fruit1:fruit } = breakfast();
console.log(dessert, drink, fruit );
```
# 模板字符串
```javascript
let dessert = ‘cake’, drink = ‘coffee’;
let breakfast = “今天的早餐是” + drink + “和” + cake;
console.log(breakfast);
//字符串模板使用变量要用${ 变量名 }来表示，常用语HTML的输出，这样` content `，content的内容的格式都不会变化
let breakfast_1 = `今天的早餐是${cake}和${coofee}`;
console.log(breakfast_1);

// 带标签的模板字符串

let dessert = ‘cake’,drink = ‘coffee’;

let breakfast = kitchen`今天的早餐是${ dessert }和${ drink }`;

function kitchen(strings,…values){//kitchen是标签，再以标签名做函数名，里面的内容就可以作为参数传递过来了

       //console.log(strings);

       //console.log(values);

       //strings里存放的是breakfast里面的字符串，values存放的是变量，strings和values都是数组

       let result = ‘’;

       for(let i = 0;I < values.length;i++){

              result += strings[i];

              result += values[i];

       }

       result += strings[strings.length -1];

       return result;//result返回的就是kitchen的那一段话

}
```
# 判断字符串里是否包含其他字符串
```javascript
let str = “今天吃的是apple”;

str.startWith(‘今天’);//返回true，判断str是否是以今天开头

str.endsWith(‘le’);//true,判断str是否是以le结尾

str.includes(‘apple’);//true，判断str是否含有apple这个字符串
```
# 默认参数
```javascript
function breakfast(dessert = “cake”,drink = “coffee”){//设置默认参数

​       return `${dessert} ${drink}`;

}

console.log( breakfast() );//cake coffee
```
# 展开操作符-Spread（…）
```javascript
let fruits = [‘apple’,’lemo’], foods = [‘cake’,…fruits];

console.log(fruits);  //返回的是数组[‘apple’,’lemo’]

console.log(…fruits);  //返回的两个字符串apple 和 lemo

console.log(foods);  //[‘cake’,’apple’,’lemo’];
```
# 剩余操作符(还是…)
```javascript
function breakfast(dessert,…foods){//多出来的参数都会放在foods里面

​       console.log(dessert,foods);

}

breakfast(‘cake’,lemo’,’coffee’);//打印的是cake,[‘lemo’,’coffee’]，因为lemo和coffee都放在了foods这个数组里
```
# 解构参数
```javascript
function breakfast(dessert,drink,{location,restaurant} = { } ){//其实就是利用解构对象来做参数传递
	console.log(dessert,drink,location,restaurant);
}

breakfast(‘cake’,’coffee’,{ location:”wuhan”,restaurant:”baijingyuan” } );
```
# 函数的名字-name属性
```javascript
function foo1(){ }

let foo2 = function(){ }

let foo3 = function foo4(){ }

console.log(foo1.name,foo2.name,foo3.name);//foo1,foo2,foo4
```
# 箭头函数
```javascript
箭头函数里的this指向是绑定在父级作用域上下文中的，并没有产生新的作用域。

// 函数名             参数              方法实现

let breakfast = (dessert) => { return dessert; };

 

翻译成普通函数：

var breakfast = function breakfast(dessert){

​       return dessert;

}

# 对象表达式

let dessert = ‘cake’ , drink = ‘coffee’;

//全写

let food = {

​       dessert:dessert,

​       drink:drink,

​       breakfast:function(){ }

}

//简写

let food = {

​       dessert,

​       drink,

​       breakfast(){ }

}
```
# 对比两个值是否相等Object.is(var1,var2)

# 把对象的值复制到另一个对象Object.assign() 是深copy
```javascript
let breakfast = { };

Object.assign( breakfast,{ dessert:’cake’,drink:’coffee’ } );

console.log(breakfast);//dessert:’cake’,drink:’coffee’
```


# 设置对象的prototype
```javascript
//Object.getPrototypeOf()

//Objeect.setPrototypeOf()

let breakfast={

​       getDrink(){

​              return ‘coffee’;

​       }

};

let dinner = {

​       getDrink(){

​              return ‘beer’;

​       }

};

 

let drink = Object.create(breakfast);

drink.getDrink();//coffee

console.log( Object.getPrototypeOf(drink) === breakfast);//true

Object.setPrototypeOf(drink,dinner);

drink.getDrink();//beer

console.log(Object.getPrototyprOf(drink) === dinner);//true

 

```

 

# __proto__
```javascript
let breakfast={

       getDrink(){

              return ‘coffee’;

       }

};

let dinner = {

       getDrink(){

              return ‘beer’;

       }

};

 

let drink ={

       __proto__:breakfast,

}

drink.getDrink();//coffee

console.log( Object.getPrototypeOf(drink) === breakfast);//true

drink.__proto__ = dinner;

drink.getDrink();//beer

console.log(Object.getPrototyprOf(drink) === dinner);//true
```


# super
```javascript
let breakfast={

​       getDrink(){

​              return ‘coffee’;

​       }

};

let dinner = {

​       getDrink(){

​              return ‘beer’;

​       }

};

 

let drink = {

​       __proto__:breakfast,

​       getDrink(){

​              return super.getDrink() + ‘,milk’;

​       }

}

drink.getDrink();//coffee,milk
```


# 迭代器

 

# 生成器Generators
```javascript
function* chef(){

​       yield ‘apple’; //yield就是需要迭代的变量

​       yield ‘lemo’;

}

function* foods(foodArr){

​       for(var i = 0;I < foodArr.length;i++){

​              yield foodArr[i];

​       }

}

let wanghao = chef();

console.log(wanghao.next());//apple

console.log(wanghao.next() );//lemo

console.log(wanghao.next() );//undefined
```
# class
```javascript
class Chef {

​       constructor(food){

//当使用new去生成一个对象的时候会自动调用该方法

//我们也可以使用该方法对这个类进一些初始化

​              this.food = food;

​       }//注意此处没有逗号

​       cook(){

​              console.log(this.food);

​       }

}

 

let wanghao = new Chef(‘apple’);

wanghao.cook();

```

# get与set
```javascript
class Chef {

​       constructor(food){ 

​              this.food = food;

​              this.dish = [ ];

​       } 

​       

​       get menu(){

​              return this.dish;

​       }

​       set menu(dish){

​              this.dish.push(dish);

​       }

​       cook(){

​              console.log(this.food);

​       }

}

 

let wanghao = new Chef();

wanghao.menu = ‘apple’;

wanghao.menu = ‘lemo’;//会自动调用set方法

console.log(wanghao.menu);//会自动调用get方法

```

# 静态方法static
```javascript
//不需要实例化类就可以使用的方法

class Chef {

​       constructor(food){ 

​              this.food = food;

​              this.dish = [ ];

​       } 

​       

​       get menu(){

​              return this.dish;

​       }

​       set menu(dish){

​              this.dish.push(dish);

​       }

​       cook(){

​              console.log(this.food);

​       }

​       static cook(food){

​              console.log(food);

​       }

}

 

Chef.cook(‘apple’);//不需要class的实例化对象，直接用类名就可以调用static方法

```

# 继承extends
```javascript
class Person {

​       constructor(name,birthday){

​              this.name = name;

​              this.birthday = birthday;

​       }

​       intro(){

​              return `${ this.name },${ this.birthday }`;

​       }

}

class Chef extends Person {

​       constructor(name,birthday){

​              super (name,birthday);

​       }

}

 

let wanghao = new Chef(‘王浩’,’1995-07-11’) );

wanghao.intro();//王浩，1995-07-11
```
# Set
```javascript
let dessert = new Set(‘apple’,’lemo’);//此时dessert = { ‘apple’,’’lemo’ }

dessert.add(‘ice’);//此时dessert = {'apple’,’lemo’,’ice’ }

dessert.add(‘ice’);//Set里面的元素不允许有重复的，所以此时的dessert = { ‘apple’,’lemo’,’ice’}

console.log(dessert.size);// 3,size是deseert的长度

console.log(dessert.has(‘apple’);//true，has(str)是判断是否有这个元素

console.log(dessert.delete(‘apple’);//delete(str)是删除元素str这个元素

dessert.forEach(dessert => {

​       console.log(dessert);

});//forEach()是循环里面的元素

dessert.clear();//清空里面的元素

 
```
# Map
```javascript
let food = new Map();

let fruit = { }.cook =function(){ },dessert = ‘甜点’;

food.set(fruit,’lemo’);// Map{ Object {} => ‘lemo’

food.set(dessert,’pie’);//‘甜点’ = ‘pie’

food.size;//2

food.get(fruit);//lemo

food.delete(dessert);

food.has(dessert);//false

food.forEach((value,key) =>{

​       console.log(`${ value },${ key}`);

});

food.clear();
```
# Moudle(模块)
```javascript
// Person.js

let name = ‘王浩’,year = 24;

export { name, year as old };//重命名year为old

//Chef.js

import { name,old as age } from ‘./Person.js’;//文件所在的相对路径，重命名old为age

//或者 import * as person from ‘./Person.js’;peroson.js里面所有的导出都会放chef对象中

console.log(name,age);

```

# 导出与导入
```javascript
严格模式

·    变量必须声明后再使用

·    函数的参数不能有同名属性，否则报错

·    不能使用with语句

·    不能对只读属性赋值，否则报错

·    不能使用前缀 0 表示八进制数，否则报错

·    不能删除不可删除的属性，否则报错

·    不能删除变量delete prop，会报错，只能删除属性delete global[prop]

·    eval不会在它的外层作用域引入变量

·    eval和arguments不能被重新赋值

·    arguments不会自动反映函数参数的变化

·    不能使用arguments.callee

·    不能使用arguments.caller

·    禁止this指向全局对象

·    不能使用fn.caller和fn.arguments获取函数调用的堆栈

·    增加了保留字（比如protected、static和interface）

export命令 

      是将模块中定义的变量引出的接口，有两种实现方式

1.      直接在export上声明和定义

export let function foo() { console.log(‘test’); };
 但是如果先定义在导出回报错

      function foo() { console.log(‘test’); };
        export foo;  //回报做，没有接口，而是直接将变量输出

2.      将输出的变量放在一个对象里

export { foo };

import命令

      命令形式：import { 这里放的是与引入模块的接口相同的名称 } from ‘引入模块的路径’

      import { area, length } from ‘./circle’;

      注意：必须是大括号，{ } 里面的内容就是会引入的接口，如果没有写，就如发获取对应的接口。可以使用as为获取的接口重命名在自己的模块使用，接口是只读的，不能更改。
      也可以用*把所有的接口都引入，这样可以不用大括号包裹

      imprt * as circlr from ‘./circle’; // 定义接口的引用为circle，

      export default 和 import混合使用，import可以省略{ }。

跨模块常量

       如果想要声明一些去全局常量，可以被多个模块使用的话，就将这些模块定义在一个模块里，然后再由最顶层模块去import，这样其他模块就是可以使用了。

import( ) 动态同步加载模块

       由于import命令是在编译之前就已经加载好了的，所以想要在编译的时候进行动态加载模块是做不到的，于是就有了import( )动态加载模块函数，他的参数和import命令一样，都是模块的路径，不过可以js对象以及字符串模板

       适用于：动态加载、按需加载、条件加载、动态模块路径


```