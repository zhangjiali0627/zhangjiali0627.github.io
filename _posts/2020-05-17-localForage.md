---
layout: post
categories: 浏览器
title: 本地存储JS库localForage简介
subtitle: 一个东西是否有生命力，看的不是其是否强大，而是看其是否足够简单.
featured-image: /images/2020-05-17/first.jpg
tags: [本地存储]
date-string: MAY 17, 2020
---

localforage项目地址：https://github.com/localForage/localForage

JS文件下载：<a href="/js/localforage.min.js" class="btn btn-success">localforage.min.js</a>
<!-- # Heading 1 -->
## localForage简介

`localForage`是用来本地存储数据的。

从命名上来看，localForage和我们熟悉的localStorage很像，确实二者之间有很大的关系。

我们先来回顾一下localStorage的缺点：
1. 存储容量限制，大部分浏览器最多存储5M；
2. 仅支持字符串，如果要存储对象，还需要使用JSON.stringify和JSON.parse方法转换；
3. 读取是同步的。大多数情况下，挺方便的，不会有什么大影响。但，如果存储的数据量比较大，例如存储了一张图片的base64格式，再读就可能会有可感知的延迟时间。

localForage就是用来规避上面localStorage的缺点，同时保留localStorage的优点而设计的。

**localForage的逻辑**：优先使用IndexedDB存储数据，如果浏览器不支持，使用WebSQL，浏览器再不支持，使用localStorage。

**localForage的API**：与localStorage完全一样
1. getItem
2. setItem
3. removeItem
4. clear
5. length

## localForage和indexedDB的区别

`indexedDB`为本地存储数据库，它的功能非常强大，再复杂的结构存储都可以支持。而localForage只是使用了其功能的一部分，很多功能都受限，比如：localForage一次只能存储一个字段。

indexedDB存储空间几乎不受限制，性能也不错，支持各种数据结构的存储，但是indexedDB的上手成本比较大：

* 前端需要了解数据库的一些基本概念，例如表、游标、事务、锁等。而普遍的前端都是与页面打交道，数据库的操作偏后端。需要从零开始学数据库。

* indexedDB的API又多又长，容易记不住，学习成本比较高。

`localForage`的出现也算是曲线救国，通常我们的数据存储并不需要特别复杂，只要不是完完全全的离线开发，localForage就足够了。既不浪费indexedDB的好，又避开了indexedDB高上手成本的坑。从这个角度看，indexedDB应该要谢谢localForage。

当然，如果存储的数据是复杂的多行多列表结构，建议还是老老实实花点功夫学习学习indexDB的使用。

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
