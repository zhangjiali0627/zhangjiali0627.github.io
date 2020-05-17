---
layout: post
categories: 浏览器
title: 本地存储JS库localForage简介
subtitle: 一个东西是否有生命力，看的不是其是否强大，而是看其是否足够简单.
featured-image: /images/2020-05-17/first.jpg
tags: [本地存储]
date-string: MAY 17, 2020
---
localStorage 能够帮助我们实现基本的数据存储，但它的速度慢，而且不能处理二进制数据。IndexedDB 是异步的，速度快，支持大数据集，但他们的API 使用起来有点复杂。

Mozilla 开发了一个叫 localForage 的库 ，使得离线数据存储在任何浏览器都是一项容易的任务。

localforage项目地址：https://github.com/localForage/localForage

JS文件下载：<a href="/js/localforage.min.js" class="btn btn-success">localforage.min.js</a>
<!-- # Heading 1 -->
## localForage简介

`localForage`是用来本地存储数据的。

从命名上来看，localForage和我们熟悉的localStorage很像，确实二者之间有很大的关系。

我们先来回顾一下localStorage的缺点：
1. 存储容量限制，大部分浏览器最多存储5M；
2. 仅支持字符串，如果要存储对象，还需要使用JSON.stringify和JSON.parse方法转换；
3. 读取是同步的。大多数情况下，挺方便的，不会有什么大影响。但，如果存储的数据量比较大，例如存储了一张图片的base64格式，再读就可能会有可感知的延迟时间。不管数据多大，我们都需要等待数据从磁盘读取和解析，这会减慢我们的应用程序的响应速度。这在移动设备上是特别糟糕的，主线程被挂起，直到数据被取出，会使你的应用程序看起来慢，甚至没有反应。

localForage就是用来规避上面localStorage的缺点，同时保留localStorage的优点而设计的。

**localForage的API**：与localStorage完全一样
1. getItem
2. setItem
3. removeItem
4. clear
5. length

**localForage的特点**
1. 支持回调的异步API；
2. 支持 IndexedDB、WebSQL 和 localStorage 三种存储模式（自动为我们加载最佳的驱动程序）
3. 支持 BLOB 和任意类型的数据，可以存储图片，文件等等。
4. 支持 ES6 Promises；

**localForage的逻辑**：优先使用IndexedDB存储数据，如果浏览器不支持，使用WebSQL，浏览器再不支持，使用localStorage。

对 IndexedDB 和 WebSQL 的支持使Web 应用程序可以存储更多的数据，要比 localStorage 允许存储的多很多。其 API 的无阻塞性质使得应用程序更快，不会因为 Get/Set 调用而挂起主线程。

## localForage 安装

### 通过 npm 安装：
* npm install localforage

### 或通过 bower：
* bower install localforage

### 引入js文件：
<script src="localforage.js"></script>
<script>console.log('localforage is: ', localforage);</script>


## localForage和indexedDB的区别

* IndexedDB的代码
```
  let db;
  let dbName = "dataspace";
  let users = [ {id: 1, fullName: 'Matt'}, {id: 2, fullName: 'Bob'} ];
  let request = indexedDB.open(dbName, 2);
  request.onerror = (event) => {
    // 错误处理
  };
  request.onupgradeneeded = (event) => {
    db = event.target.result;
    let objectStore = db.createObjectStore("users", { keyPath: "id" });
    objectStore.createIndex("fullName", "fullName", { unique: false });
    objectStore.transaction.oncomplete = (event) => {
      let userObjectStore = db.transaction("users", "readwrite").objectStore("users");
    }
  };

  let transaction = db.transaction(["users"], "readwrite");
  // 所有数据都添加到数据后调用
  transaction.oncomplete = (event) => {
    console.log("All done!");
  };
  transaction.onerror = (event) => {
    // 错误处理
  };

  let objectStore = transaction.objectStore("users");
  for (let i in users) {
    let request = objectStore.add(users[i]);
    request.onsuccess = (event) => {
      // 里面包含我们需要的用户信息
      console.log(event.target.result);
    };
  }
```

* localForage 代码：

```
// 保存用户信息
  let users = [ {id: 1, fullName: 'Matt'}, {id: 2, fullName: 'Bob'} ];
  localForage.setItem('users', users, (result) => {
    console.log(result);
  });
```
从代码量来看就简单很多~

`indexedDB`为本地存储数据库，它的功能非常强大，再复杂的结构存储都可以支持。而localForage只是使用了其功能的一部分，很多功能都受限，比如：localForage一次只能存储一个字段。

indexedDB存储空间几乎不受限制，性能也不错，支持各种数据结构的存储，但是indexedDB的上手成本比较大：

* 前端需要了解数据库的一些基本概念，例如表、游标、事务、锁等。而普遍的前端都是与页面打交道，数据库的操作偏后端。需要从零开始学数据库。

* indexedDB的API又多又长，容易记不住，学习成本比较高。

`localForage`的出现也算是曲线救国，通常我们的数据存储并不需要特别复杂，只要不是完完全全的离线开发，localForage就足够了。既不浪费indexedDB的好，又避开了indexedDB高上手成本的坑。从这个角度看，indexedDB应该要谢谢localForage。

当然，如果存储的数据是复杂的多行多列表结构，建议还是老老实实花点功夫学习学习indexDB的使用。

## 支持非字符串数据

比方说，你要下载一个用户的个人资料图片，并对其进行缓存以供离线使用。使用 localForage 很容易保存二进制数据：

```
  // 使用 AJAX 下载图片
  let request = new XMLHttpRequest();

  // 以获取第一个用户的资料图片为例
  request.open('GET', "/users/1/profile_picture.jpg", true);
  request.responseType = 'arraybuffer';

  // 当 AJAX 调用完成，把图片保存到本地
  request.addEventListener('readystatechange', () => {
    if (request.readyState === 4) { // readyState DONE
      // 保存的是二进制数据，如果用 localStorage 就无法实现
      localForage.setItem('user_1_photo', request.response, () => {
        // 图片已保存，想怎么用都可以
      });
    }
  });

  request.send()
```

下次，只用三行代码就可以从缓存中把照片读取出来

```
  localForage.getItem('user_1_photo', (photo) => {
    // 获取到图片数据后，可以通过创建 data URI 或者其它方法来显示
    console.log(photo);
  });
```

## callbacks && Promises

如果你不喜欢在你的代码中使用回调，你可以使用 ES6 Promises，来替换 localForage 的回调参数。让我们使用上面的照片例子，看下使用 Promises 的代码：

```
  localForage.getItem('user_1_photo').then((photo) => {
    // 获取到图片数据后，可以通过创建 data URI 或者其它方法来显示
    console.log(photo);
  });
```

## 浏览器兼容性
`indexDB`IE10+浏览器支持。因此，如果想要使用localForage存储任意格式数据，需要注意下浏览器的兼容性问题，例如，本地图片存储Blob数据，IE9肯定是不支持的。这些浏览器怕是只能存字符串了。

<!-- ### Heading 3

#### Heading 4

##### Heading 5

###### Heading 6

### Body text -->

<!-- ![Smithsonian Image](/images/3953273590_704e3899d5_m.jpg) -->
<!-- {: .image-right} -->

<!-- *This is emphasized*. Donec faucibus. Nunc iaculis suscipit dui. 53 = 125. Water is H<sub>2</sub>O. Nam sit amet sem. Aliquam libero nisi, imperdiet at, tincidunt nec, gravida vehicula, nisl. The New York Times <cite>(That’s a citation)</cite>. <u>Underline</u>. Maecenas ornare tortor. Donec sed tellus eget sapien fringilla nonummy. Mauris a ante. Suspendisse quam sem, consequat at, commodo vitae, feugiat in, nunc. Morbi imperdiet augue quis tellus. -->

<!-- HTML and <abbr title="cascading stylesheets">CSS<abbr> are our tools. Mauris a ante. Suspendisse quam sem, consequat at, commodo vitae, feugiat in, nunc. Morbi imperdiet augue quis tellus. Praesent mattis, massa quis luctus fermentum, turpis mi volutpat justo, eu volutpat enim diam eget metus.

### Blockquotes

> Lorem ipsum dolor sit amet, test link adipiscing elit. Nullam dignissim convallis est. Quisque aliquam.

## List Types

### Ordered Lists

1. Item one
   1. sub item one
   2. sub item two
   3. sub item three
2. Item two

## Tables

| Header1 | Header2 | Header3 |
|:--------|:-------:|--------:|
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
|----
| cell1   | cell2   | cell3   |
| cell4   | cell5   | cell6   |
|=====
| Foot1   | Foot2   | Foot3
{: rules="groups"}

## Code Snippets

Syntax highlighting via Rouge

```css
#container {
  float: left;
  margin: 0 -240px 0 0;
  width: 100%;
}
```

Non Pygments code example

    <div id="awesome">
        <p>This is great isn't it?</p>
    </div>

## Buttons

Make any link standout more when applying the `.btn` class.

```html
<a href="#" class="btn btn-success">Success Button</a>
```

<div markdown="0"><a href="#" class="btn">Primary Button</a></div>
<div markdown="0"><a href="#" class="btn btn-success">Success Button</a></div>
<div markdown="0"><a href="#" class="btn btn-warning">Warning Button</a></div>
<div markdown="0"><a href="#" class="btn btn-danger">Danger Button</a></div>
<div markdown="0"><a href="#" class="btn btn-info">Info Button</a></div> -->
