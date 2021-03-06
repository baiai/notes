面试的时候，很多情况并不是我们不知道这么个东西，而是我们很难将它讲解清楚。从而导致面试官觉得我们并没有把他掌握好。所以，这个面试笔记事宜对话形式的来写的。尽量写的口语化。

### 问：对this指针的理解？

答：一开始，我对this指针也很迷糊，直到后来看了一本凯尔辛普森的《you don't know js》之后才有了一些了解。首先this指针的指向是在运行时确定的，而不是在声明的时候，明白了这一点很多情况下，this的只想问题就很明确了。

其次，凯尔将this的绑定方式分为了四种，默认绑定、明确绑定、硬绑定和new绑定。默认绑定是说当前函数在哪个作用域执行，那么this就只想那个作用域。明确绑定则是看谁调用了这个函数，那么this就指向谁。硬绑定则是说用apply、call、bind去改变this的指向。new绑定是通过new一个function的对象去改变this的指向。

凯尔将四种绑定分了优先级，从高到低分别是new绑定 > 硬绑定 > 明确绑定 > 默认绑定。

### 问：对闭包的理解？

答：JavaScript中闭包的话，我引用一下《深入浅出NodeJS》的作者朴灵在书中提到的概念：闭包是指实现外部作用域访问内部作用域中的变量的方法。比如说，变量C指向函数A返回的函数B，函数B用到了函数A中的变量。如果执行C的话，是可以访问到A中的变量的。所以我们通常使用闭包去模拟私有变量和方法。

不过闭包用多了也会造成内存泄漏的问题，因为闭包一直有一个对象在引用它的话，那闭包以及闭包所在的词法作用域就不会被回收，已知占用内存。所以如果我们不再使用闭包的话，就要将闭包的引用指向null或者undefined。再由垃圾回收机制去释放内存。

###问：垃圾回收机制？

答：JavaScript中的垃圾回收机制是自动回收的一种回收机制，采用的是标记-清除的算法。每个一段时间，标记清除法会先将在内存中的所有对象标记，然后从根部开始寻找被引用的对象并清除他们的标记。之后，还被标记的对象就视为被回收的对象。

### 问：原型链

答：原型链是这样的，每个对象都有一个proto属性指向它的构造函数的prototype属性，而每个函数都有一个prototype的属性。当我们访问一个对象的属性时，如果没有找到的话，那么就会去这个对象的构造函数的prototype上找，直到找到位置或者遇到null为止。这就构成了一个原型链。

### 问：继承

红宝书中提到了6种继承方式，但是尼古拉斯比较认可的是组合继承和寄生组合继承，再加上ES6的class继承。组合继承的优点是继承了原型链继承和构造函数的优点，但是需要执行两次父级构造函数，第一次是在子构造函数中，第二次是在将子构造函数的prototype指向父构造函数的对象。而寄生承组合继承解决了这个问题，它是声明了一个中间函数，将中间函数的prototype指向父构造函数的prototype。再将子构造函数指向这个中间函数的对象。class继承需要注意的就是子类是没有this指针的，所以一定要调用super函数去继承父类的this指针。