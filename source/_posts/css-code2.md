---
title: 《CSS揭秘》精读笔记（二）形状
date: 2021-06-23 09:22:01
tags: CSS
---
## 前言
本文为《CSS揭秘》的第二部分-形状的笔记，介绍了自适应椭圆、平行四边形、棱形图片、切角效果、梯形标签页、简单饼图的实现原理，并在每节中附上了个人的代码链接。文中对各类形状的实现给出了多种方案，不得不感慨新属性（如clip-path、conic-gradient）的出现给我们带来的便捷，其中conic-gradient在实现饼图上简直绝了，强烈推荐~

以下是一张总体概要的脑图，方便帮助大家进行知识梳理。其中标♥的部分为一些比较妙的方案，可以重点学习。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/478277ee156047f899a5730a278c14ca~tplv-k3u1fbpfcp-watermark.image)

## 1.自适应椭圆

> 背景知识<br>
> border-radius 属性的基本用法

### 原理
border-radius的三个小知识：
1. border-radius可以**单独指定水平和垂直半径**，只要用一个**斜杠（ / ）分隔**这两个值即可;
2. border-radius不仅可以接受长度值，还可以接受**百分比值**;
3. border-radius是这个简写属性，他可以写成以空格分开的四个值，这**四个值分别从左上角开始以顺时针顺序**应用到元素的各个拐角。

```css
border-radius: 水平左上 水平右上 水平右下 水平左下 /
               垂直左上 垂直右上 垂直右下 垂直左下;
```

### 案例
有了上面的知识我们可以轻松搞定自适应的椭圆。[以下代码详情点此处](http://jsrun.net/BfVKp/edit)。

#### 1.自适应椭圆:
利用百分比值会基于元素的尺寸进行解析，从而达到自适应的目的。
```css
.full {
    border-radius: 50%; 
}
```
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e114cb823a7f424fb942dd081ee4749c~tplv-k3u1fbpfcp-watermark.image)

#### 2.x轴方向左半椭圆
这个形状，水平方向上，左边的两个圆角占据了整个元素的宽度，而且右边没有圆角，因此在水平方向border-radius的值为100% 0 0 100%；垂直方向上，左上角垂直半径和左下角的垂直半径相同，都为50%，同时因为右边两个圆角水平方向的border-radius为0，故右边两个圆角垂直方向的border-radius已经不重要了，可以为任意值。
```css
.x-semi {
    border-radius: 100% 0 0 100% / 50%;  /* 相当于100% 0 0 100%/ 50% 50% 50% 50% */ ;
}
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d2b3187b1df146ed9ae420870378b2b8~tplv-k3u1fbpfcp-watermark.image)

#### 3.四分之一椭圆
```css
.quarter {
    border-radius: 100% 0 0 0;  /* 相当于100% 0 0 0 / 100% 0 0 0; */
}   
```
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/69e5974026244148968d24b88c5e5517~tplv-k3u1fbpfcp-watermark.image)

## 2.平行四边形

> 背景知识<br>
>  CSS 变形 skew


要生成平行四边形，我们可以很自然地想到利用transform的skew属性来对矩形进行斜向拉伸。
```css
transform: skewX(-45deg);
```
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/568e103c813c478abaf01565dac33e8c~tplv-k3u1fbpfcp-watermark.image)

但是这样做，导致平行四边形内的内容也发生了斜向变形。下面提供两种只让容器的形状倾斜，而保持其内容不变的方案。

### 嵌套元素方案
**嵌套一层DOM元素，再对内容进行一次反向的 skew() 变形**。

核心代码如下：（[代码链接](http://jsrun.net/ufVKp/edit)）

```html
<div class="parallelogram-1">
    <div class="text">CLICK ME</div>
</div>
```
```css
.parallelogram-1 {
     transform: skewX(-45deg);
}
.parallelogram-1>.text {
    transform: skewX(45deg);
}
```
![嵌套元素方案](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/575eb989c1da4b0da0c0d6e894f7de21~tplv-k3u1fbpfcp-watermark.image)

**缺点： 需要用两层DOM标签**

### 伪元素方案
**把所有样式（背景、边框等）应用到伪元素上，然后再对伪元素进行变形**，因为我们的内容并不是包含在伪元素里的，所以内容并不会受到变形的影响。此时，用伪元素生成的方块是重叠在内容之上的，一旦给它设置背景，就会遮住内容，我们可以**给伪元素设置z-index: -1 样式**，这样它的堆叠层次就会被推到宿主元素之后。

最终代码如下：（[代码链接](http://jsrun.net/ufVKp/edit)）

```css
.parallelogram-2 {
    position: relative;
}
.parallelogram-2::before {
    content: '';
    position: absolute;
    top: 0; right: 0; bottom: 0; left: 0;
    /* 以下为重要代码 */
    transform: skewX(-45deg);
    z-index:-1;
}
```
![伪元素方案](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26cf44940f9a4da398efc41270fe784d~tplv-k3u1fbpfcp-watermark.image)

**缺点： 变形后导致事件的点击区域与看见的实际元素区域不一致**

## 3.棱形图片

> 背景知识<br>
> CSS 变形，“平行四边形”

### 基于变形的方案
基于“平行四边形”中讨论的第一个解决方案，**需要把图片用一个div标签包裹起来，然后对其应用相反的 rotate()变形样式**，代码如下：
```html
<div class="diamond-1">
    <img src="nana.jpg" alt="">
</div>
```
```css
.diamond-1 {
    width: 200px;
    height: 200px;
    background-color: yellow;
    overflow: hidden;
    transform: rotate(45deg);
}
.diamond-1>img {
    max-width: 100%;
    transform: rotate(-45deg);  
}
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d3ab7b75bb76417cbb51b0349f8dbb53~tplv-k3u1fbpfcp-watermark.image)

我们想要得到上面图片右边的效果，但实际却得到了左边的效果。主要问题在于 max-width: 100% 这条声明。 100% 会被解析为容器（.diamond-1）的边长，但是我们想让图片的宽度与容器的对角线相等，对角线的长度为边长的$√2≈ 1.42$，因此我们可以设置max-width为142%，但此时效果如下图所示，因为通过 width 属性来放大图片时，只会以它的左上角为原点进行缩放。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1ffa7bb47f1644e5a35ab4b4ceb6abe0~tplv-k3u1fbpfcp-watermark.image)

最终我们可以**利用 scale() 变形样式来把这个图片放大1.42倍**来达到效果，最终代码如下：([代码链接](http://jsrun.net/S2VKp/edit))
```css
.diamond-1 {
    width: 200px;
    height: 200px;
    background-color: yellow;
    overflow: hidden;
    /* 以下为重要代码 */
    transform: rotate(45deg);
}
.diamond-1>img {
     /* 以下为重要代码 */
    max-width: 100%;
    transform: rotate(-45deg) scale(1.42); 
}
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e8ac511786754934a38062ee3d57bc71~tplv-k3u1fbpfcp-watermark.image)

**缺点： 需要用两层DOM标签**

### 裁切路径方案

使用**新属性clip-path**来对图片进行裁剪，但**目前这个属性浏览器的支持程度还有限**。
```css
.diamond-2 {
    width: 200px;
    height: 200px;
    clip-path: polygon(50% 0, 100% 50%, 50% 100%, 0 50%);
}
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ea95ef552434c89a284416691a446f3~tplv-k3u1fbpfcp-watermark.image)

除了浏览器支持有限外，这个属性能创造的奇迹还不止于此，它还能通过polygon()函数裁剪出各种多边形，通过circle()函数裁剪出圆，通过ellipse()函数裁剪出椭圆。

## 4.切角效果

> 背景知识<br>
> CSS 渐变， background-size ，“条纹背景”，border-image, clip-path

### CSS渐变方案
**切一个角效果：** 以右下角为例。我们可以只需要一个线性渐变就可以达到目标。这个渐变需要把一个透明色标放在切角处，然后在相同位置设置另一个色标，并且把它的颜色设置为我们想要的背景色。（[代码链接](http://jsrun.net/wwVKp/edit)）
```css
    background: #58a;
    background: linear-gradient(-45deg, transparent 15px, #58a 0);
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3ab5fdbfcccf4817b0fb7d8279deb76b~tplv-k3u1fbpfcp-watermark.image)

注意：**事实上，第一行声明并不是必需的，加上它是将其作为回退机制**：如果某些浏览器不支持 CSS渐变，那第二行声明会被丢弃，而此时我们至少还能得到一个简单的实色背景。

**切两个角效果：** 以左下角和右下角切掉为例。我们需要把这个图形看成两部分，从中间分成两半，右边使用一个右下切角的线性渐变，左边使用一个左下切角的线性渐变，**同时需要把background-repeat关掉**，代码如下：
```css
  background: #58a;
  background:linear-gradient(-45deg, transparent 15px, #58a 0) right,/*right指的background-position属性，表明从右边开始计算background-size*/
             linear-gradient(45deg, transparent 15px, #655 0) left;/*left指的background-position属性，表明从左边开始计算background-size*/
  background-size: 50% 100%;
  background-repeat: no-repeat;
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/582b8afd731d41acbd3992447cdbb3c0~tplv-k3u1fbpfcp-watermark.image)

**切四个角效果：** 明白了切两个角的原理，切四个角怎么难得到机智的我们。代码如下：
```css
    background: #58a;
    background: linear-gradient(135deg, transparent 15px, #58a 0) top left,
                linear-gradient(-135deg, transparent 15px, #58a 0) top right,
                linear-gradient(-45deg, transparent 15px, #58a 0) bottom right,
                linear-gradient(45deg, transparent 15px, #58a 0) bottom left;
    background-size: 50% 50%;
    background-repeat: no-repeat;
```
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f6c6cc673af5430baa82ba581366b8ad~tplv-k3u1fbpfcp-watermark.image)

**弧形切角：** 弧形切角原理通切四个角的效果，只不过把线性渐变变为径向渐变。代码如下：

```css
    background: #58a;
    background: radial-gradient(circle at top left, transparent 15px, #58a 0) top left, 
                radial-gradient(circle at top right, transparent 15px, #58a 0) top right,
                radial-gradient(circle at bottom right, transparent 15px, #58a 0) bottom right,
                radial-gradient(circle at bottom left, transparent 15px, #58a 0) bottom left;
    background-repeat: no-repeat;
    background-size: 50% 50%;
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/01724af5b3db4bfc8d96117574acdca1~tplv-k3u1fbpfcp-watermark.image)


### 内联 SVG 与 border-image 方案
在常规设计中，四个角的切角尺寸往往是一致的。由于border-image会解决缩放问题，而 SVG 可以实现与尺寸完全无关的完美缩放，将以下的svg应用于border-image，并设置border-image-slice为1，如虚线标注的切片方式，便可以得到一个切角效果。
```html
<svg xmlns="http://www.w3.org/2000/svg" width="3" height="3" fill="#58a">
    <polygon points="0,1 1,0 2,0 3,1 3,2 2,3 1,3 0,2"/>
</svg>
```
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/53dc98ef115d4e1b9ff8a758b47d33da~tplv-k3u1fbpfcp-watermark.image)

[切角代码](http://jsrun.net/wwVKp/edit)如下：

```css
border: 15px solid transparent;
border-image: 1 url('data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" width="3" height="3" fill="%2358a"><polygon points="0,1 1,0 2,0 3,1 3,2 2,3 1,3 0,2"/></svg>');
```
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/24d21fd5b861460cac4baa31bf56900b~tplv-k3u1fbpfcp-watermark.image)

通过以上代码，我们发现两个问题，一个是背景图片的问题，一个是切角尺寸比原来小的问题。

我们先来解决背景图片的问题。要么给 **border-image 属性值在border-image-slice后加上 fill 关键字**——这样它就不会丢掉SVG中央的那个切片了；或者给他**指定一个背景色，并且设置background-clip:padding-content**，这样还能发挥一个回退的作用。

切角尺寸比原来小的原因，请看下图。原来通过渐变生成的切角，15px是沿着渐变轴来度量的，也就是下图中的斜向距离，而通过此方法用到的15px是边框的宽度，也就是下图的水平或者垂直距离，故要使斜向距离达到15px，border的宽度应为$15×√2≈ 21.213$，近似取20。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/81cc074f2968414e93927e067ba1c4cc~tplv-k3u1fbpfcp-watermark.image)

另外，我们给边框加一个背景颜色，当border-image属性不被浏览器支持时提供一个回退方案。所以最终代码如下（[代码链接](http://jsrun.net/wwVKp/edit)）：

```css
    border: 20px solid #58a; /* 用背景色而不是transparent是为了在浏览器不支持border-image时，提供回退方案 */
    border-image: 1 url('data:image/svg+xml,<svg xmlns="http://www.w3.org/2000/svg" width="3" height="3" fill="%2358a"><polygon points="0,1 1,0 2,0 3,1 3,2 2,3 1,3 0,2"/></svg>');
    background: #58a;
    background-clip: padding-box;
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b2f011a4dbe044a1a0bdab09f9d7054f~tplv-k3u1fbpfcp-watermark.image)

### 裁切路径方案

```css
background: #58a;
clip-path: polygon(20px 0, calc(100% - 20px) 0, 100% 20px, 100% calc(100% - 20px),
            calc(100% - 20px) 100%, 20px 100%, 0 calc(100% - 20px), 0 20px);
```
### 三种方案的对比

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8de2c3a6d8a346249cd9967fd283a5fc~tplv-k3u1fbpfcp-watermark.image)

## 5.梯形标签页

> 背景知识<br>
>  基本的 3D 变形，“平行四边形”

类似于平行四边形那一节学到的方法，我们给伪元素做3D变形，代码如下：

```css
.trapezoid {
    position: relative;
    display: inline-block;
    padding: .5em 1em .35em;
    color: white;
}
.trapezoid::before {
    content: ''; /* 用伪元素来生成一个矩形 */
    position: absolute;
    top: 0; right: 0; bottom: 0; left: 0;
    z-index: -1;
    background: #58a;
    transform: perspective(.5em) rotateX(5deg);
}
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c35ddf0ab54349d59f954a31fc1e7ef3~tplv-k3u1fbpfcp-watermark.image)

只给元素做3D变得到的结果为上图1所示，图2为图1变形前后的对照图，我们可以看出默认变形中心transform-orign在元素自身的中心线上，经过旋转，元素的宽度增加，梯形占据的位置会稍微下移，高度也会有少许缩减。而文章中提到一种方法如图3所示，把变形中心transform-orign设为bottom，这时我们只要补偿高度，而这个垂直方向上的缩放程度大概为130%左右，于是得到[最终代码](http://jsrun.net/DUVKp/edit)，如下：

```css
.trapezoid {
    position: relative;
    display: inline-block;
    padding: .5em 1em .35em;
    color: white;
}
.trapezoid::before {
    content: ''; /* 用伪元素来生成一个矩形 */
    position: absolute;
    top: 0; right: 0; bottom: 0; left: 0;
    z-index: -1;
    background: #58a;
    transform: scaleY(1.3) perspective(.5em) rotateX(5deg);
    transform-origin: bottom;
}
```
于是得到一个漂亮的梯形。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ed056929fc404980bff10a7e9a14a07b~tplv-k3u1fbpfcp-watermark.image)

既然我们是要生成梯形标签，我们根据上述方案，就能轻而易举地得到如下梯形标签tab样式，由于篇幅原因，[代码请点此处](http://jsrun.net/DUVKp/edit)。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b7ae4de41d254d55acb13d72e6b62a49~tplv-k3u1fbpfcp-watermark.image)

## 6.简单的饼图

> 背景知识<br>
>  CSS 渐变，基本的 SVG，CSS 动画，“条纹背景”，“自适应的椭圆”，conic-gradient

我们案例用黄绿色（yellowgreen）表示底色，并采用棕色（#655）来显示比率。

### 基于transform的方案
原理是这样的（篇幅比较长），我们**先利用linear-gradient把圆分为两部分，然后再用伪元素构造成一个半圆盖上去，通过旋转伪元素来决定露出多大的扇区**。

先构造一个两种颜色的圆，代码如下：
```css
.pie {
    width: 100px;
    height: 100px;
    border-radius: 50%;
    background: yellowgreen;
    background-image:linear-gradient(to right, transparent 50%, #655 0);
}
```


![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7fc6fec013334577800583223ac9860d~tplv-k3u1fbpfcp-watermark.image)

然后再利用伪元素构造一个半圆盖在此元素上，并把旋转中心设置在圆心。

```css
pie::before {
    content: '';
    display: block;
    margin-left: 50%;
    height: 100%;
    border-radius: 0 100% 100% 0 / 50%;
    transform-origin: left;
}
```
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/20f3c7b0b6b641dfbd82d85d19a839c0~tplv-k3u1fbpfcp-watermark.image)

若要得到一个显示率为0-50%的饼图，只需把伪元素背景设为黄绿色，然后旋转相应的度数，我们以20%为例，旋转
$ 360deg×0.2 = 72deg = 0.2turn $ （注：turn为CSS3角度单位，表示圈，1圈为360deg ）。
```css
pie::before {
    content: '';
    display: block;
    margin-left: 50%;
    height: 100%;
    border-radius: 0 100% 100% 0 / 50%;
    transform-origin: left;
    background-color: inherit; /* 继承父元素的黄绿色 */
    transform: rotate(.2turn); /* 旋转0.2圈 */
}
```

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/41b5550dd5334c46ad5902bc9191a9fe~tplv-k3u1fbpfcp-watermark.image)

若要得到一个显示率大于50%的饼图，只需把伪元素背景设为棕色，然后旋转相应的度数，我们以60%为例，旋转
$ 360deg×(0.6-0.5) = 36deg = 0.1turn $。
```css
pie::before {
    content: '';
    display: block;
    margin-left: 50%;
    height: 100%;
    border-radius: 0 100% 100% 0 / 50%;
    transform-origin: left;
    background-color: #655; /* 棕色 */
    transform: rotate(.1turn); /* 旋转0.1圈 */
}
```
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8c5a79205d984e099a0aac287700ae1c~tplv-k3u1fbpfcp-watermark.image)

知道了原理，我们接下来需要把这两种情况封装一下。我们先来看一个动画，以下代码实现了一个饼图从 0 变化到 100% 的动画。

```css
@keyframes spin {
    to { transform: rotate(.5turn); }
}
@keyframes bg {
    50% { background: #655; }
}
.pie::before {
    content: '';
    display: block;
    margin-left: 50%;
    height: 100%;
    border-radius: 0 100% 100% 0 / 50%;
    background-color: inherit;
    transform-origin: left;
    animation: spin 3s linear infinite,
    bg 6s step-end infinite;
}
```
再来看一个关于animation-delay的规范。

> “一个负的延时值是合法的。与 0s 的延时类似，它意味着动画会立即开始播放，但会自动前进到延时值的绝对值处，就好像动画在过去已经播放了指定的时间一样。因此实际效果就是动画跳过指定时间而从中间开始播放了。”<br>
> ——CSS 动画（第一版）（http://w3.org/TR/css-animations/#animation-delay）

在了解了以上动画和animation-delay的规范后，我们将使用上面的动画来解决这个封装，但动画必须处于暂停状态，且必须暂停在我们想要的位置。我们要**用负的动画延时来直接跳至动画中的任意时间点，并且定格在那里**。

我们设置动画的总时长为100s，我们可以用内联样式的方式为其设置 animation-delay 属性，然后再在伪元素上应用 animation-delay: inherit属性。综合以上要素，如果要让饼图显示为 20% 和 60%，则html结构代码为:
```html
<div class="pie" style="animation-delay: -20s"></div>
<div class="pie" style="animation-delay: -60s"></div>
```
进一步优化html代码结构，优化后代码如下：
```html
<div class="pie">20%</div>
<div class="pie">60%</div>
```
这时，我们采用一段js脚本来把animation-delay 写到内联样式中，同时用color: transparent 来把文字隐藏起来。
```js
document.querySelectorAll('.pie').forEach(function (pie) {
    var p = parseFloat(pie.textContent);
    pie.style.animationDelay = '-' + p + 's';
});
```
最终改良版的css代码如下（[详情点击此处](http://jsrun.net/c7VKp/edit)）：
```css
.pie {
    position: relative;
    width: 100px;
    line-height: 100px;
    border-radius: 50%;
    background: yellowgreen;
    background-image: linear-gradient(to right, transparent 50%, #655 0);
    color: transparent;
    text-align: center;
}

@keyframes spin {
    to {
        transform: rotate(.5turn);
    }
}

@keyframes bg {
    50% {
        background: #655;
    }
}

.pie::before {
    content: '';
    position: absolute;
    top: 0;
    left: 50%;
    width: 50%;
    height: 100%;
    border-radius: 0 100% 100% 0 / 50%;
    background-color: inherit;
    transform-origin: left;
    animation: spin 50s linear infinite, bg 100s step-end infinite;
    animation-play-state: paused;
    animation-delay: inherit;
}
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71ba212cef384b8faeea9d857fb33da1~tplv-k3u1fbpfcp-watermark.image)
### SVG方案
原理：我们先从一个svg开始，画一个半径为50的圆。
```html
<svg width="100" height="100">
    <circle r="30" cx="50" cy="50" />
</svg>
```
给圆加点基础样式，描边，如下图1所示。
```css
circle {
    fill: yellowgreen;  /* 填充颜色 */
    stroke: #655;  /* 描边颜色 */
    stroke-width: 30; /* 描边宽度 */
}
```
在上面的样式中追加下面一行代码，使描边变成虚线，如下图2所示。
```css
.stroke-dasharray: 20 10; /* 虚线的线段长度为20 且间隙长度为10 */
```
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9e25be07107542f1b2c31751b4120bde~tplv-k3u1fbpfcp-watermark.image)

当我们把这个虚线描边的线段长度指定为0 ，并且把虚线间隙的长度设置为等于或大于整个圆周的长度，这里是$2π×30≈ 189$时。我们可以看到，它完全去除了描边效果，只剩下绿色的圆形。当我们开始增加第一个值时，整个圆周上覆盖的长度正是我们给它指定的长度值。

![QQ图片20210622222600.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e434ed2c78894e00b2f2dda8dbd2b286~tplv-k3u1fbpfcp-watermark.image)

根据这个原理，我们优化一下代码，首先我们为了便于计算，我们把**圆的周长设为100**，那么半径就是$100/2π≈ 16$，然后我们还需要把**svg图形以逆时针方向旋转 90°，使描边起始点位置来到0点钟方向**，这样我们就得到了一个60%比率的饼图，代码如下（[详情点此处](http://jsrun.net/c7VKp/edit)）：

```html
<svg viewBox="0 0 32 32">
    <circle class="circle" r="16" cx="16" cy="16" />
</svg>
```
```css
svg {
    width: 100px; height: 100px;
    transform: rotate(-90deg);
    background: yellowgreen;
    border-radius: 50%;
}
.circle {
    fill: yellowgreen;  /* 填充颜色 */
    stroke: #655;  /* 描边颜色 */
    stroke-width: 32; /* 描边宽度 */
    stroke-dasharray: 60 100; /* 关键代码 虚线的线段长度为60 且间隙长度为100*/ 
}
```

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/85378ddcb21f41d890611a0d7f27cebe~tplv-k3u1fbpfcp-watermark.image)

利用这个原理，我们也可以实现多色饼图，我们每多一种颜色多添加一个circle层，同时越靠近起点的circle层在图层的越上层。代码如下（[详情点此处](http://jsrun.net/c7VKp/edit)）：

```html
<svg viewBox="0 0 32 32">
    <circle class="circle1" r="16" cx="16" cy="16" />
    <circle class="circle2" r="16" cx="16" cy="16" />
</svg>
```
```css
svg {
    width: 100px; height: 100px;
    transform: rotate(-90deg);
    background: yellowgreen;
    border-radius: 50%;
}
.circle1 {
   fill: yellowgreen;
    stroke: #655; 
    stroke-width: 32;
    stroke-dasharray: 60 100; 
}
.circle2 {
    fill: transparent;
    stroke: grey;
    stroke-width: 32;
    stroke-dasharray: 20 100; 
}
```
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ad1e962236644c74b6bf87e8f1692cd0~tplv-k3u1fbpfcp-watermark.image)

### conic-gradient方案
结合以前学的linear-gradient的知识，利用圆锥渐变conic-gradient实现多种颜色的饼图真是太easy了~墙裂推荐！代码如下（[详情点此处](http://jsrun.net/c7VKp/edit)）：

```css
.conic {
    width: 100px;
    height: 100px;
    border-radius: 50%;
    background: conic-gradient(grey 0 20%, #655 0 60%, yellowgreen 0 100%);
}
```
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71e1326ce9ae4756979d31f230d2262f~tplv-k3u1fbpfcp-watermark.image)

通过三种生成饼图的方案，可以看出：svg方案和conic-gradient方案代码简洁，且能轻松实现多种颜色的饼图，非常值得推荐~
