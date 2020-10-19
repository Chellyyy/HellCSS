# Border

我平常border写得最多的样式就是
```
    {
        border:1px solid color    
    }
```
这就能看出来border其实是三个属性的简写，分别是 border-width、border-style、border-color。
这三个属性都与方向有关，所以可以分开按照方向写，也可以简写。

## 简写快速记忆
* 方向记忆：顺序为上右下左，也可以按照从上开始顺时针的方向进行记忆
```
        top
    left  right
        down
```
* 缩写
```
    {
        border-width:0; /*一个值，四个方位相同*/
        border-width:10px 20px; /*两个值，上下为10px，左右为20px*/
        border-width:5px 10px 15px; /*三个值，左右为10px，上为5px，下为15px*/ 
        border-width:5px 10px 15px 20px; /*四个值，按照方向记忆，上5px，右10px，下15px，左20px*/
    }
```
## 实践1实现一个三角形
实现三角形的原理很简单，我们先了解一下基础知识，CSS盒模型  
![image](https://github.com/Chellyyy/HelloCSS/blob/master/demo/border/box.png?raw=true)  

一个盒子包括: margin+border+padding+content
### 扩展
盒子宽度高度的计算由CSS的属性box-sizing控制  
`content-box`默认值，盒子的宽高=width(content)+padding+border设置border和padding不影响content的宽高  
`border-box`IE盒子模型，盒子的宽高=width(padding+border+content)，在设置了宽高后，如果修改了border和padding，content的宽高会受影响

言归正传，在我们设置border时如何实现一个三角形呢，我们先来看看border是怎么工作的  
在i标签上添加如下样式
```
    {
        display: inline-block;
        border-color: #ff0000 #00ff00 #0000ff #ffff00;
        border-style:solid;
        border-width:20px;
        height:50px;
        width:50px;
    }
```
会显示如下样式  
![image](https://github.com/Chellyyy/HelloCSS/blob/master/demo/border/border1.png?raw=true)  
在上面的基础上，我们设置宽高为0，就得到了如下样式  
![image](https://github.com/Chellyyy/HelloCSS/blob/master/demo/border/border2.png?raw=true)  
通过这两个样式我们可以得出border的工作原理如下
* 上下左右边框交界处出呈现平滑的斜线，在宽高为0的情况下内容为空，原本的梯形就变成了三角形
* 通过调整border的宽度大小可以调节三角形形状  

那要实现我们想要的三角形不是很容易了嘛，只要其他三个边的颜色是透明的，就能得到各个角度的三角形  
现在只保留top的颜色，将其他颜色设置为透明试试
```
    {
        width: 0;
        height: 0;
        border: 10px solid transparent;
        position: absolute
        border-top-color: red;
    }
```
就得到了这样的三角形  
![image](https://github.com/Chellyyy/HelloCSS/blob/master/demo/border/triangle.png?raw=true)  

日常使用中三角形基本上都需要居中显示，但是通过border实现的三角形其实本质上还是一个正方形边缘画了一个三角形，是无法居中的。  
![image](https://github.com/Chellyyy/HelloCSS/blob/master/demo/border/triangle2.png?raw=true)  
这个时候我们就可以在这个标签外部包一层容器，计算相对应的padding，使其居中，这样我们用起来也会比较方便。
```
    {
        width: 20px;
        height: 20px;
        display: inline-block;
        padding: 0 5px 10px;
    }
```
![image](https://github.com/Chellyyy/HelloCSS/blob/master/demo/border/triangle3.png?raw=true)  

[demo地址](https://github.com/Chellyyy/HelloCSS/blob/master/demo/border/index.html)