---
layout: post
title: 网页渲染原理以及性能优化
categories: [HTML]
description: 
keywords: DOM Tree, Render Tree, repaint, reflow
---
性能是衡量一个网站质量的重要指标，要解决网站的性能问题，就得了解网站是如何生成的。
## 一道面试题：讲讲输入完网址按下回车，到看到网页这个过程中发生了什么？
1. 域名解析
2. 发起TCP的3次握手
3. 建立TCP连接后发起http请求
4. 服务器端响应http请求，浏览器得到html代码
5. HTML代码转化成DOM
6. CSS代码转化成CSSOM（CSS Object Model）
7. 结合DOM和CSSOM，生成一棵渲染树（render tree包含每个节点的视觉信息）
8. 生成布局（layout），即将所有渲染树的所有节点进行平面合成
9. 将布局绘制（paint）在屏幕上
    
<br>

![浏览器渲染大致过程](/images/posts/html/render-process.png)


> 其中比较耗时的是生成布局和布局绘制最后两步。"生成布局"（flow）和"绘制"（paint）这两步，合称为"渲染"（render）。

## 重排和重绘

网页生成后，会根据用户的操作不断进行回馈，重新渲染。重新渲染就会引发重新生成布局和重新绘制，既重排和重绘的过程。

重排：重新生成布局
重绘：重新绘制（若布局不变，则只会触发重绘，例如：修改背景颜色）

> 一道题目：
> 以下哪些操作会触发Reflow：()
>```js
> var obj = document.getElementById("test");
> 1. alert(obj.className)
> 2. alert(obj.offsetHeight)
> 3. obj.style.height = "100px"
> 4. obj.style.color = 'red"
> ```

正确答案： BC。
B计算了offsetHeight，C重新设置了高度。
A打印出类名，无影响。
D重新设置背景，引起重绘。


reflow：几乎是无法避免的。现在界面上流行的一些效果，比如树状目录的折叠、展开（实质上是元素的显 示与隐藏）等，都将引起浏览器的 reflow。鼠标滑过、点击……只要这些行为引起了页面上某些元素的占位面积、定位方式、边距等属性的变化，都会引起它内部、周围甚至整个页面的重新渲 染。通常我们都无法预估浏览器到底会 reflow 哪一部分的代码，它们都彼此相互影响着。

repaint：如果只是改变某个元素的背景色、文 字颜色、边框颜色等等不影响它周围或内部布局的属性，将只会引起浏览器 repaint（重绘）。repaint 的速度明显快于 reflow

下面情况会导致reflow发生

1：改变窗口大小

2：改变文字大小

3：内容的改变，如用户在输入框中敲字

4：激活伪类，如:hover

5：操作class属性

6：脚本操作DOM

7：计算offsetWidth和offsetHeight

8：设置style属性


## 对性能的影响
重排和重绘会不断触发，这是不可避免的。但是，它们非常耗费资源，是导致网页性能低下的根本原因。

提高网页性能，就是要降低"重排"和"重绘"的频率和成本，浏览器已经很智能了，会尽量把所有的变动集中在一起，排成一个队列，然后一次性执行，尽量避免多次重新渲染。

```js
//good
div.style.color = 'blue';
div.style.marginTop = '30px';
//bad
div.style.color = 'blue';
var margin = parseInt(div.style.marginTop);
div.style.marginTop = (margin + 10) + 'px';
//上面代码对div元素设置背景色以后，第二行要求浏览器给出该元素的位置，所以浏览器不得不立即重排。

/*offsetTop/offsetLeft/offsetWidth/offsetHeight
 *scrollTop/scrollLeft/scrollWidth/scrollHeight
 *clientTop/clientLeft/clientWidth/clientHeight
 *getComputedStyle()
*/
```

## 提高性能的几个技巧

1. DOM的多个读操作应该放在一起，不要读写交叉写。
2. 样式如何通过重排获得，应该尽量缓存起来。
3. 减少一条一条的改变style，通过class改变
4. 减少直接在Dom上的操作，而是cloneNode（），操作完之后再替换元素结点。
5. 类似上一条，需要多次操作的，可以先将display设置为none
6. 只在必要的时候，才将元素的display属性为可见，因为不可见的元素不影响重排和重绘。另外，visibility : hidden的元素只对重绘有影响，不影响重排。
7. 使用虚拟DOM的脚本库，比如React等。
8. position属性为absolute或fixed的元素，重排的开销会比较小，因为不用考虑它对其他元素的影响。
   
## 刷新率 （FPS）

很多时候，密集的重新渲染是无法避免的，比如scroll事件的回调函数和网页动画。

网页动画的每一帧（frame）都是一次重新渲染。每秒低于24帧的动画，人眼就能感受到停顿。一般的网页动画，需要达到每秒30帧到60帧的频率，才能比较流畅。如果能达到每秒70帧甚至80帧，就会极其流畅。


