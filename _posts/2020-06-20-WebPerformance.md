---
layout: post
categories: js
title: "网页性能工具——performance（Timeline）"
subtitle: "Chrome Devtool Performance，是基于谷歌浏览器下的一个开发调测工具，它的前身是Timeline。主要功能是分析运行时性能表现"
featured-image: /images/img/performance.jpeg
tags: ['性能优化']
date-string: Jun 20, 2020
---

在开始介绍Chrome Devtool Performance工具的应用之前，我们先来了解一下什么是“刷新率”？

## 刷新率

很多时候，密集的重新渲染是无法避免的，比如scroll事件的回调函数和网页动画。

网页动画的每一帧（frame）都是一次重新渲染。

每秒低于24帧的动画，人眼就能感受到停顿。一般的网页动画，需要达到每秒30帧到60帧的频率，才能比较流畅。如果能够达到每秒70帧甚至80帧，就会极其流畅。

<img src="/images/img/fps.png" alt="" >

大多数显示器的刷新频率是60Hz，为了与系统一致，以及节省电力，浏览器会自动按照这个频率，刷新动画（如果可以做到的话）。

<img src="/images/img/speed.png" alt="" >

所以，如果网页动画能够做到每秒60桢，就会跟显示器同步刷新，达到最佳的视觉效果。这意味着，一秒之内进行60次重新渲染，每次重新渲染的时间不能超过16.66毫秒。

<img src="/images/img/rendering.png" alt="" >

一秒之间能够完成多少次重新渲染，这个指标就被称为“刷新率”，英文为FPS（frame per second）。60次重新渲染，就是60FPS。

## 开发者工具的performance面板

Chrome浏览器开发者工具的performance面板（Timeline），是查看“刷新率”的最佳工具。

使用Chrome开发者工具的performance（Timeline）功能可以帮助我们查看每个Javascript脚本的运行时间（包括子脚本），帮助我们发现并突破性能瓶颈。

### 打开 Chrome 调试面板【F12或者Command+Option+I】，进入 performance 界面。如下所示，按照步骤来

<img src="/images/img/timeLine.png" alt="" >

- 第二步：screenshots 是对你的屏幕进行截图，后面会生成相关的比较直观的截图
- 第四步：模拟 CPU 速度，更加方便你重现问题，如果 4x slowdown 不行，你可以选择 6x slowdown

我们可以看到左上侧的位置有几个重要按钮，作用如下：

<img src="/images/img/performance1.png" alt="" >

我们点击重新录制，就会出现：

<img src="/images/img/performance2.png" alt="" >

完成之后就会出现以下的界面，这里都是我们应该重点关注的内容。先来看看有哪些部分：

<img src="/images/img/performance4.png" alt="" >

- 第一部分：控制面板：controls，上面已经介绍
- 第二部分：概览面板：overView窗格，重要参数。我们可以看到FPS、CPU、NET在页面加载时候的变化。
  
  - FPS（frames per second）：每秒帧数，绿色竖线越高，FPS越高，我们应该关注红色部分，这说明我们的页面很可能出现了卡顿现象，FPS是用来分析动画的一个主要性能指标。另外60是一个比较理想的值。

    <img src="/images/img/performance7.jpeg" alt="" >

  - CPU：CPU资源。在CPU图表中的各种颜色与Summary面板里的颜色是相互对应的。CPU图表中的各种颜色代表着在这个时间段内，CPU在各种处理上所花的时间。如果你看到了某个处理占用了大量的时间，那么这很可能就是一个性能瓶颈的线索。

    <img src="/images/img/performance8.jpeg" alt="" >

  - NET：每条彩色横杆代表一种资源，横杆越长，检索资源所需要的事件越长。每个横杆的浅色部分表示等待时间（请求资源到第一个字节下载完成时间）。

    - HTML文件是蓝色。脚本是黄色。样式是紫色。媒体文件是绿色。其它资源是灰色。
  
  #### 把鼠标移动到FPS、CPU或者NET图表之上，就会展示这个时间点界面的截图。左右移动鼠标，可以重发当时的屏幕录像。这就是scrubbing，可以用来分析动画的各个细节。 

- 第三部分：线程面板：火焰图
1. Frames：帧线程，在Frames图表中，把鼠标移动到绿色条状图上，Devtools会展示这个帧的FPS。看有没有达到60的标准。

  <img src="/images/img/performance8.jpeg" alt="" >

2. Main：主线程，展示主线程的运行状况，负责执行Javascript，解析HTML/CSS，完成绘制。
  
  - 可以看到主线程调用栈和耗时情况，每个长条都是一个事件，悬浮可以看到耗时和事件名

    - x轴代表着加载的时间。长条越长就代表这个event花费的时间越长。
    - y轴代表了调用栈（call stack）。表示事件（线程）的执行顺序，在栈里，上面的event调用了下面的event。先是上面的执行再到下面的，我们要特别注意红色的三角形部分

    <img src="/images/img/performance4.jpeg" alt="" >

    不同的颜色表示不同的事件。

    <img src="/images/img/performance5.png" alt="" >

    哪种色块比较多，就说明性能耗费在那里。色块越长，问题越大。

    <img src="/images/img/performance6.png" alt="" >
    
3. Raster线程：负责完成某个layer或者某些块(tile)的绘制。
   
    <img src="/images/img/performance7.png" alt="" >
  

- 第四部分： 统计面板：
  
  统计面板选择因点击选择不同的目标统计的内容不同
  
  - Summary：统计图：展示各个事件阶段耗费的时间，可以看到CPU重的资源分配。因为提高性能就是一门做减法的艺术，如果你发现CPU花费了大量的时间在rendering上，那你的目标就是减少rendering的时间

    <img src="/images/img/performance5.jpeg" alt="" >

  - Bottom-Up：排序：可以看到各个事件消耗时间排序
    - (1)self-time 指除去子事件这个事件本身消耗的时间
    - (2)total-time 这个事件从开始到结束消耗的时间（包含子事件）

  - Call Tree：调用栈：Main选择一个事件，可以看到整个事件的调用栈（从最顶层到最底层，而不是只有当前事件）
    
    - 可以通过调用树来查看各个文件调用运行的时间及占比。

    <img src="/images/img/performance11.png" alt="" >

  - Event Log：事件日志
    - (1) 多了个start time，指事件在多少毫秒开始触发的
    - (2) 右边有事件描述信息


- 其他功能

  对于简单的页面，可以相当容易观察到性能问题。但是在现实使用场景下，就不是那么容易观察到了。所以可以使用这些工具来分析页面。


  - 开启实时监控
    1. 在控制台按下 Command+Shift+P（Mac）或者 Ctrol+Shift+P(Windows, Linux) 打开命令菜单
    2. 搜索Show Rendering
    3. 在Rendering面板里，激活FPS Meter。FPS实时面板就出现在页面的右上方。

    关闭FPS Meter只要按下Escape就可以了

    <img src="/images/img/performance10.jpeg" alt="" >

### 使用小提示： 第二、三、四部分是联动的，比如你选择了火焰图中的某个部分，下面的Summary就会显示这个部分的总结。

  <img src="/images/img/performance3.png" alt="" >


## 定位性能瓶颈

在性能报告中，有很多的数据。可以通过双击，拖动等等动作来放大缩小报告范围，从各种时间段来观察分析报告。

<img src="/images/img/performance3.jpeg" alt="" >

在事件长条的右上角出，如果出现了红色小三角，说明这个事件是存在问题的，需要特别注意。

双击这个带有红色小三角的的事件。在Summary面板会看到详细信息。注意reveal这个链接，双击它会让高亮触发这个事件的event。如果点击了ui.dll.js:45这个链接，Devtools会跳转到需要优化的代码处。

<img src="/images/img/performance12.png" alt="" >


<img src="/images/img/performance10.png" alt="" >

<img src="/images/img/performance11.jpeg" alt="" >



### 如果想达到60帧的刷新率，就意味着javascript线程中每个任务的耗时，必须少于16ms，一个解决办法就是使用Web Worker，主线程只用于UI渲染，然后跟UI渲染不相干的任务，都放在Worker线程。

## Web Workers

我们知道Javascript是单线程的，但是浏览器并不是单线程的。Javascript在浏览器的主线程运行，这正好可以与样式计算、布局等许多其他情况下的渲染操作一起运行，如果Javascript的运行时间过长，就会阻塞这些后续工作，导致帧丢失。


## 说明

  下一篇中会重点介绍Web Worker。