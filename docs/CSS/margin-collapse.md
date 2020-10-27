# margin 折叠问题 Margin Collapse
>块的上外边距(margin-top)和下外边距(margin-bottom)有时合并(折叠)为单个边距，其大小为单个边距的最大值(或如果它们相等，则仅为其中一个)，这种行为称为边距折叠。

[MDN-外边距折叠](https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing)  
## 折叠的结果：  
1、两个相邻的外边距都是正数时，折叠结果是它们两者之间较大的值。  
2、两个相邻的外边距都是负数时，折叠结果是两者绝对值的较大值。  
3、两个外边距一正一负时，折叠结果是两者的相加的和。  
margin 折叠本质上的问题还是 BFC，要理解这个问题，我们先来看看折叠的条件。
## 折叠的条件
* 必须是处于常规文档流（非 float 和绝对定位）的块级盒子，并且处于同一个 BFC 当中。
* 没有线盒，没有空隙（clearance，下面会讲到），没有 padding 和 border 将他们分隔开
* 都属于垂直方向上相邻的外边距，可以是下面任意一种情况
    * 元素的 margin-bottom 与其下一个常规文档流的兄弟元素的 margin-top
    * 元素的 margin-top 与其第一个常规文档流的子元素的 margin-top
    * height 为 auto 的元素的 margin-bottom 与其最后一个常规文档流的子元素的 margin-bottom
    * 高度为 0 并且最小高度也为 0，不包含常规文档流的子元素，并且自身没有建立新的 BFC 的元素的 margin-top 和 margin-bottom  

根据以上描述的条件，我们可以得出以下结论
* 浮动元素不与任何元素的外边距产生叠（包括其父元素和子元素）
* 绝对定位元素不与任何元素的外边距产生折叠
* 不同 BFC 的元素不会产生折叠
* inline-block 元素不与任何元素的外边距产生折叠
* 存在间隙（clearance）的元素不会产生折叠 
### 相邻兄弟元素折叠
```

```