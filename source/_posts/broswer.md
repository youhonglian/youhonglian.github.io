---
title: 常见的浏览器兼容问题
categories:  面试
tags: 
- interview
---

## 浏览器兼容问题



###  不同浏览器的标签默认的外补丁和内补丁不同

 这个是最常见的也是最易解决的一个浏览器兼容性问题，几乎所有的CSS文件开头都会用通配符*来设置各个标签的内外补丁是0。

 ```javascript
 *{margin:0;padding:0;}
 ```

 ###  图片处理问题

 png24位的图片在iE6浏览器上出现背景，解决方案是做成PNG8.也可以引用一段脚本处理.

 ### IE6双边距bug

 块属性标签float后，又有横行的margin情况下，在ie6显示margin比设置的大。浮动ie产生的双倍距离（IE6双边距问题：在IE6下，如果对元素设置了浮动，同时又设置了margin-left或margin-right，margin值会加倍。）
 解决方案是在float的标签样式控制中加入
```
_display:inline;
```
将其转化为行内属性。(_这个符号只有ie6会识别)

### 上下margin重合问题

ie和ff都存在，相邻的两个div的margin-left和margin-right不会重合，但是margin-top和margin-bottom却会发生重合。
解决方法，养成良好的代码编写习惯，同时采用margin-top或者同时采用margin-bottom。

### 图片默认有间距

使用float属性为img布局,因为img标签是行内属性标签，所以只要不超出容器宽度，img标签都会排在一行里，但是部分浏览器的img标签之间会有个间距。去掉这个间距使用float是正道。（使用负margin，虽然能解决，但负margin本身就是容易引起浏览器兼容问题的用法，所以尽量不要使用。）

### 设置较小高度标签（一般小于10px），在IE6，IE7中高度超出自己设置高度

给超出高度的标签设置overflow:hidden;或者设置行高line-height 小于你设置的高度。这种情况一般出现在我们设置小圆角背景的标签里。出现这个问题的原因是IE8之前的浏览器都会给标签一个最小默认的行高的高度。即使你的标签是空的，这个标签的高度还是会达到默认的行高。

### 行内属性标签，设置display:block后采用float布局，又有横行的margin的情况，IE6间距bug

在display:block;后面加入display:inline;display:table.行内属性标签，为了设置宽高，我们需要设置display:block;(除了input标签比较特殊)。在用float布局并有横向的margin后，在IE6下，他就具有了块属性float后的横向margin的bug。不过因为它本身就是行内属性标签，所以我们再加上display:inline的话，它的高宽就不可设了。这时候我们还需要在display:inline后面加入display:talbe。