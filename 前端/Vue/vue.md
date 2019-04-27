[vue中的选项](#vue中的选项)

[Vue生命周期](#Vue生命周期)

[全局注册和局部注册](#全局注册和局部注册)

[条件渲染（v-if，v-show）](#条件渲染（v-if，v-show）)

[列表渲染（v-for）](#列表渲染（v-for）)

[事件处理（methods）](#事件处理（methods）)

[表单输入绑定（还不熟悉）](#表单输入绑定（还不熟悉）)

[组件](#组件)

[Prop](#Prop)

[自定义事件](#自定义事件)

[混入](#混入)

[动画](#动画)

[数据双向绑定](#数据双向绑定（proxy）)


# Vue

<h1 id = "vue中的选项">vue中的选项</h1>

​	1	`data`顾名思义，只能是数据，不推荐观察拥有状态行为的对象

​	2	`props`是用来接受父组件传递过来的数据，props可以是数组或者对象并提供类型检测

​	3	`computed`，所有的getter和setter的this都自动绑定为vue实例，可以缓存

​	4	`methods`，可以通过vm实例访问这些方法，自动绑定到vm实例上

​	5	`watch`，watch监听的是data中的数据，在数据变化时，执行异步或开销较大的操作时比computed好，watch还有handler（值变化时的处理方法）、immediate（在watch声明是要不要执行handler）、deep属性（是否深度watch，即内部属性也watch）。

```javascript
var vm = new Vue({
    data:function(){ // 组件的data必须是函数，这样才能保证组件的每个实例返回的是相互独立的对象
        return {
            a:1
        }
    },
    watch:{
        a:function(val,oldVal){ // val 是改变之后的值 oldVal是改变之前的值
        }
    }
})
```

`*`这些实例都最好`不要用`箭头函数，因为箭头函数会导致`this`指向的不是本实例，而是父级作用域或者undefined

### <h1 id = "Vue生命周期">Vue生命周期</h1>

`*`Vue中所有的生命周期钩子都`不可以`用箭头函数，以下周期是按时间顺序，没有全部列出，只列了重点。

####created

​	在实例已经创建完成后立即调用，此时data、computed、methods、watch、event callback都已经完成，但是挂载阶段还没有开始，所以$el属性不可见，而且DOM元素也没有渲染。



#### mounted

​	元素挂载到实例上之后调用该钩子，但是mounted不会承诺所有的子组件一起被挂载，如果需要等到所有视图都渲染完毕，可以只用vm.$nectTick()替换mounted，具体实现如下：

```javascript
mounted: {
    this.$nextTick(function () {
        // code that will run only after the entire view has been randered
    })
}
```



#### nextTick

​	官方文档：在下次DOM更新循环之后执行延迟回调，在修改数据之后立即使用这个方法，获取更新后的DOM

​	在Vue的生命周期钩子中的created和mounted中，如果需要在所有视图渲染完毕之后再进行操作就可以使用nectTick，使用方法如mounted中的栗子。此外，nextTick在2.1.0版本中还新增Promise回调。使用方法如下：

```javascript
created: {
    this.$nextTick().then(function () {
        // code that will run only after the entire view has been rendered
    })
}
```



<h1 id = "全局注册和局部注册">全局注册和局部注册</h1>

全局注册：

```javascript
// main.js 文件
import componentA from './components/componentA.vue';

// 如果是component-A的话，引用组件必须也是<component-A>
 // 如果是componentA的话，引用组件可以是<componentA>或者<component-A>
Vue.component('componentA',componentA); 

```
局部注册：`*`局部注册组件在其子组件中不可使用

```javascript
// 某个组件内
import componentA from './components/componetA.vue';

export default{
    components:{
		componentA, // 这是简写 全写：'componentA' : componentA
    }
}
```
`*`全局组件意味着即使已经不再使用这个组件，他仍会被包含在最终构建的结果中，造成用户下载的js无谓增加，这个时候最好使用局部组件

### <h1 id = "Class与Style绑定 （v-bind)">Class与Style绑定 （v-bind)</h1>

就是一些绑定的方法，没什么的太多的注意事项，用了`v-bind就表示里面的值不是字符串而是表达式`了

### <h1 id = "条件渲染（v-if，v-show）">条件渲染（v-if，v-show）</h1>

#### v-if

v-if、v-else-if、v-else，`注意`v-else必须紧跟前两位的后面。

用key管理可复用元素：

因为模板A与模板B不可能同时存在，当vue渲染元素时，会重复使用里面相同的元素，而不是重复渲染。可是当元素被添加了key属性之后，就具有唯一标识，vue渲染时不会复用。

```html
<!--模板A与模板B的label标签使用的是同一个元素，而input使用的是不同的元素>
<!--模板A-->
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username" key="username-input" type = "text">
</template>
<!--模板B-->
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address" key="email-input" type = "email">
</template>
```

#### v-show

与v-if相比，v-show其实就是display：none，虽然不显示，但是还是会被渲染，可用于频繁切换到模板。

v-show会有更高的初始渲染开销，v-if有更高的切换开销。

###  <h1 id = "列表渲染（v-for）">列表渲染（v-for）</h1>

`v-for优先级高于v-if`

当vue更新以渲染过的列表是，默认采用`就地复用`策略，如上面的v-if不太一样，如果数据项的顺序变化，vue不会移动DOM元素来匹配数据项的顺序。这种默认模式值适用于不依赖组件状态或临时DOM状态（图表单输入值）的列表渲染输出。如果想要DOM的顺序随着数据项一起移动的话，就需要给这些元素动态绑定key

```html
<div v-for = "item in items" :key = "item.id">
    <!--内容-->
</div>
```

由于javascript的限制，`vue不能检测变动的数组和对象`。比如

```javascript
// 这是数组
var vm = new Vue({
    data:function(){
        return {
            items:['a','b','c'];
        }
    }
});
vm.items[1] = 'd'; // 不是响应性的
vm.items.length = 2; // 不是响应性的

```

为了解决这个问题，可以使用Vue.set和vm.items.splice(newLength)去解决。

```javascript
Vue.set(vm.items,indexOfItems,newValue);
// 或者可以使用vm.$set()，这是Vue.set全局的一个别称
vm.$set(items,indexOFItems,newValue);

vm.items.splice(2) // 这样就可以响应式的监控数组的变化了。会触发视图更新
```

```javascript
// 这是对象
var vm = new Vue({
    data：function(){
        return {
            obj:{name:'barney'}
        }
    }
});
vm.obj.age = 27; // 是不会响应的，即视图不会更新

// 解决方案
vm.$set(obj,age,27); // 或者 Vue.set(vm.obj,age,27);

// 如果想要添加多个属性
vm.obj = Object.assign({},vm.obj,{age:27,job:'front-end'});
```



### <h1 id = "事件处理（methods）">事件处理（methods）</h1>

形式：

```html
<div v-on:click = "deal(args)"></div>
<script>
    new Vue({
        methods:{
            deal(args){
                console.log('done');
            }	
        }
    });
</script>

```

事件修饰符：.stop、.prevent、.capture、.self、.once、.passive

```html
<!-- 如果子元素不添加stop修饰符的话，子元素的dothis和父元素的doThis都会执行，这可能并不是我们希望的结果，加上stop修饰符，阻止冒泡事件，父元素doThis不会执行 -->
<div @click = "doThis">
    点击父元素
    <a @click.stop = "dothis">点击子元素</a>
</div>

<!-- capture可以拦截冒泡事件  -->
<!-- capture修饰符会在冒泡事件的捕获阶段就执行，所以alert的顺序是1 2 4 3 -->
<div @click.capture = "alert(1)">
    <div @click.caputure = "alert(2)">
        <div @click = "alert(3)">
            <div @click = "alert(4)">
         
            </div>
        </div>
    </div>
</div>

<!-- self表示只有直接作用在他标记的元素上才可以执行，否则冒泡跳过该元素 -->
<!-- once表示事件只执行一次 -->
<!-- prevent表示阻止默认事件，比如a的连接跳转，submit的事件提交 -->
<!-- passive用于scroll事件，并且让preventDefault失效 -->
```

按钮修饰符：.enter、.tab、.delete（back键和del键)、.esc、.up、.down、.left、.right

```html
<!-- 按enter键up的 -->
<input @keyup.enter = "submit">
```

鼠标修饰符：.left、.right、.middle

### <h1 id = "表单输入绑定（还不熟悉）">表单输入绑定（还不熟悉）</h1>



### <h1 id = "组件">组件</h1>

#### 基础

1	通过Prop向子组件传值，$emit向父组件传值。通过new Vue({ })实现组件的传值

2	slot插槽分发内容

```html
// 父组件
<componentA>
	<p name = "slotA">
        this is slotA
    </p>
    <p name = "slotB">
        this is slotB
    </p>
</componentA>

// 子组件componentA
<div>
    <slot name = "slotA"></slot>
    <slot name = "slotB"></slot>
</div>

// 渲染之后
<div>
    <p name = "slotA">
        this is slotA
    </p>
    <p name = "slotB">
        this is slotB
    </p>
</div>
```

#### `动态组件`

当我们希望可以切换一个多标签的界面时可以使用is特性来切换不同的组件，具体见官方文档

```html
<component v-bind:is = "currentComponent"></component>

<script>
    import componentA from './components/componentA'
    import componentB from './components/componentB'
    new Vue({
        data(){
            return {
                flag = 'true'
            }
        }
        computed:{
            currentComponent(){
                if(flag){
                    return 'componentA';
                }else{
                    return 'componentB'
                }
            }
        }
        components:{
            componentA,
            componentB
        }
    })
    
</script>
```

但是这样每次切换都会回到初始状态，不会保持切换之前的状态，如果要保存切换之前的状态，可以用keep-alive

```html
<keep-alive>
	<component :is = "currentComponent"></component>
</keep-alive>
```

#### `异步组件`

在需要使用的时候才回去加载，减少了第一次渲染时的加载时间，算是一个性能优化。

```javascript
// 一下返回的都是Promise对象

// webpack + code-spliting
// 全局组件的异步加载 
Vue.component('asyncComponent',function(resolve,reject){
    // 注意，第一个参数是数组，可以加载多个组件
    require(['./components/asyncComponent.vue'],resolve); 
})

// 局部组件的异步加载
new Vue({
    components:{
        'asyncComponent':function(resolve,reject){
			require(['./components/asyncComponent.vue'],resolve);
        }
    }
})

// webpack + es6
// 全局组件的异步加载
Vue.component('asyncComponent', () => import('./components/asyncComponent.vue'));

// 局部组件的异步加载
components:{
    'asyncComponent': () => import('./components/asyncComponent.vue')
}
```

加载状态的处理

```javascript
// 如果要vue-router使用这个语法的话，要2.4.0+的版本

const AsyncComponent = () => ({
  // 需要加载的组件 (应该是一个 `Promise` 对象)
  component: import('./MyComponent.vue'),
  // 异步组件加载时使用的组件
  loading: LoadingComponent,
  // 加载失败时使用的组件
  error: ErrorComponent,
  // 展示加载时组件的延时时间。默认值是 200 (毫秒)
  delay: 200,
  // 如果提供了超时时间且组件加载也超时了，
  // 则使用加载失败时使用的组件。默认值是：`Infinity`
  timeout: 3000
})
```





### <h1 id = "Prop">Prop</h1>

向子组件传值，提供类型检测。

父组件的绝大多数特性会覆盖子组件的特性，但是class和style会合并，而且是与根元素合并。

### 自定义事件

1	推荐使用kebab-case命名方式

2	自定义组件上的v-model会默认使用组件上名为value的值和input的事件，如果value有其他用途，那么这个时候就需要使用model来避免。具体看文档，并不常用。

3	将原生是将绑定到组件上要使用   `.native`修饰符。

```html
<!-- 但是如果遇到组件的根元素标签是label、input之类也会失效。这个时候需要用到$listeners -->
<componentA @click.native = "greet"></componentA>
```

4	`.sync`修饰符，可以实现父子组件双向数据绑定

```html
<componentA :title.sync = "doc.title"></componentA>
<!-- 此时当子组件里的doc.title改变时，父组件里的doc.title也会改变 -->
<!-- 如果需要用一个对象设置多个prop时，可以如下使用 -->
<componentA v-bind.sync = "doc"></componentA>
```

### <h1 id = "混入">混入</h1>

mixins一般有两种用途：

1	在已经写好了构造器后，需要增加方法或者临时的活动时使用，以减少源代码的污染。

2	很多地方都会用到的公用方法，用混入的方法减少代码量，实现代码复用

```javascript
var mixin = {
    data:function(){
        return {
            message:'hello',
            foo:'abc'
        }
    }，
    created:function(){
    	console.log("混入钩子")
	}
}

new Vue({
    mixins:[mixin], // 混入
    data:function(){
        return {
            message:'goodbye',
            bar:'def'
        }
    },
    created:function(){
        console.log(this.$data); // 混入钩子 {message:'goodbye', foo:'bar', bar:'def'}
    }
})

// 其实混入之后的效果就是下面这样的
new Vue({
    data() {
        return {
            message: 'goodbye',
            bar: 'def',
            foo: 'bar'
        }
    },
    created(){
        console.log("混入钩子");
        console.log(this.$data);
    }
})
```

数据对象在内部会进行浅合并 (一层属性深度)，在和组件的数据发生冲突时以组件数据优先（如message）。 如果混入了钩子，混入的钩子函数先执行，混入的对象的钩子函数后执行（如created）。

### <h1 id = "动画">动画</h1>

#### transition属性过渡

```html
<div>
    <transition name="fade" mode="in-out"> //transition有切换的时候的切换模式
    	<h1 v-if="show">
            hello world
        </h1>
    </transition>
    <button @click="hadleClick">
        switch
    </button>
</div>
<script>
    new Vue({
        data:function(){
            return {
                show:true;
            }
        },
        methods:{
            hadleClick:function(){
                this.show = !this.show;
            }
        }
    })
</script>
<style>
    .fade-enter{ /*fade-enter和fade-leave都是给属性做初始化*/
		opacity:0;
    }
    .fade-enter-active{
        trsition:opacity 3s;
    }
    .fade-leave-to{/*fade-enter-to和fade-leave-to都是表明哪些属性变化*/
        opacity:0;
    }
    .fade-leave-active{/*fade-enter-active和fade-leave-active就是监控哪些属性变化，然后做动画*/
        trsition:opacity 0 3s;
    }
</style>
```

#### 使用Animate.css文件的动画效果

```html
<link rel="stylesheet" type="text/css" href='./animate.css'>
<div>
    <!-- 自定义出现动画和消失动画的名字 使用animate.css库需要在加上一个animated类名和动画效果名-->
    <transition name="fade"
                type="transition"<!-- 以transition的时长为准 -->
    		   :duration="1000" 自定义动画时长
    		   :duration="{enter:5000,leave:1000}" 分别设置出场动画时间和消失动画时间
                appear <!声明初次动画>
                enter-active-class="animated swing fade-enter-active" 混写animate和transition
                leave-active-class="animated shake fade-leave-active"
                appear-active-class="animated swing">初次动画
    	<h1 v-if="show">
            hello world
        </h1>
    </transition>
    <button @click="hadleClick">
        switch
    </button>
</div>
<script>
    new Vue({
        data:function(){
            return {
                show:true;
            }
        },
        methods:{
            hadleClick:function(){
                this.show = !this.show;
            }
        }
    })
</script>
```

#### vue中的js动画

vulocity：JS动画库

动画函数钩子：before-enter(el)、enter(el,done)、after-enter(el)；done被调用之后才会触发afterEnter

   			   before-leave(el)、leave(el,done)、after-leave(el)

```html
<link rel="stylesheet" type="text/css" href='./animate.css'>
<div>
    <!-- 自定义出现动画和消失动画的名字 使用animate.css库需要在加上一个animated类名和动画效果名-->
    <transition name="fade"
                @before-enter="handleBeforeEnter"
                @enter="handleEnter"
                @after-enter="handleAfterEnter">
    	<h1 v-if="show">
            hello world
        </h1>
    </transition>
    <button @click="hadleClick">
        switch
    </button>
</div>
<script>
    new Vue({
        data:function(){
            return {
                show:true;
            }
        },
        methods:{
            hadleClick:function(){
                this.show = !this.show;
            },
            handleBeforeEnter(el){
                el.style.opacity = 0;
            },
            handleEnter(el,done){
                el.style.opacity = 1;
                done(); // 不调用done，after-enter无法触发
            },
            handleAfterEnter(el){
                alert("done");
            }
        }
    })
</script>
```

#### transition-group列表过渡

相当于列表的每一个元素都添加一个<transition></transition>,简写成<transition-group></transition-group>

```html
<div>
    <transition-group name="fade">
    	<div v-for="item in titles" :key="item.id">
        	{{item.title}}
        </div>
    </transition>
    /*
    transition-group相当于以下的transition
    <transition name="fade">
    	<div>
            {{item.title}}
        </div>
    </transition>
    <transition name="fade">
    	<div>
            {{item.title}}
        </div>
    </transition>
    <transition name="fade">
    	<div>
            {{item.title}}
        </div>
    </transition>
    */
    <button @click="hadleClick">
        ADD
    </button>
</div>
<script>
    let count = 0;
    new Vue({
        data:function(){
            return {
                titles:[];
            }
        },
        methods:{
            hadleClick:function(){
                this.titles.push({id:count++,title:`number${count}`});
            }
        }
    })
</script>
<style>
    .fade-enter, .fade-leave-to{ 
		opacity:0;
    }
    .fade-enter-active, .fade-leave-active{
        trsition:opacity 3s;
    }
</style>
```

#### Vue动画封装成组件

```html
<div>
    <fade :show="show">
    	<h1>
            hello world
        </h1>
    </fade>
    <button @click="handleClick">
        switch
    </button>
</div>
<script>
    let count = 0;
    Vue.component('fade',{
        props:['show'],
        template:`
			<transition @before-enter="handleBeforeEnter" @enter="handleEnter" @after-enter="handleAfterEnter">
    			<slot v-if="show"></slot>
    		</transition>
		`,
        methods:{
		   handleBeforeEnter(el){
			el.style.opacity = 0;
           },
            handleEnter(el,done){
                el.style.opacity = 1;
                done();
            }
            handleAfterEnter(el){
                alert("done");
            }
        }
    })
    new Vue({
        data:function(){
            return {
                show:true;
            }
        },
        methods:{
            hadleClick:function(){
                this.show = !this.show;
            }
        }
    })
</script>
<style>
    .fade-enter, .fade-leave-to{ 
		opacity:0;
    }
    .fade-enter-active, .fade-leave-active{
        transition:opacity 3s;
    }
</style>
```



### <h1 id = "数据双向绑定（proxy）">数据双向绑定（proxy）</h1>

```javascript
const input = document.getElementById('input');
const p = document.getElementById('p');
const obj = {};
const newobj = new Proxy(obj,{
    // 代理target及目标对象，key是目标对象的属性，receiver是Proxy本身
    get:function(target,key,receiver){
        console.log(`getting ${key}`);
    },
    // value是key的值
    set:function(target,key,value,receiver){
        console.log(target,key,value,receiver);
        if(key === 'text'){
            input.value = value;
            p.innerHTML = value;
        }
        return Reflect.set(target,key,value,receive);
    }
});

input.addEventListener = function('keyup',function(e){
    newobj.text = e.target.value;
})
```

`Proxy的优点`：Proxy和Object.defineProperty都是数据劫持，但是Object.defineProperty是劫持的对象的属性，而Proxy可以劫持整个对象，同时Proxy还定义了许多其他的拦截方法，这个是Object.defineProperty不具备的。不过，Proxy现在还有兼容性的问题，所以听说Vue会在3.0是才是用Proxy代替Object.defineProperty。



