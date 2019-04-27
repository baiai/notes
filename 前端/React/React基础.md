# 1. Virtual DOM

## 1. 1问题的提出

浏览器渲染出的一个表格如表1-1所示，当表1中排序方式发生变化的时候，浏览器的常规操作是将整个表按照新的排序方式重新渲染。但是当表格很大的时候，可能重新排序只需要变动少数的列表即可，此时所有列表都重新渲染对浏览器的消耗会很大，而且浪费。那怎样才能避免消耗，做到性能最优呢？ 

![表格排序](file:///C:/Users/baiai/AppData/Local/Packages/Microsoft.Office.OneNote_8wekyb3d8bbwe/TempState/msohtmlclip/clip_image001.png) 

## 1.2 Virtual DOM的概念

为了避免DOM树的渲染消耗，做到`按需渲染页面`。React提出了Virtual DOM的概念。`Virtual DOM是通过JavaScript中的对象模拟DOM元素和DOM元素之间的嵌套关系`，当页面需要重新渲染的时候，先对比两个新旧Virtual DOM之间的差异，将变化的节点标记出来。最后只需要渲染标记的节点，从而达到按需加载的目的。 

## 1.3 Diff算法

为了对比出两个Vitual DOM之间的差异，React加入了Diff算法。Diff算法分为三个部分：Tree Diff、Component Diff、Element Diff三种。 

### 1.3.1 Tree Diff

React认为DOM跨层级的移动操作很少，少到可以忽略不计。所以，React通过updateDepth()对Virtual DOM树进行层级控制，只会对如图1-1所示的相同颜色方框内的DOM节点进行比较，即同一父节点下的子节点。当新旧Virtual DOM树对比后，发现节点不存在就会删除该节点及其所有子节点，不会进行下一步比较。这样就可以值遍历一遍Virtual DOM树，就完成整个DOM树的比较，对比的事件复杂度就是O(n)。

如果真的出现DOM跨层级操作，如图1-2所示。那么React的Diff执行情况为：创建A->创建B->创建C->删除A。但是这样的操作中，以A为根节点的树被重新创建，是会影响React的性能的操作，所以React官方建议不要进行DOM节点跨层级操作。

![图1-1](C:\Users\baiai\AppData\Local\Temp\1556197455787.png)

![图1-2](C:\Users\baiai\AppData\Local\Temp\1556197462395.png)

### 1.3.2 Component Diff

React是基于组件构建应用的，对于组件间的比较采用的策略是高效、简洁。比较采用以下三个策略：

- 如果是同类型组件，则继续比较Virtual DOM树；
- 如果不是同类型组件，则认为该组件为Dirty Component，从而替换整个组件下的所有子节点；
- 对于同类型组件，有可能Virtual DOM没有任何改变。如果能够确切的知道这点那可以节省大量的 diff 运算时间，因此，React允许用户通过shouldComponentUpdate()判断组件是否需要进行diff。

### 1.3.3 Element Diff

React在Element Diff时，为同一层的节点添加了唯一key进行区分。在新旧节点集合进行差异化对比的时候，如果新老集合中有相同的节点，无需进行删除和创建，只移动节点位置就可以了。

有待详解：[React 源码剖析系列 － 不可思议的 react diff](https://zhuanlan.zhihu.com/p/20346379)

# 2. jSX语法

## 2.1 全局变量React和ReactDOM

在react中，有两个重要的全局变量React和ReactDOM。React是负责创建需要渲染的组件和虚拟DOM元素对象，而ReactDOM则是负责将React创建的组件和虚拟DOM元素对象渲染到HTML文档中。

例子1：以下是将index.js文件中的创建的虚拟DOM元素创建到index.html文件中的`id="app"`的元素下：

```javascript
// index.js
import React from 'react'
import ReactDOM from 'react-dom'

// 创建虚拟DOM元素
/*
** React.creatElement()的参数解释
** 1. 第一个参数是标签类型
** 2. 第二个参数是标签的属性
** 3. 第三个到第n个参数是标签中的子节点
*/
let myh1 = React.creatElement('h1', {id: 'mydiv', title: 'ba'}, 'this is a h1 tag')
/*
** ReactDOM.render()的参数解释
** 1. 第一个参数是需要渲染的DON元素对象
** 2. 第二个参数是该虚拟DOM元素对象需要渲染到哪个元素下
*/
ReactDOM.render(myh1, document.getElemntById('app'))
```

```html
<!-- index.html -->
<div id="app"></div>
<script src="./index.js"></script>
```

## 2.2 JSX

2.1中的例子，有一个明显的缺点就是，每次都需要通过React.createElement()去创建虚拟DOM元素。这样在庞大的页面中，会显得十分冗余。所以，React支持了一种叫JSX的语法，意思就是可以在js可以使用XML代码。如例子2的代码：

```javascript
// 例子2
let myh1 = <h1>this is a h1 tag </h1>
ReactDOM.render(myh1, document.getElementById('app'))
```

### 2.2.1 启动JSX语法

需要通过babel将js中的XML代码转换成React中的虚拟DOM元素对象

### 2.2.2 JSX中渲染变量

在JSX语法中，可以将JS代码放入`{}`中，这样就可以渲染变量值到HTML中了。如例子3所示，分别渲染了数字、字符串、bool值、属性绑定值、jsx元素、jsx元素数组、以及将普通字符数组转换为jsx数组并渲染到页面上。

```javascript
import React from 'react'
import ReactDOM from 'react-dom'

let num = 1
let str = 'react is awesome'
let boolValue = true
let className = 'react'
let h2 = <h2> this is a h2 tag.</h2>
// 数组中需要注意key，如同Vue中v-for中的key一样
let arrH3 = [
    <h3> this is a h3 tag.</h3>,
    <h3> this is another h3 tag.</h3>
]
// 将普通字符数组转换为JSX数组
let arrStr = ["1","2","3"]

ReactDOM.render(
	{num + 1}
    <hr />
    {str}
    <hr />
    {boolValue.toString()}
    <hr />
    <h1 class="{className}"> this is a h1 tag.</h1>
    <hr />
    {h2}
    <hr />
    {arrH3}
    <hr />
    {
    arrStr.map(item => <h4>number {item}</h4>)
    }
)
```

