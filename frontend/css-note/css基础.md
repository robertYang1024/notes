# position
- relative: 相对定位
   - 相对自己之前的位置移动
- absolute：绝对定位
   - 相对具有定位属性的父容器定位，如果没有则相对于body元素定位
   - 脱离文档流
- fixed：固定定位
   - 相对浏览器窗口定位，位置固定，不随滚动条变化，除非移动窗口
   - 脱离文档流

# 块状元素、内联元素和内联块状元素
- 块状元素（display：block）
   1. 每个块级元素都从新的一行开始，并且其后的元素也另起一行。
   2. 元素的高度、宽度、行高以及顶和底边距都可设置。
   3. 元素宽度在不设置的情况下，是它本身父容器的100%（和父元素的宽度一致）。默认高度等于子元素高度。父子均是块级元素时，子块的高度可能冲破父级的限制。
   4. 常用的块状元素有：`<div>、<p>、<h1>-<h6>、<ol>、<ul>、<dl>、<table>、<address>、<blockquote> 、<form>`
- 内联元素(又叫行内元素display：inline)
   1. 和其他内联元素都在一行上
   2. 可以通过margin、padding来改变左右的距离，但不可以改变上下的距离，导致width、height、line-height失效或。可以使用border。
   3. 常用的内联元素有：`<a>、<span>、<br>、<i>、<em>、<strong>、<label>、<q>、<var>、<cite>、<code>`

- 内联块状元素（display:inline-block）
   1. 和其他元素都在一行上；
   2. 元素的高度、宽度、行高以及顶和底边距都可设置。
   3. 它也会有元素间出现空白区域的问题
   4. 常用的内联块状元素有：`<img>、<input>`

 # flex
 ## 语法
 `display:flex` 或者 `display:inline-flex`

作用在flex容器上 | 作用在flex子项上
----------------|-----------------
flex-direction  |  order
flex-wrap       |  flex-grow
flex-flow       |  flex-shrink
justify-content |  flex-basis
align-items     |  flex
align-content   |  align-self

 ## flex缩写语法
单值语法      |     等同于      |备注        |特性
-------------|----------------|------------|-----
flex: initial|	flex: 0 1 auto	|初始值，常用 |                                                          
flex: auto   |	flex: 1 1 auto |	适用场景少 | 在尺寸不足时会优先最大化内容尺寸
flex: none   |	flex: 0 0 auto |	推荐      | 不会换行，无视容器的尺寸限制，直接溢出容器，表现为最大内容宽度
flex: 1      |	flex: 1 1 0%   |	推荐      | 在尺寸不足时会优先最小化内容尺寸
flex: 0      |	flex: 0 1 0%   |	适用场景少 | 表现为最小内容宽度，会换行，“一柱擎天”

### 适用场景：
- **flex:initial**：除了上图所示的布局效果外，flex:initial声明还适用于一侧内容宽度固定，另外一侧内容宽度任意的两栏自适应布局场景
- **flex:1**：当希望元素充分利用剩余空间，同时不会侵占其他元素应有的宽度的时候，适合使用flex:1，这样的场景在Flex布局中非常的多。例如所有的等分列表，或者等比例列表都适合使用flex:1或者其他flex数值。
- **flex:auto**：当希望元素充分利用剩余空间，但是各自的尺寸按照各自内容进行分配的时候，适合使用flex:auto。flex:auto多用于内容固定，或者内容可控的布局场景，例如导航数量不固定，每个导航文字数量也不固定的导航效果就适合使用flex:auto效果来实现
- **flex:0**：由于应用了flex:0的元素表现为最小内容宽度，因此，适合使用flex:0的场景并不多，除非元素内容的主体是替换元素，此时文字内容就会庇护在替换元素的宽度下从而不会出现“一柱擎天”的排版效果。
- **flex:none**：当flex子项的宽度就是内容的宽度，且内容永远不会换行，则适合使用flex:none。例如按钮里的文字不能换行

### 总结：
- flex:initial表示默认的flex状态，无需专门设置，适合小控件元素的分布布局，或者某一项内容动态变化的布局；
- flex:0适用场景较少，适合设置在替换元素的父元素上；
- flex:none适用于不换行的内容固定或者较少的小控件元素上，如按钮。
- flex:1适合等分布局；
- flex:auto适合基于内容动态适配的布局；

<br>

### 参考资料：
- 写给自己看的display: flex布局教程：https://www.zhangxinxu.com/wordpress/2018/10/display-flex-css3-css/ 
- flex:0 flex:1 flex:none flex:auto应该在什么场景下使用？：https://www.zhangxinxu.com/wordpress/2020/10/css-flex-0-1-none/ 
- CSS flex属性深入理解：https://www.zhangxinxu.com/wordpress/2019/12/css-flex-deep/ 
- CSS flex-basis原来有这么多细节：https://www.zhangxinxu.com/wordpress/2019/12/css-flex-basis/ 

# 重绘回流

### 回流 (Reflow)
当Render Tree中部分或全部元素的尺寸、结构、或某些属性发生改变时，浏览器重新渲染部分或全部文档的过程称为回流。
### 重绘 (Repaint)
当页面中元素样式的改变并不影响它在文档流中的位置时（例如：color、background-color、visibility等），浏览器会将新样式赋予给元素并重新绘制它，这个过程称为重绘。

### 回流必将引起重绘，重绘不一定会引起回流。

<br>

### 如何避免

CSS
- 避免使用table布局。
- 尽可能在DOM树的最末端改变class。
- 避免设置多层内联样式。
- 将动画效果应用到position属性为absolute或fixed的元素上。
- 避免使用CSS表达式（例如：calc()）。

JavaScript
- 避免频繁操作样式，最好一次性重写style属性，或者将样式列表定义为class并一次性更改class属性。
- 避免频繁操作DOM，创建一个documentFragment，在它上面应用所有DOM操作，最后再把它添加到文档中。
- 也可以先为元素设置display: none，操作结束后再把它显示出来。因为在display属性为none的元素上进行的DOM操作不会引发回流和重绘。
- 避免频繁读取会引发回流/重绘的属性，如果确实需要多次使用，就用一个变量缓存起来。
- 对具有复杂动画的元素使用绝对定位，使它脱离文档流，否则会引起父元素及后续元素频繁回流。

<br>

参考资料：https://juejin.cn/post/6844903569087266823