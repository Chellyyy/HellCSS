# margin 重叠问题 Margin Collapse
>块的上外边距(margin-top)和下外边距(margin-bottom)有时合并(重叠)为单个边距，其大小为单个边距的最大值(或如果它们相等，则仅为其中一个)，这种行为称为边距重叠。

[MDN-外边距重叠](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing)  
## 重叠的结果：  
1、两个相邻的外边距都是正数时，重叠结果是它们两者之间较大的值。  
2、两个相邻的外边距都是负数时，重叠结果是两者绝对值的较大值。  
3、两个外边距一正一负时，重叠结果是两者的相加的和。  
margin 重叠本质上的问题还是 BFC，要理解这个问题，我们先来看看重叠的条件。
## 重叠的条件
* 必须是处于常规文档流（非 float 和绝对定位）的块级盒子，并且处于同一个 BFC 当中。
* 没有线盒，没有空隙（clearance，下面会讲到），没有 padding 和 border 将他们分隔开
* 都属于垂直方向上相邻的外边距，可以是下面任意一种情况
    * 元素的 margin-top 与其第一个常规文档流的子元素的 margin-top
    * 元素的 margin-bottom 与其下一个常规文档流的兄弟元素的 margin-top
    * height 为 auto 的元素的 margin-bottom 与其最后一个常规文档流的子元素的 margin-bottom
    * 高度为 0 或 auto 并且最小高度也为 0，不包含常规文档流的子元素，并且自身没有建立新的 BFC 的元素的 margin-top 和 margin-bottom  

根据以上描述的条件，我们可以得出以下结论
* 浮动元素不与任何元素的外边距产生重叠（包括其父元素和子元素）
* 绝对定位元素不与任何元素的外边距产生重叠
* 建立 BFC 的元素（例如浮动或 overflow 属性的值不为 visible）不会与其在流中的子元素的外边距重叠
* inline-block 元素不与任何元素的外边距产生重叠
* 一个常规文档流元素的 margin-bottom 与它下一个常规文档流的兄弟元素的 margin-top 会产生重叠，除非它们之间存在间隙（clearance）
* 一个不包含 padding 和 border 的父元素的 margin-top 与其第一个不包含 clearance 子元素的 margin-top 产生重叠
* 一个 'height' 为 'auto' 并且 'min-height' 为 '0'的常规文档流元素的 margin-bottom 会与其最后一个常规文档流子元素的 margin-bottom 重叠，条件为父元素不包含 padding 和 border ，子元素的 margin-bottom 不与包含 clearance 的 margin-top 重叠
* 一个不包含border-top、border-bottom、padding-top、padding-bottom的常规文档流元素，并且其 'height' 为 0 或 'auto'， 'min-height' 为 '0'，其里面也不包含行盒(line box)，其自身的 margin-top 和 margin-bottom 会重叠。
  
[Collapsing margins](https://drafts.csswg.org/css2/#collapsing-margins)
总结一下就是
* 浮动元素、绝对定位、inline-block元素不与任何元素的外边距产生重叠
* 不同 BFC 的元素不会产生重叠
* 特殊情况下存在间隙（clearance）的元素不会产生重叠  

我们来看看产生重叠的几种情况
### 相邻兄弟元素重叠
```
<div class="BFC">
  <div class="middle-box margin red"></div>
  <div class="middle-box margin green"></div>
</div>
<style>
  .BFC{
    display: flow-root;
  }
  .margin{
    margin:20px;
  }
  .big-box{
    width: 100px;
    height: 100px;
  }
  .middle-box{
    width: 50px;
    height: 50px;
  }
  .red{
    background-color: red;
  }
  .green{
    background-color: green;
  }
  .blue{
    background-color: blue;
  }
</style>
```
效果如下图  
![image](https://github.com/Chellyyy/HelloFE/blob/master/demo/margin/1.png?raw=true)  
我们通过absolute、inline-block、float可以使其margin不重叠  
![image](https://github.com/Chellyyy/HelloFE/blob/master/demo/margin/2.png?raw=true)
![image](https://github.com/Chellyyy/HelloFE/blob/master/demo/margin/3.png?raw=true)
![image](https://github.com/Chellyyy/HelloFE/blob/master/demo/margin/4.png?raw=true)  
但是需要注意的是，使用absolute和float时，元素会脱离文档流，需要对其定位做相应处理。
另外，由于这两者脱离文档流之后，如果后面还有兄弟元素，这个兄弟元素仍然会与前一个元素邻接，也就是会继续产生 margin 重叠
```
<div class="BFC relative">
  <div class="middle-box margin red"></div>
  <div class="middle-box margin green" style="position:absolute;">absolute</div>
  <div class="middle-box margin blue"></div>
</div>
<div class="BFC">
  <div class="middle-box margin red"></div>
  <div class="middle-box margin green" style="float: left">float</div>
  <div class="middle-box margin blue"></div>
</div>
```
效果如下图  
![image](https://github.com/Chellyyy/HelloFE/blob/master/demo/margin/5.png?raw=true)
![image](https://github.com/Chellyyy/HelloFE/blob/master/demo/margin/6.png?raw=true)  
在这里就必须提一下经常与 float 一起出现的 clear 属性，这与我们上面提到的 [间隙（clearance）](https://drafts.csswg.org/css2/#clearance)也有关  
clear 属性规定元素的哪一侧不允许其他浮动元素  

|值|描述|
|:---|:---|
|left|元素框的上外边界低于这个元素之前的任何左浮动框的下外边界|
|right|元素框的上外边界低于这个元素之前的任何右浮动框的下外边界|
|both|元素框的上外边界低于这个元素之前的任何左浮动框或右浮动框的下外边界|
|none|对浮动框没有限制|
clear 属性设置了非 none 值后，就会产生间隙。这里摘抄文档中的原文，意思是间隙可以抑制重叠，并且充当元素的 margin-top
>Values other than none potentially introduce clearance. Clearance inhibits margin collapsing and acts as spacing above the margin-top of an element. It is used to push the element vertically past the float.
  
看一个例子 
```
<div class="BFC">
  <div class="middle-box margin green" style="float: left">float</div>
  <div class="middle-box margin blue" style="clear: both;"></div>
  <div class="middle-box margin red"></div>
</div>
```
效果如下，由于元素的 margin-top 会与 float 元素的 margin-bottom 相接，间隙就是元素 margin-top 到 float 元素 margin-top 的距离  
![image](https://github.com/Chellyyy/HelloFE/blob/master/demo/margin/7.png?raw=true)  
其实这里又隐式地产生了一个 margin 重叠，这不是套娃吗！  
**所以关于兄弟组件的 margin 重叠问题，还是建议用新建一个 BFC 的方式来解决。**  
上面 float 产生的重叠也可以用 BFC 来解决。
```
<div class="BFC">
  <div class="middle-box margin red"></div>
  <div  class="BFC">
    <div class="middle-box margin green"></div>
  </div>
</div>
```
需要注意的是，建议新建 BFC 的时候使用 display:flow-root，意为创建一个行为类似于根元素的元素，这样语义化更好  
### 父子元素 margin-top 重叠
```
<div class="BFC">
  <div class="big-box margin green">
    <div class="middle-box margin red"></div>
  </div>
</div>
```
效果如下，子元素和父元素的 margin-top 重叠了  
![image](https://github.com/Chellyyy/HelloFE/blob/master/demo/margin/8.png?raw=true)  
根据上面的总结，我们也可以在父子元素是上使用 inline-block、absolute、float 或在子元素上建立 BFC 来解决重叠问题  
但这里提供一种更好的方式，**设置父元素的 padding 或 border**  
```
<div class="BFC">
  <div class="big-box margin green" style="padding: 10px">
    <div class="middle-box margin red"></div>
  </div>
</div>
```
![image](https://github.com/Chellyyy/HelloFE/blob/master/demo/margin/9.png?raw=true)  
这样父元素子元素的 margin 也就不会重叠了  
另外，你可以在父元素中填充内容来制造间隙，可以使用::before 或 ::after 伪元素，不过这样会被内容占掉一部分的容器高度  
```
<div class="BFC">
  <div class="big-box margin green before">
    <div class="middle-box margin red"></div>
  </div>
</div>
<style>
  .before::before{
    content: '';
    display: inline-block;
  }
</style>
```
![image](https://github.com/Chellyyy/HelloFE/blob/master/demo/margin/10.png?raw=true)  
在实际使用中需要具体问题具体分析
### auto 高度的父元素与子元素的 margin 重叠
```
<div class="BFC">
  <div class="auto-box margin green">
    <div class="middle-box margin red"></div>
  </div>
</div>
<style>
  .auto-box{
    width: 100px;
    height: auto;
  }
</style>
```
效果如下  
![image](https://github.com/Chellyyy/HelloFE/blob/master/demo/margin/11.png?raw=true)  
解决的方式和上面的思路类似，这里需要注意的就是由于父元素的高度是 auto，子元素脱离文档流的情况需认真考虑
仍然推荐在父元素中添加 border、padding 属性或在子元素上建立 BFC 的方式来解决这个问题  
```
<div class="BFC">
  <div class="auto-box margin green" style="border: 10px solid;">
    <div class="middle-box margin red"></div>
  </div>
</div>
```
![image](https://github.com/Chellyyy/HelloFE/blob/master/demo/margin/12.png?raw=true)  
在查阅资料中发现可以通过设置 min-height 来解决 margin 重叠，其实本质上只是把父元素的内容撑开了。  
我个人认为这没有从根本上解决重叠的问题。
```
<div class="BFC">
  <div class="auto-box margin green" style="min-height: 100px">
    <div class="middle-box margin red"></div>
  </div>
</div>
```
![image](https://github.com/Chellyyy/HelloFE/blob/master/demo/margin/13.png?raw=true)  
