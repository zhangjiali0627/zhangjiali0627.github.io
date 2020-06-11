---
layout: post
categories: js
title: "网页性能"
subtitle: "本文将介绍性能问题的出现原因及解决办法"
featured-image: /images/img/anchor.jpg
tags: ['前端交互']
date-string: Jun 5, 2020
---

#### 前言

你遇到过性能很差的网页吗？

这种网页响应非常缓慢，占用大量的CPU和内存，浏览起来特别卡顿，页面中的动画效果也不流畅。

你会有什么反应？

大多数用户应该都会关闭这个页面，作为开发者，我们肯定不愿意看到这种情况，那我们应该怎样提高性能呢？

## 一、网页生成的过程

要理解网页的性能为什么不好，就必须了解网页是怎么生成的。

网页的生成过程，可以分为5步：

1. HTML代码转化为DOM tree
2. CSS代码转化为CSS tree （CSSOM  CSS Object Model）
3. 合成DOM tree 和 CSS tree，生成一棵渲染树（包含每个节点的视觉信息）
4. 生成(layout)布局，也就是将所有渲染树的所有节点进行平面合成；
5. 将布局绘制在屏幕上（paint)

前三步是非常快的，第四步和第五步比较耗时；

<img src="/images/img/render.jpg" alt="" >



实现锚点滚动, 不要用<a>标签, 因为会触发路由跳转

- 可以使用H5提供的API scrollToAnchor

```js
  scrollToAnchor = (anchorName) => {
    if (anchorName) {
        let anchorElement = document.getElementById(anchorName);
        if(anchorElement) { anchorElement.scrollIntoView(); }
    }
  }
```

eg:

不用scrollToAnchor的情况

```html
  <div>
    <div>be together. not the same.</div>
    <div>welcome to android... be yourself. do your thing. see what's going on.</div>
    <div>
      <a href=""><img src="images/andy.png" /> create your android character</a>
    </div>

    <!-- 按钮 -->
    <a href="#screens">
      <button>
        <i>expand_more</i>
      </button>
    </a>
  </div>
  <div style={{height: 800}}>
    <!-- 需要跳转到的锚点 -->
    <a name="screens"></a>
    跳到这里
  </div>
```

如果在react中，遇到了<a>标签href="#xxx"的时候，会进行单页前端路由跳转的问题。

利用H5 提供的API——scrollToAnchorAPI来进行修改

### MND定义此方法让当前的元素滚动到浏览器窗口的可视区域内。

```js
  scrollToAnchor = (anchorName) => {
    if (anchorName) {
      // 找到锚点
      let anchorElement = document.getElementById(anchorName);
      // 如果对应id的锚点存在，就跳转到锚点
      if(anchorElement) { anchorElement.scrollIntoView({block: 'start', behavior: 'smooth'}); }
    }
  }
```
```html
  <div>
    <div>be together. not the same.</div>
    <div>welcome to android... be yourself. do your thing. see what's going on.</div>
    <div>
      <a href=""><img src="images/andy.png" /> create your android character</a>
    </div>

    <!-- 按钮 -->
    <a onClick={()=>this.scrollToAnchor('screens')}>
      <button>
        <i>expand_more</i>
      </button>
    </a>
  </div>
  <div style={{height: 800}}>
    <!-- 需要跳转到的锚点 -->
    <a id="screens"></a>
    跳到这里
  </div>
```

主要的修改:

1. 将锚点使用的传统的name属性，改成id属性。这样我们就可以用document.getElementById方法方便的查询查询到锚点。
2. 将原来的红色按钮的href属性去掉，然后添加一个onClick方法。onClick方法传入一个锚点的id，然后用下面的函数来找到锚点并跳转到锚点。


### scrollIntoView的语法

- 用法如下:

```js
element.scrollIntoView(); // 等同于element.scrollIntoView(true)
element.scrollIntoView(true);
element.scrollIntoView(alignToTop); // Boolean型参数
element.scrollIntoView(scrollIntoViewOptions); // Object型参数
```

alignToTop是一个布尔值，
- 如果为true，元素的顶端将和其所在滚动区的可视区域的顶端对齐。
- 如果为false，元素的底端将和其所在滚动区的可视区域的底端对齐.
- 默认是true

scrollIntoViewOptions是一个boolean或一个带有选项的object：
```js
  {
    behavior: 'auto' | 'instant' | 'smooth',
    block: 'start' | 'end'
  }
```

如果是一个boolean, true 相当于{block: "start"}，false 相当于{block: "end"}

eg:
```js
  let element = document.getElementById("box");
  element.scrollIntoView();
  element.scrollIntoView(false);
  element.scrollIntoView({block: "end"});
  element.scrollIntoView({block: "end", behavior: "smooth"});
```

smooth: 带滚动动画

### 区分scrollIntoViewIfNeeded

- MND的定义此方法用来将不在浏览器窗口的可见区域内的元素滚动到浏览器窗口的可见区域。
- 如果该元素已经在浏览器窗口的可见区域内，则不会发生滚动。 此方法是标准的Element.scrollIntoView()方法的专有变体。

语法:
```js
  element.scrollIntoViewIfNeeded(); // 等同于element.scrollIntoViewIfNeeded(true)
  element.scrollIntoViewIfNeeded(true);
  element.scrollIntoViewIfNeeded(false);
```

参数:
- opt_center一个 Boolean 类型的值，默认为true:
  - 如果为true，则元素将在其所在滚动区的可视区域中居中对齐。
  - 如果为false，则元素将与其所在滚动区的可视区域最近的边缘对齐。
  - 根据可见区域最靠近元素的哪个边缘，元素的顶部将与可见区域的顶部边缘对准，或者元素的底部边缘将与可见区域的底部边缘对准。

eg:
```js
let element = document.getElementById("child");

element.scrollIntoViewIfNeeded();
element.scrollIntoViewIfNeeded(true);
element.scrollIntoViewIfNeeded(false);
```

## 监听滚动条

- 需要在componentDidMount中，监听scroll事件，绑定一个函数，让这个函数进行监听处理
  - window.addEventListener('scroll', this.scroll);

- 在componentWillUnmount，进行scroll事件的注销
  - window.removeEventListener('scroll', this.scroll);

- 在监听函数中我们可以监听到scrollTop值以及scrollLeft值，并且我们可以结合获取到的值来做相应的判断；

  -  let clientHeight = document.documentElement.clientHeight; //可视区域高度
  - let scrollTop  = document.documentElement.scrollTop;  //滚动条滚动高度
  - let scrollHeight =document.documentElement.scrollHeight; //滚动内容高度


上代码：

```js
  // 在componentDidMount，进行scroll事件的注册，绑定一个函数，让这个函数进行监听处理
  componentDidMount() {
    window.addEventListener('scroll', this.bindHandleScroll);
  }
  bindHandleScroll=(event)=>{
    // 滚动的高度
    const scrollTop = (event.srcElement ? event.srcElement.documentElement.scrollTop : false)
                  || window.pageYOffset
                  || (event.srcElement ? event.srcElement.body.scrollTop : 0);
    // 判断用户当前是否进行了横向滚动，如果用户发生了横向滚动，则设置元素为static
    const scrollLeft = (event.srcElement ? event.srcElement.documentElement.scrollLeft : false)
                  || window.pageXOffset
                  || (event.srcElement ? event.srcElement.body.scrollLeft : 0);

    // 如果页面滚动超过450px的时候，可以改变某些元素的样式
    if(scrollTop > 450){
      this.setState({
        positionType:'static'
      })
    }else{
      this.setState({
        positionType:'fixed'
      })
    }
  }
  // 在componentWillUnmount，进行scroll事件的注销
  componentWillUnmount(){
    window.removeEventListener('scroll', this.bindHandleScroll);
  }
```

## 总结
    以上就是在react中实现锚链接的方式以及监听滚动条的方法,当然也有实现类似效果的其他方法,后续会逐步补充!