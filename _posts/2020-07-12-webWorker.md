---
layout: post
categories: js
title: "web Worker（下）"
subtitle: "web worker 是运行在后台的 JavaScript，不会影响页面的性能。"
featured-image: /images/img/performance.jpeg
tags: ['性能优化']
date-string: Jun 23, 2020
---

## 数据通信

主线程与Worker之间的通信内容，可以是文本，也可以是对象。但需要注意的是，这种通信是拷贝关系，也就是说传值不传址，Web Worker对通信内容的修改不会影响到主线程。

浏览器内部的运行机制是，先将通信内容串行化，然后把串行化后的字符串发给Worker，Web Worker再将它还原。

主线程与Worker之间也可以交换二进制数据，比如：File、Blob、ArrayBuffer等类型，也可以在线程之间发送。For example：

```js
  // 主线程 
  let uInt8Array = new Uint8Array(new ArrayBuffer(10));
  for (let i = 0; i < uInt8Array.length; i++) {
    uInt8Array[i] = i * 2; // [0,2,4,6,8,...] 
  }
  worker.postMessage(uInt8Array);

  // Worker 线程
  self.onmessage = (e) => {
    let uInt8Array = e.data;
    postMessage('Inside worker.js: uInt8Array.toString() = ' + uInt8Array.toString());
    postMessage('Inside worker.js: uInt8Array.byteLength = ' + uInt8Array.byteLength);
  }
```
但是，拷贝方式发送二进制数据，会造成性能问题。比如：主线程向Worker发送一个500MB文件，默认情况下浏览器会生成一个原文件的拷贝。为了解决这个问题，Javascript允许主线程把二进制数据直接转移给子线程，但是一旦转移，主线程就无法再使用这些二进制数据了，这是为了防止出现多个线程同时修改数据的麻烦局面。这种转移数据的方法，叫做Transferable Objects。这就使主线程可以快速把数据交给Worker，对于影像处理、声音处理、3D运算等就非常方便了，不会产生性能负担。

如果要直接转移数据，可以使用以下写法：
```js
  // Transferable Objects 格式
  worker.postMessage(arrayBuffer, [arrayBuffer]);

  // eg:
  let ab = new ArrayBuffer(1);
  worker.postMessage(ab, [ab]);
```

## 同页面的 Web Worker

通常情况下，Worker载入的是一个单独的Javascript脚本文件，但是也可以载入与主线程在同一个网页的代码。

```html
  <!DOCTYPE html>
    <body>
      <script id="worker" type="app/worker">
        addEventListener('message', () => {
          postMessage('some message');
        }, false)
      </script>
    </body>
  </html>
```

上面是一段嵌入网页的脚本，注意必须指定 ***script*** 标签的type属性是一个浏览器认识的值，上例是app/worker。
然后，读取这一段嵌入页面的脚本，用Worker来处理。

```js
  let blob = new Blob([document.querySelector('#worker').textContent]);
  let url = window.URL.createObjectURL(blob);
  let worker = new Worker(url);

  worker.onmessage = (e) => {
    // e.data === 'some message'
  }
```
上面的代码中，先将嵌入网页的脚本代码转成一个二进制对象，然后为这个二进制对象生成URL，再让Worker加载这个URL。这样就做到了，主线程和Worker的代码都在同一个网页上面。

## 实例：Worker线程完成轮询
有时，浏览器需要轮询服务器状态以便第一时间得知状态改变。这个工作可以放在Worker里面。For example：

```js
  createWorker (f) => {
    let blob = new Blob(['(' + f.toString() + ')()']);
    let url = window.URL.createObjectURL(blob);
    let worker = new Worker(url);
    return worker;
  }
  let pollingWorker = createWorker((e) => {
    let cache;
    compare (new, old) {
      ...
    };
    setInterval(() => {
      fetch('/my-api-endpoint').then((res) => {
        let data = res.json();

        if(!compare(data, cache)) {
          cache = data;
          self.postMessage(data);
        }
      })
    }, 1000)
  });
  pollingWorker.onmessage = () => {
    // render data
  }
  pollingWorker.postMessage('init);
```
上面代码中，Worker 每秒钟轮询一次数据，然后跟缓存做比较。如果不一致，就说明服务端有了新的变化，因此就要通知主线程。

### Worker 线程内部还能再新建Worker线程（目前只有Firefox浏览器支持）。

## API

#### 主线程
浏览器原生提供Worker()构造函数，用来供主线程生成Worker线程。

```js
  let myWorker = new Worker(jsUrl, options);
```
Worker()构造函数可以接受两个参数。
- 第一个参数是脚本的网址（必须遵守同源政策），该参数是必需的，且只能加载JS脚本，否则会报错。
- 第二个参数是配置对象，该对象是可选的。它的一个作用就是指定Worker的名称，用来区分多个Worker线程。

```js
  // 主线程
  let myWorker = new Worker('worker.js', { name : 'myWorker' });

  // Worker 线程
  self.name // myWorker
```
**Worker()** 构造函数返回一个 Worker 线程对象，用来供主线程操作 Worker。Worker 线程对象的属性和方法如下:

- Worker.onerror: 指定error事件的监听函数。
- Worker.onmessage: 指定message 事件的监听函数，发送过来的数据在Event.data属性中。
- Worker.onmessageerror: 指定messageerror事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
- Worker.postMessage(): 向Worker线程发送消息。
- Worker.terminate(): 立即终止Worker线程。

### Worker 线程
Web Worker有自己的全局对象，不是主线程的window，而是一个专门为Worker定制的全局对象。因此定义在Window上的对象和方法不是全部都可以使用。

Worker线程有一些自己的全局属性和方法：
- self.name: Worker的名字，该属性只读，有构造函数指定。
- self.onmessage: 指定message事件的监听函数。
- self.onmessageerror: 指定messageerror事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
- self.close(): 关闭Worker线程。
- self.postMessage(): 向产生这个Worker线程发送消息。
- self.importScripts(): 加载JS脚本。