# 居中布局

### position + margin

优点：不需要知道content的宽高

缺点：脱离文档，一旦contain坍塌就会得不到想要的结果，所以contain需要完全控制。

```html
<div class="contain">
  <div class="content"></div>
</div>
<style>
	.contain{
      position:relative;
      width:300px;
      height:300px;
    }
    .content{
      width:100px;
      height:100px;
      position:absolute;
      left:0;
      right:0;
      top:0;
      bottom:0;
      margin:auto;
    }
</style>
```

### line-hieght

优点：兼容性好，

缺点：需要对contain完全控制，没有太多的弹性。

```html
<div class="contain">
  <div class="content"></div>
</div>
<style>
    /*关键点就是contain的
    line-height==height，
    text-align:center。
    content的display：inline-block，
    vertical-align:middle*/
	.contain{
      width:300px;
      height:300px;
      background-color:rgba(7,17,27,.3);
      line-height:300px; 
      text-align:center; /*对块级子元素不起作用*/
    }
    .content{
      width:100px;
      height:100px;
      background-color:#f99;

      display:inline-block;
      vertical-align:middle; /*vertical-align只对本身是inline、inline-block水平的元素有用*/
    }
</style>
```

### table

优点：自带对齐功能

缺点：table属性，需要外加一层wrapper

```html
<div class="contain">
  <div class="content"></div>
</div>
<style>
    /*关键点就是contain的
    display：table-cell
    vertival-align：middle
    content的
    margin：0 auto*/
	.contain{
      width:300px;
      height:300px;
      background-color:rgba(7,17,27,.3);
      display:table-cell;
      vertical-align:middle; 
    }
    .content{
      width:100px;
      height:100px;
      background-color:#f99;
      margin:0 auto;
    }
</style>
```

###flex

```html
<div class="contain">
  <div class="content"></div>
</div>
<style>
    /*关键点就是contain：display：flex，align-items:center
    content的margin：0 auto*/
    .contain{
      width:300px;
      height:300px;
      background-color:rgba(7,17,27,.3);
      display:flex;
      align-items:center;
    }
    .content{
      width:100px;
      height:100px;
      background-color:#f99;
      margin:0 auto;
    }
</style>
```

# 响应式布局

###viewport

```html
<meta name = "viewport" content="width = device-width,initial-scale=1.0">
```

###响应式图片

1 大图随容器自动缩放，保持宽高比

```html
<style>
    img{
        height:auto;
        width:auto;
        max-height:100%;
        max-width:100%;
    }
</style>
```

####背景图片

```html
background-size:cover
background-size:contain
```

### 保持宽高比

margin\padding的值是百分比的时候是相对父元素的width

```html
<div></div>
<style>
    div{
        height:0;
        padding-top:50%;
        background:#f99;
    }
</style>
```

### 导航栏布局

```html

```

### 网格布局

```html
<ul class="contain">
  <li class="list"></li>
  <li class="list"></li>
  <li class="list"></li>
  <li class="list"></li>
  <li class="list"></li>
  <li class="list"></li>
  <li class="list"></li>
  <li class="list"></li>
  <li class="list"></li>
</ul>
```

inline-block+justify：每行放下的数目固定

```html
<style>
.contain{
  margin:0;
  padding:0;
  text-align:center;
  }
.list{
  display:inline-block;
  width:30%; /*每行放下的数目固定*/
  height:0;
  padding-top:20%;
  margin-bottom:10px;
  background:#f99;
}
</style>
```

flex：宽高度固定，能放几个就放几个

```html
.contain{
  margin:0;
  padding:0;
  display:flex;
  flex-wrap:wrap;
  }
.list{
  display:inline-block;
  width:70px;
  height:50px;
  margin: 0 10px 10px 0;
  background:#f99;
}
```

### media query

针对不同的屏幕，应用不同的样式。

可以查询的media

- width height
- device-width device-height
- device-pixel-ratio 像素比
- orientation 

```html
<style>
.contain{
  margin:0;
  padding:0;
  display:flex;
  flex-wrap:wrap;
  }
.list{
  display:inline-block;
  width:70px;
  height:50px;
  /* padding-top:20%; */
  margin: 0 10px 10px 0;
  background:#f99;
}
/*当宽度<= 1000px的时候采用这个布局*/
@media screen and (max-width:1000px){
 .contain{
  margin:0;
  padding:0;
  text-align:justify;
  }
.list{
  display:inline-block;
  width:30%; 
  height:0;
  padding-top:20%;
  margin-bottom:10px;
  background:#f99;
}
</style>
```
#三拦布局

###圣杯布局

### 双飞翼布局



# BFC

### BFC定义

​	BFC(Block formating context)直译为“块级格式化上下文”，他是一个独立的渲染区域，无论内部元素如何变化都不会影响到BFC区域外的元素。

### BFC的生成

1. ​	根元素
2. ​        float的值不为none
3. ​        position的值为absolute或者fixed
4. ​        display的值为inline-block，table-cell，table -caption，table，flex，inline-flex
5. ​         overflow的值不为visible

### BFC的约束规则

1. 内部的Box会在垂直方向上一个接一个设置
2. 同一个BFC内部的元素的margin会重叠，不同BFC之间的margin不会重叠（垂直和水平方向都是）
3. 每个元素的做外边距与包含块的左边界向接触（包括浮动元素）
4. BFC区域不会和float区域的元素重叠
5. 计算BFC高度时，float也会包括在内
6. BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面元素，反之亦然

### BFC的应用

1. 防止margin塌陷

   ```html
   <style>
       .wrap { 
           overflow: hidden;/* 新的BFC，如果没有wrap，两者之间的为100px，有就是200px */
       }
       p {
           color: #f55;
           background: #fcc;
           width: 200px;
           line-height: 100px;
           text-align:center;
           margin: 100px;
       }
   </style>
   <body>
       <p>Haha</p>
       <div class="wrap">
           <p>Hehe</p>
       </div>
   </body>
   ```

   

2. 清除内部浮动

   根据第五条约束规则，可以将包含浮动元素的元素声明为BFC，从而避免浮动带来的布局变化

   ```html
   <style>
       .par {
           overflow: hidden;
           border: 5px solid #fcc;
           width: 300px;
       }
    
       .child {
           border: 5px solid #f66;
           width:100px;
           height: 100px;
           float: left;
       }
   </style>
   <body>
       <div class="par">
           <div class="child"></div>
           <div class="child"></div>
       </div>
   </body>
   ```

   

3. 自适应多兰布局

   根据约束规则第四条，比如三拦布局。

   ```html
   <style> 
       html, body { height: 100%; width: 100%; margin: 0; padding: 0; }
       .left{
         background:pink;
         float: left;
         width:180px;
       }
       .center{
         background:lightyellow;
         overflow:hidden; 
       }
       .right{
         background: lightblue;
         width:180px;
         float:right;
       }
   </style> 
   
   <body class="claro"> 
     <div class="container">
       <div class="left"></div>
       <div class="right"></div>
       <div class="center"></div>
     </div>
   </body>
   ```

参考资料

1. https://juejin.im/entry/59c3713a518825396f4f6969
2. http://www.zhangxinxu.com/wordpress/2015/02/css-deep-understand-flow-bfc-column-two-auto-layout/

# %到底是相对于谁的

### position定位类中的%是相对于谁

1. position:   absolute的元素中的left、right是相对于父级position元素的width值

   ​		   top、bottom是相对于父级position元素的height值

   position：relative的元素的left、right是相对于自身的width值，

   ​		   top、bottom是相对于自身的height值

   position：fixed的元素是相对于视窗定位，所以left、right是相对于视窗的width

   ​		   top、bottom是相对于视窗的height值

2. 盒子模型：

   盒子模型的height、width是相对于父级元素的height、width，但是padding和margin在writing-mode为水平方向时是相对于父级元素的`width`值，为垂直方向时是相对于父级元素的`height`值。

3. background-size是相对于元素自身的宽高