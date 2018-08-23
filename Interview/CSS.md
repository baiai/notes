本文将CSS主要分为了两个部分，一个是属性问题，另一个是布局问题。属性问题主要是讲解一些属性的细节和不为认知的特性与应用。布局问题主要是收集一些常用的布局问题和布局手段。

##属性问题

###inline元素是否可以设置padding和margin？

答案是不行，那为什么img、input、textarea、select这些标签可以呢？因为这些标签是替换元素。所谓的替换元素就是元素本身是没有内容的，他们的宽高表示的是要替换他们的元素的宽高，比如img标签如果设置了宽高，就表示放在img标签中的图片会按照设置的宽高进行缩放使得img标签的宽高能够完整的显示图片的内容。替换元素有很多在布局上的特性和块级元素相似，比如垂直边界压缩而水平边界不压缩，边框和补白默认为0。

###box-sizing的值的区别？

box-sizing有三个值：content-box和border-box以及inherit

context-box表示的是元素的width和height值表示的是content的width和height

border-box表示的是元素的width和height值表示的是border的width和height

###BFC原理

具有BFC特性的元素可以看做一个独立的容器，容器内的元素在布局上是不会影响容器外的元素的。同一级别的BFC之间不会发生margin重叠。

常见的触发BFC特性的条件有：

1. body根元素
2. float出none以外的值
3. position的值为absolute、fixed
4. display的值为inline-block、table-cells、flex
5. overflow出visible以外的值。

###伪类和伪元素

伪类用于当已有元素处于的某个状态时，为其添加对应的样式，这个状态是根据用户行为而动态变化的。比如说，当用户悬停在指定的元素时，我们可以通过:hover来描述这个元素的状态。虽然它和普通的css类相似，可以为已有的元素添加样式，但是它只有处于dom树无法描述的状态下才能为元素添加样式，所以将其称为伪类。

伪元素用于创建一些不在文档树中的元素，并为其添加样式。比如说，我们可以通过:before来在一个元素前增加一些文本，并为这些文本添加样式。虽然用户可以看到这些文本，但是这些文本实际上不在文档树中。

参考资料：http://www.alloyteam.com/2016/05/summary-of-pseudo-classes-and-pseudo-elements/



##布局问题

###居中总结

m-1：绝对定位 ，通过将内容设为position：absolute，上下左右设为0，margin设为auto，适用于块级元素。但是需要明确父级元素的宽高。

m-2：line-height ，适用于行内元素。同样也需要知道父元素的高。

m-3：flex 布局。

###%到底是相对于谁的

1. position定位类中的%是相对于谁

   position:   absolute的元素中的left、right是相对于父级position元素的width值top、bottom是相对于父级position元素的height值

   position：relative的元素的left、right是相对于自身的width值，top、bottom是相对于自身的height值

   position：fixed的元素是相对于视窗定位，所以left、right是相对于视窗的widthtop、bottom是相对于视窗的height值

2. 盒子模型：

   盒子模型的height、width是相对于父级元素的height、width，但是padding和margin在writing-mode为水平方向时是相对于父级元素的width值，为垂直方向时是相对于父级元素的height值。

3. background-size是相对于元素自身的宽高

###媒体查询

针对不同的屏幕，应用不同的样式。

可以查询的media

- width height
- device-width device-height
- device-pixel-ratio 像素比
- orientation 

###响应式图片

```css
/*大图随容器自动缩放并且保持宽高比*/
img{
    height:auto;
    width:auto;
    max-height:100%;
    max-width:100%;
}
```
###保持宽高比

margin\padding的值是百分比的时候是相对父元素的width

```css
div{
    height:0;
    padding-top:50%;
    background:#f99;
}
```



