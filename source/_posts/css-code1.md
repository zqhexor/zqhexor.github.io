---
title: 《CSS揭秘》精读笔记（一）背景与边框
date: 2021-06-02 10:22:01
tags: CSS
---
## 前言
最近在看《CSS揭秘》这本书，平时做笔记都是用OneNote粗略记一些我认为重要的内容，自己看懂了，却有时候不知道怎么把知识说清楚。这次想换个方式输出，一方面想练习一下自己的表达和总结能力，另一方面也可以督促自己的学习。<br>
先介绍一下这本书，《CSS揭秘》是由W3C CSS工作组的成员编写的，适合于有一定CSS基础的同学阅读。全篇主要内容包括背景与边框、形状、视觉效果、字体排印、用户体验、结构与布局、过渡与动画七大主题，涵盖了47个鲜为人知的CSS技巧。<br>
本文为《CSS揭秘》的第一部分背景与边框的笔记，精要记录了相关经典案例的实现方式和原理，同时在最后附上了个人对此章的一个总结和知识的梳理的脑图，帮助大家承上启下串接和回忆所学内容。
## 1.半透明边框
我们给边框设置成一个半透明色，但默认情况下，背景会延伸到边框所在的区域下层。我们用一个虚线边框样式就可以发现这一点。
```css
    border: 10px dashed hsla(120, 50%, 50%, 0.5);
    background: yellow;
```
![image1.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/29e8e25d849e4793a1215824eb7903a3~tplv-k3u1fbpfcp-watermark.image)
<br/>
如果我门要设置一个纯种的半透明边框（不希望背景侵入边框所在的范围），我们要调整**background-clip属性**（默认值是border-box），把其**变为padding-box**。
```css
    border: 10px dashed hsla(120, 50%, 50%, 0.5);
    background: yellow;
    background-clip: padding-box;
```
![image2.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/34ec95022d184b6681f5c4cd4401b04b~tplv-k3u1fbpfcp-watermark.image)

## 2.多重边框
### box-shadow方案
原理：
1. box-shadow接收的第四个参数（称作“扩张半径”），通过指定正值或负值，可以让投影面积加大或者减小。一个正值的扩张半径加上两个为零的偏移量以及为零的模糊值，得到的“投影”其实就像一道实线边框；
2. box-shadow支持逗号分隔语法，因此，我们可以创建任意数量的投影。
3. box-shadow 是层层叠加的，第一层投影位于最顶层，依次类推。因此，我们只需要按此规律调整扩张半径。
```css
    background: yellow;
    box-shadow: 0 0 0 10px blue, 0 0 0 15px deeppink;
```
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e8ea4927c682419c827586119ef31473~tplv-k3u1fbpfcp-watermark.image)
<br>
注意事项：
    **box-shadow所创建出的假“边框”出现在元素的外圈。它们并不会响应鼠标事件，比如悬停或点击**。如果这一点非常重要，你可以给**box-shadow 属性加上 inset 关键字，来使投影绘制在元素的内圈**。
### outline方案
```css
    background: yellow;
    border: 10px solid blue;
    outline: 5px solid deeppink;
```
使用outline的优点：
1. 可以用它来构造虚线边框；
2. 可以通过 outline-offset 属性来控制它跟元素边缘之间的间距。
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/78fa180b8a764b40a39632ad1c00f9e1~tplv-k3u1fbpfcp-watermark.image)
<br>
注意事项：
    **outline只适用于双层“边框”的场景，边框不会贴合 border-radius 属性产生的圆角**
    
## 3.灵活背景定位
需求：实现一个背景图片距离目标区域右边20px,下方10px
### background-position 的扩展语法方案
background-position 属性已经得到扩展，它**允许我们指定背景图片距离任意角的偏移量，只要我们在偏移量前面指定关键字**。
```css
  background: url(rabbit.png) no-repeat #2aaa71;
  background-position: right 20px bottom 10px;
```
![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6d234c2689c14bb5b0639f5eb938dfb8~tplv-k3u1fbpfcp-watermark.image)
<br>
我们还需要提供一个合适的回退方案。因为对上述方案来说，在不支持 background-position 扩展语法的浏览器中，背景图片会紧贴在左上角（背景图片的默认位置）。把bottom right 定位值写进 background 的简写属性中。
```css
background: url(rabbit.png) no-repeat bottom right #58a;
background-position: right 20px bottom 10px;
```
### background-origin 方案
在给背景图片设置距离某个角的偏移量时，有一种情况极其常见：偏移量与容器的内边距一致。如果采用上面提到的 background-position 的扩展语法方案，代码看起来会是这样的：
```css
padding: 10px 20px;
background: url(rabbit.png) no-repeat bottom right #58a;
background-position: right 20px bottom 10px;
```
但代码不够DRY：每次改动内边距的值时，我们都需要在四个地方更新这个值！还有一个更简单的办法可以实现这个需求：让它自动地跟着我们设定的内边距走，不用另外声明偏移量的值。
<br>
默认情况下， background-position 是以 padding box 为准的；**把background-origin 的值改成 content-box， background-position 属性中使用的边角关键字将会以内容区的边缘作为基准**
```css
padding: 10px 20px;
background: url(rabbit.png) no-repeat #58a bottom right; 
background-origin: content-box;
```
### calc() 方案
```css
background: url("code-pirate.svg") no-repeat;
background-position: calc(100% - 20px) calc(100% - 10px);
```

## 4.边框内圆角
> 背景知识<br>
> **box-shadow，outline， “多重边框”**

描边并不会跟着元素的圆角走（因而显示出直角），但 box-shadow 却是会的。因此，如果我们把这两者叠加到一起（**设置一个描边outline，然后再用box-shadow会刚好填补描边和容器圆角之间的空隙**），这两者的组合达成了我们想要的效果。
```css
background: tan;
border-radius: .8em;
padding: 1em;
box-shadow: 0 0 0 .6em #655;

```
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5aeccb5fb4064494ab08f0e47142eb2f~tplv-k3u1fbpfcp-watermark.image)
<br>
到底多大的投影扩张值可以填补这些空隙呢？
<br>
通过下图我们可以看出，为了让这个效果得以达成，扩张半径需要比描边的宽度值小，但它同时又要比$(√2-1)r $大。为了避免每次都要计算，你**可以直接使用圆角半径的一半**，因为 $(√2-1)<0.5 $。
<br>
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a366304803d42ab9b94531fe1b5904e~tplv-k3u1fbpfcp-watermark.image)

## 5.条纹
### 水平条纹
我们先来看按默认渐变生成的背景图。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/13b1fd1e412d4cf5ac1119a518b33298~tplv-k3u1fbpfcp-watermark.image)

我们试着把这两个色标拉近一点。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/603592de0c754fb5a1ae50eb853847e4~tplv-k3u1fbpfcp-watermark.image)

如果多个色标具有相同的位置，它们会产生一个无限小的过渡区域，过渡的起止色分别是第一个和最后一个指定值。从效果上看，颜色会在那个位置突然变化，而不是一个平滑的渐变过程。

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0a8314770caf46f58313dc6f3e6cc47f~tplv-k3u1fbpfcp-watermark.image)
我们通过background-size来调整其尺寸，又于背景在默认情况下是重复平铺的，此时整个容器其实已经被填满了水平条纹
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c48695475d7b499dbbe9192feba8504c~tplv-k3u1fbpfcp-watermark.image)
为了避免每次改动条纹宽度时都要修改两个数字，我们可以再次从规范那里找到捷径。
> 如果某个色标的位置值比整个列表中在它之前的色标的位置值都要小，则该色标的位置值会被设置为它前面所有色标位置值的最大值<br>
>                 ——CSS 图像（第三版）（http://w3.org/TR/css3-images）

于是得到横条纹的最终代码如下：
```css
background: linear-gradient(#fb3 30%, #58a 0);
background-size: 100% 30px
```

利用这个原理，我们可以生成任意比例，多种颜色的条纹，下面代码可以生成三种颜色的水平条纹。

```css
background: linear-gradient(#fb3 33.3%, #58a 0, #58a 66.6%, yellowgreen 0);
background-size: 100% 45px;
```
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7a4b44681e0a4debad0fea8970e75a9d~tplv-k3u1fbpfcp-watermark.image)
### 垂直条纹
垂直条纹的代码跟水平条纹几乎是一样的，差别主要在于：我们需要在开头加上一个额外的参数来指定渐变的方向。另外，还需要把 background-size 的值颠倒一下：

```css
background: linear-gradient(to right, #fb3 50%, #58a 0);
background-size: 30px 100%;
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3df4909f70bf48d4829ddce75e32b0e6~tplv-k3u1fbpfcp-watermark.image)
### 斜条纹
我们利用 repeating-linear-gradient()来实现斜条纹。
```css
background: repeating-linear-gradient(45deg,#fb3, #fb3 15px, #58a 0, #58a 30px);
```
以上代码就相当于linear-gradient做无限的循环。

```css
background: linear-gradient(45deg,
#fb3, #fb3 15px, #58a 0, #58a 30px,
#fb3 0, #fb3 45px, #58a 0, #58a 60px,
#fb3 0, #fb3 75px, #58a 0, #58a 90px,
#fb3 0, #fb3 105px, #58a 0, #58a 120px ...);
```

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/361cba69ec8f4a29831aeac08e0b152d~tplv-k3u1fbpfcp-watermark.image)

### 灵活的同色系条纹
在大多数情况下，我们想要的条纹图案并不是由差异极大的几种颜色组成的，这些颜色往往属于同一色系，只是在明度方面有着轻微的差异。

```css
background: repeating-linear-gradient(30deg, #79b, #79b 15px, #58a 0, #58a 30px);
```

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/941dceedbad04207be3914f3cccacd35~tplv-k3u1fbpfcp-watermark.image)
<br>
如果我们按上述方式写，两种颜色之间的关系在代码中并没有体现出来。此外，如果我们想要改变这个条纹的主色调，甚至需要修改四处，采用如下方式只需要修改一个地方（background）就可以改变所有颜色。

```css
background: #58a;
background-image: repeating-linear-gradient(30deg, hsla(0,0%,100%,.1), hsla(0,0%,100%,.1) 15px, transparent 0, transparent 30px);
```
## 6.复杂的背景图案
> 背景知识<br>
> **CSS 渐变，“条纹背景”**
### 网格
```css
background: white;
background-image: linear-gradient(90deg, rgba(200,0,0,.5) 50%, transparent 0),linear-gradient(rgba(200,0,0,.5) 50%, transparent 0);
background-size: 30px 30px;
```
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5a0d4378e86e4604ba5443c26d647e32~tplv-k3u1fbpfcp-watermark.image)
### 波点
我们先用径向渐变生成下图案：
```css
    background: #655;
    background-image: radial-gradient(tan 30%, transparent 0);
    background-size: 30px 30px;
```
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b29af8ca3eae4ee2ad5a4e5733202d6b~tplv-k3u1fbpfcp-watermark.image)
<br>
但这样还不是真正的波点图案，我们可以把上述图案阵列**使用背景定位错开**，然后再把两者叠加。得到如下图案：
```css
background: #655;
background-image: radial-gradient(tan 30%, transparent 0),radial-gradient(tan 30%, transparent 0);
background-size: 30px 30px;
background-position: 0 0, 15px 15px;
```
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fe7094eb450b40e8af7de41b80f0ddff~tplv-k3u1fbpfcp-watermark.image)
### 棋盘
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/694a664f270f4d62a2bf207872e31896~tplv-k3u1fbpfcp-watermark.image)
<br>
我们先来看棋盘的一块图形，棋盘的一块图形可以看成以下两部分组合而成。
<br>
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f98fb3df1b874592b065728f952b4a4c~tplv-k3u1fbpfcp-watermark.image)
<br>
棋盘的第一部分，我们很自然的可以用线性渐变路径得到，代码如下：

```css
    background: #eee;
    background-image: linear-gradient(45deg, #bbb 25%, transparent 0,transparent 75%,#bbb 0);
    background-size: 30px 30px;
```
棋盘的第二部分，我们可以参照波点图案的生成思路，把第一部分往右移15px，往下移动15px，便可得到第二部分，代码如下：
```css
    background: #eee;
    background-image: linear-gradient(45deg, #bbb 25%, transparent 0,transparent 75%,#bbb 0);
    background-size: 30px 30px;
    background-position: 15px 15px;
```
再把两者组合起来，代码如下：

```css
background: #eee;
background-image:
    linear-gradient(45deg, rgba(0,0,0,.25) 25%, transparent 0, transparent 75%, rgba(0,0,0,.25) 0),
    linear-gradient(45deg,rgba(0,0,0,.25) 25%, transparent 0,transparent 75%, rgba(0,0,0,.25) 0);
background-size: 30px 30px;
background-position: 0 0, 15px 15px;
```
## 7.伪随机背景
> 背景知识<br>
> **CSS 渐变，“条纹背景”，“复杂的背景图案”**

一种颜色作为底色，另三种颜色作为条纹，然后再让条纹以不同的间隔进行重复平铺，再用 background-size 来控制条纹的间距。**在background-image中，前面的图案会位于图层的上层，上层的图层会覆盖下层的图层，background位于最底层**。

```css
background: hsl(20, 40%, 90%);
background-image:
    linear-gradient(90deg, #fb3 10px, transparent 0),
    linear-gradient(90deg, #ab4 20px, transparent 0),
    linear-gradient(90deg, #655 20px, transparent 0);
background-size: 80px 100%, 60px 100%, 40px 100%;
```

![QQ图片20210530152133.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0bce30ad7f50458393f87b5f16633765~tplv-k3u1fbpfcp-watermark.image)
<br>
可以看出图案每隔240px就会重复一次。这个最小重复图案的尺寸即为所有 background-size 的最小公倍数，而 80、60 和 40的最小公倍数正是 240。根据这个逻辑，要让这种随机性更加真实，我们得把最小重复图案的尺寸最大化。为了让最小公倍数最大化，这些数字最好是“**相对质数**”。这个技巧被 Alex Walker 定名为“蝉原则”。

```css
background: hsl(20, 40%, 90%);
background-image:
    linear-gradient(90deg, #fb3 11px, transparent 0),
    linear-gradient(90deg, #ab4 23px, transparent 0),
    linear-gradient(90deg, #655 41px, transparent 0);
background-size: 41px 100%, 61px 100%, 83px 100%;
```
这样最小重复图案的尺寸现在是 $41×61×83=207583$ 像素。

## 8.连续的图像边框
> 背景知识<br>
> **CSS 渐变，“条纹背景”，“复杂的背景图案”**
### 实现原理
实现原理：边框设置为透明色，接着用两层背景来实现，上面一层为背景颜色，下面一层为图像边框，**同时给两层背景指定不同的绘制区域background-clip**，**上层背景指定为padding-box，下层图片边框为border-box**，同时由于下层图片的background-origin默认放置在 padding-box 的原点，而现在绘制区域background-clip变为border-box，故需把**下层图片的background-origin变为border-box**,代码如下：

```css
padding: 1em;
border: 1em solid transparent;
background: linear-gradient(white, white),
            url(sakura.jpg);
background-size: cover;
background-clip: padding-box, border-box;
background-origin: border-box;
```
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a647f03db58743b393412ab2b137033f~tplv-k3u1fbpfcp-watermark.image)
<br>
这些新属性也是可以整合到 background 这个简写属性中的，这样可以显著地减少代码量：

```css
padding: 1em;
border: 1em solid transparent;
background:
    linear-gradient(white, white) padding-box,
    url(sakura.jpg) border-box 0 / cover
```
为了更好的阅读代码，附一个background简写属性的顺序 。
```css
background: 路径 background-origin background-clip background-position/ background-size;
```
### 案例
 #### 老式信封样式边框 vintage-envelope

```css 
padding: 1em;
border: 1em solid transparent;
background: 
    linear-gradient(white, white) padding-box,
    repeating-linear-gradient(-45deg, red 0, red 12.5%, transparent 0, transparent 25%,
                                #58a 0, #58a 37.5%, transparent 0, transparent 50%) border-box 0 / 5em 5em;
```
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/aa19a16d1b1d4657a023d219a574a8f7~tplv-k3u1fbpfcp-watermark.image)

#### 蚂蚁行军边框 marching-ants
蚂蚁行军效果，我们将会用到“老式信封”技巧的一个变种。我们将把条纹转变为黑白两色，并把边框的宽度减少至 1px，然后再把 background-size 改为某个合适的值。最后，我们把 background-position 以动画的方式改变为 100% ，就可以让它滚动起来了

```css
@keyframes ants { to { background-position: 100% } }

.marching-ants {
    width: 200px;
    height: 200px;
    padding: 1em;
    border: 1px solid transparent;
    background:
        linear-gradient(white, white) padding-box,
        repeating-linear-gradient(-45deg, black 0, black 25%, white 0, white 50%) border-box 0 / .6em .6em;
    animation: ants 12s linear infinite;
}
```
[动态效果点此处](http://jsrun.net/BZVKp/edit)
<br>
![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/765d795d98d04e258c4e199f558674cb~tplv-k3u1fbpfcp-watermark.image)

## 总结

![2021-05-31_045437.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59d96351f279488694ca73fe20afe50e~tplv-k3u1fbpfcp-watermark.image)
<br>

> 名词解释：<br>
> DRY：Don't Repeat Yourself，常指代码简洁<br>
> WET：We Enjoy Typing，常指代码重复冗杂<br>
> 蝉原则：以质数作为循环周期来增加“自然随机性”的策略，可以以最小成本实现更自然的随机效果。



