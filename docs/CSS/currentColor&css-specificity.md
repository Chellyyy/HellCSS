# currentColor 与 CSS 优先级算法

今天看vue-element-admin的源码时，意外发现一个很有意思的变量 currentColor
```
.sub-el-icon[data-v-18eeea00] {
    color: currentColor;
    width: 1em;
    height: 1em;
}
```
调试的时候他也是直接以 currentColor 变量名的形式出现的。  
这有点像 CSS 中的变量，但是CSS的变量鼠标移上去是可以追到实际的值的，currentColor 就是一个单纯的变量名。  
![image](https://github.com/Chellyyy/HelloFE/blob/master/demo/currentColor/1.png?raw=true)   
那 currentColor 这个变量到底有什么作用呢？从字面上理解，他就是当前元素的color。我们来测试一下  
```
  .test-color{
    width: 50px;
    height: 50px;
    color: red;
    border: 1px solid currentColor;
  }
  <div class="test-color">
    test
  </div>
```
效果如下图  
![image](https://github.com/Chellyyy/HelloFE/blob/master/demo/currentColor/2.png?raw=true)  
看上去好像如我们所猜测的，currentColor 就是当前元素的颜色。  
这里提供各方搜索对 currentColor 的解释  
> currentColor 关键字代表原始的 color 属性的计算值。它允许让继承自属性或子元素的属性颜色属性以默认值不再继承。  
它也能用于那些继承了元素的 color 属性计算值的属性，相当于在这些元素上使用 inherit 关键字，如果这些元素有该关键字的话。  

>在CSS1和CSS2中定义了border-color属性的默认值是color属性的值，但却没有为此定义一个相应的关键字。  
这个问题在 SVG 中被意识到了，于是在 SVG 1.0 中引入了currentColor关键字。  
在CSS3中扩展了颜色值包含currentColor关键字，并用于所有接受颜色的属性上。  
currentColor 是 color 属性的值，具体意思是指：currentColor关键字的使用值是color属性值的计算值。  
如果 currentColor 关键字被应用在 color 属性自身，则相当于是color: inherit。  



