---
layout: post
categories: js
title: "网页性能"
subtitle: "互联网有一项闻名的8秒准绳。用户在访问Web网页时，若是时间超过8秒就会感应不不耐烦，若是加载时间太长，他们就会放弃访问。大部分用户希望网页能在2秒之内就完成加载。事实上，加载时间每多1秒，你就会流失7%的用户。8秒并不是切确的8秒钟，只是向网站开发者剖清楚明了加载时间的重要性。"
featured-image: /images/img/anchor.jpg
tags: ['性能优化']
date-string: Jun 11, 2020
---

#### 前言

如果网页响应非常慢，大多数用户应该都会选择关闭这个页面，作为开发者，我们肯定不愿意看到这种情况，那我们应该怎样提高性能呢？

## 一、网页生成的过程

要理解网页的性能为什么不好，就必须了解网页是怎么生成的。

网页的生成过程，可以分为5步：

1. HTML代码转化为DOM tree
2. CSS代码转化为CSS tree （CSSOM  CSS Object Model）
3. 合成DOM tree 和 CSS tree，生成一棵渲染树（包含每个节点的视觉信息）
4. 生成(layout)布局，也就是将所有渲染树的所有节点进行平面合成；
5. 将布局绘制在屏幕上（paint)

前三步是非常快的，第四步和第五步比较耗时；

- "生成布局"（flow）和"绘制"（paint）这两步，合称为"渲染"（render）。

<img src="/images/img/render.png" alt="" >


## 二、重排和重绘

网页生成的时候，至少会渲染一次。用户访问的过程中，还会不断重新渲染。

导致网页重新渲染的几种可能的情况：
- 修改DOM
- 修改样式表
- 用户事件（比如鼠标悬停、页面滚动、输入框键入文字、改变窗口的大小等等）

重新渲染，就需要重新生成布局和重新绘制。

- 生成布局：即重排；（reflow）
- 重新绘制：即重绘；（repaint）

需要注意的是：

- “重绘”不一定需要“重排”，比如改变某个网页元素的颜色，就只会触发“重绘”，不会触发“重排”，因为布局没有改变；
- “重排”必然会导致“重绘”，比如改变了一个网页元素的位置，就会同时出发“重排”和“重绘”，因为布局发生了改变。

## 三、对于性能的影响

重排和重绘不断触发是不可避免的。但是，它们非常耗费资源，是导致网页性能低下的根本原因。

#### 提高网页性能，就是要降低“重排”和“重绘”的频率和成本，尽量减少触发重新渲染。

我们前面已经提到，DOM变动和样式变动，都会触发重新渲染。但是浏览器已经很智能了，会尽量把所有的变动集中在一起，排成一个队列，然后一次性执行，尽量避免多次重新渲染。

```js
  div.style.color = 'blue';
  div.style.marginTop = '30px';
```

上面的代码中，div元素有两个样式变动，但是浏览器只会触发一次重排和重绘。

如果写的不好，就会触发两次重排和重绘。

```js
  div.style.color = 'blue';

  let margin = parseInt(div.style.marginTop);

  div.style.marginTop = (margin + 10) + 'px';
```

上面代码对div元素设置背景色以后，第二行要求浏览器给出该元素的位置，所以浏览器不得不立即重排。

一般来说，样式的写操作之后，如果有下面这些属性的读操作，都会引发浏览器立即重新渲染。

```js
  offsetTop/offsetLeft/offsetWidth/offsetHeight

  scrollTop/scrollLeft/scrollWidth/scrollHeight

  clientTop/clientLeft/clientWidth/clientHeight

  getComputedStyle()
```
所以，从性能角度考虑，尽量不要把读操作和写操作，放在一个语句里面。

```js
// bad
div.style.left = div.offsetLeft + 10 + "px";
div.style.top = div.offsetTop + 10 + "px";

// good
  let left = div.offsetLeft;

  let top  = div.offsetTop;

  div.style.left = left + 10 + "px";

  div.style.top = top + 10 + "px";
```

一般的规则是：

```js

  样式表越简单，重排和重绘就越快。

  重排和重绘的DOM元素层级越高，成本就越高。

  table元素的重排和重绘成本，要高于div元素

```

## 四、通过降低浏览器重新渲染的频率，从而提高性能的几个小技巧

1. DOM的多个读操作（或者写操作），应该放在一起。不要两个读操作之间，加入一个写操作。

2. 如果某个样式是通过重排得到的，那最好缓存以下结果，避免下次用到时，浏览器又要重排。

3. 不要一条条地改变样式，而要通过改变class或者cssText属性，一次性改变样式。

    ```js
      // bad
      let left = 10;
      let top = 10;
      el.style.left = left + "px";
      el.style.top  = top  + "px";

      // good
      el.className += " theclassname";

      // good
      el.style.cssText += "; left: " + left + "px; top: " + top + "px;";
    ```

4. 尽量使用离线DOM，而不是真实的DOM，来改变元素样式。比如，使用 cloneNode() 方法，在克隆的节点上进行操作，然后再用克隆的节点替换原始节点。再比如，操作Document Fragment对象，完成后再把这个对象加入DOM。

5. 先将元素设为display: none（需要1次重排和重绘），然后对这个节点进行100次操作，最后再恢复显示（需要1次重排和重绘）。这样一来，你就用两次重新渲染，取代了可能高达100次的重新渲染。

6. position属性为absolute或fixed的元素，重排的开销会比较小，因为不用考虑它对其他元素的影响。

7. 只在必要的时候，才将元素的display属性为可见，因为不可见的元素不影响重排和重绘。另外，visibility : hidden的元素只对重绘有影响，不影响重排。

8. 使用虚拟DOM；

9. 【个人很少使用，可以尝试】 可以使用window.requestAnimationFrame()、window.requestIdleCallback()这两个方法来调节重新渲染。


## 总结
    以上就是在react中实现锚链接的方式以及监听滚动条的方法,当然也有实现类似效果的其他方法,后续会逐步补充!