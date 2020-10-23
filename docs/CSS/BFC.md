# BFC  
块格式化上下文（Block Formatting Context，BFC） 是Web页面的可视CSS渲染的一部分，是块盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域。

```
BFC(Block formatting context)直译为"块级格式化上下文"。  
它是一个独立的渲染区域，只有Block-level box参与， 它规定了内部的Block-level Box如何布局，并且与这个区域外部毫不相干。
```

### 扩展 Box和Formatting Context
**Box：css布局的基本单位**  
**Formatting Context：是页面中的一块渲染区域，并且有一套渲染规则，它决定了其子元素将如何定位，以及和其他元素的关系和相互作用。  
最常见的 Formatting context 有 Block formatting context (简称BFC)和 Inline formatting context (简称IFC)**


### 如何创建BFC
* 根元素或其它包含它的元素
* 浮动 (元素的float不为none)
* 绝对定位元素 (元素的position为absolute或fixed)
* 行内块inline-blocks(元素的 display: inline-block)
* 表格单元格(元素的display: table-cell，HTML表格单元格默认属性)
* overflow的值不为visible的元素
* 弹性盒 flex boxes (元素的display: flex或inline-flex)  
[Introduction to formatting contexts 格式化上下文简介](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Flow_Layout/Intro_to_formatting_contexts)

## BFC的规则
* 内部的Box会在垂直方向，一个接一个地放置  
其实这个不难理解，就是一个BFC内部的元素，如果不指定特殊样式，会一个一个垂直排列。  
有点类似于flex布局中的column排列，来看两个例子  
```
<div class="vertical">
    <div class="vertical-item vertical-item1">item1</div>
    <div class="vertical-item vertical-item2">item2</div>
    <div class="vertical-item vertical-item3">item3</div>
    <div class="vertical-item vertical-item4">item4</div>
</div>
<br/>
<div class="flex">
    <div class="flex-item flex-item1">item1</div>
    <div class="flex-item flex-item2">item2</div>
    <div class="flex-item flex-item3">item3</div>
    <div class="flex-item flex-item4">item4</div>
</div>
<style>
    .vertical {
        height: auto;
        background: gray;
        overflow: auto;
    }

    .vertical-item {
        height: 30px;
        opacity: 0.5;
        text-align: center;
    }

    .vertical-item1 {
        width: 200px;
        background: deepskyblue;
        float: left;
    }

    .vertical-item2 {
        width: 100px;
        background: yellow;
        position: absolute;
    }

    .vertical-item3 {
        width: 300px;
        background: pink;
    }

    .vertical-item4 {
        width: 400px;
        background: brown;
    }

    .flex {
        display: flex;
        flex-direction: column;
        background: gray;
    }

    .flex-item {
        height: 30px;
        opacity: 0.5;
        text-align: center;
    }

    .flex-item1 {
        width: 200px;
        background: deepskyblue;
        float: left;
    }

    .flex-item2 {
        width: 100px;
        background: yellow;
        position: absolute;
    }

    .flex-item3 {
        width: 300px;
        background: pink;
    }

    .flex-item4 {
        width: 400px;
        background: brown;
    }
</style>
```
效果如下图所示  
![image](https://github.com/Chellyyy/HelloCSS/blob/master/demo/BFC/1.png?raw=true)  
蓝色框：item1，黄色框：item2，粉色框item3，红色框item4，为了便于看清重叠，他们都是半透明的。  
先看第一个容器，在同一个BFC中，item1和item2都脱离了文档流，所以他们和item3重叠到了一块，导致三个区域都在一行。item4独占一行。
flex布局中，由于容器设置了display:flex，会导致子元素的float属性失效。所以item1仍独占一行，由于item2脱离文档流，所以他们重叠了。

* Box垂直方向的距离由margin决定。属于同一个BFC的两个相邻Box的margin会发生重叠。
```
<div class="margin">
    <div class="margin-item"></div>
    <div class="margin-item"></div>
</div>
<style>
    .margin{
        overflow: auto;
    }
    .margin-item{
        margin:20px;
        background-color: red;
        width: 50px;
        height: 50px;
    }
</style>
```
效果如下图  
![image](https://github.com/Chellyyy/HelloCSS/blob/master/demo/BFC/2.png?raw=true)  
两个div都设置了margin，但是两个div垂直方向的margin重叠了  

* 每个盒子（块盒与行盒）的margin box的左边，与包含块border box的左边相接触(对于从左往右的格式化，否则相反)。即使存在浮动也是如此。  
要理解这一点，我们得先搞清除，包含块是什么。  
包含块受元素的position属性影响，也就是说，就算是同级元素，也可能处在不同的包含块中。
[布局和包含块](https://developer.mozilla.org/zh-CN/docs/Web/CSS/All_About_The_Containing_Block)  
```
<div class="block1">
    block1
    <div class="block2">
        block2
        <div class="block-item block-item1">item1</div>
        <div class="block-item block-item2">item2</div>
        <div class="block-item block-item3">item3</div>
    </div>
</div>
<style>
    .block1{
        overflow: hidden;
        background-color: gray;
        position: relative;
        height: 200px;
        padding-top: 50px;
        margin: 10px;
        border: 5px solid black;
    }
    .block2{
        overflow: hidden;
        background-color: aliceblue;
        height: 150px;
        margin: 10px;
    }
    .block-item{
        width: 50px;
        height: 50px;
    }
    .block-item1{
        top: 0;
        position: absolute;
        background-color: red;
    }
    .block-item2{
        background-color: deepskyblue;
        float: left;
    }
    .block-item3{
        background-color: yellow;
    }
</style>
```
效果如下图  
![image](https://github.com/Chellyyy/HelloCSS/blob/master/demo/BFC/3.png?raw=true)  
在这个例子中，block1和block2都是独立的BFC，但是由于item1的position是absolute，所以他与第一个position属性不为static的包含块的左边相接触。  
猜测这也是absolute为什么相对于第一个position不为static的父类定位的原因。 
 
* 计算BFC的高度时，浮动元素也参与计算。
```
<div class="float">
    <div class="float-item">float</div>
</div>
<style>
    .float{
        overflow: hidden;
        background-color: gray;
        border: 1px solid black;
    }
    .float-item{
        float: left;
        width: 50px;
        height: 50px;
        background-color: red;
    }
</style>
```
当容器未设置overflow时，效果如下图  
![image](https://github.com/Chellyyy/HelloCSS/blob/master/demo/BFC/4.png?raw=true)  
设置了overflow时，效果如下图，所以这个用法也经常被用来 清除浮动  
![image](https://github.com/Chellyyy/HelloCSS/blob/master/demo/BFC/5.png?raw=true)  
* BFC的区域不会与float box重叠。
```
<div class="container">
    <div class="left">left</div>
    <div class="right">right</div>
</div>
<style>
    .container{
        background-color: gray;
    }
    .left{
        float: left;
        background-color: red;
        width: 50px;
    }
    .right{
        overflow: hidden;
        height: 200px;
        background-color: deepskyblue;
    }
</style>
```
不设置overflow时，效果如下图  
![image](https://github.com/Chellyyy/HelloCSS/blob/master/demo/BFC/6.png?raw=true)  
设置了overflow时，效果如下图，可以实现一个 自适应两栏布局  
![image](https://github.com/Chellyyy/HelloCSS/blob/master/demo/BFC/7.png?raw=true)  

* BFC就是页面上的一个隔离的独立容器，容器里面的子元素不会影响到外面的元素。反之也如此。  
上面的例子都说明了这一点，所以BFC在日常布局中还是十分有用的。

### 总结
BFC可以用来  
* 避免margin重叠
* 清除浮动
* 阻止元素被浮动元素覆盖

### 扩展 清除浮动常用的办法
* 在最后一个浮动元素后面添加一个标签（或后一个邻接元素），添加属性clear:both 这个方法会多一个无意义的标签
* 容器添加浮动 这个方法不利于布局
* 父元素添加overflow:hidden触发BFC 容易有溢出隐藏问题
* 编写clearfix样式，用after伪类来清除浮动
```
.clearfix:after{/*伪元素是行内元素 正常浏览器清除浮动方法*/
    content: "";
    display: block;
    height: 0;
    clear:both;
    visibility: hidden;
}
.clearfix{
    *zoom: 1;/*ie6清除浮动的方式 *号只有IE6-IE7执行，其他浏览器不执行*/
}
```
### 参考资料
[什么是BFC？看这一篇就够了](https://blog.csdn.net/sinat_36422236/article/details/88763187)  
[关于BFC的特征，为什么说内部的box会在垂直方向一个接一个放置？](https://segmentfault.com/q/1010000008875016)  
[Introduction to formatting contexts 格式化上下文简介](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Flow_Layout/Intro_to_formatting_contexts)  
[布局和包含块](https://developer.mozilla.org/zh-CN/docs/Web/CSS/All_About_The_Containing_Block)  

### Demo
[demo地址](https://github.com/Chellyyy/HelloCSS/blob/master/demo/BFC/index.html)