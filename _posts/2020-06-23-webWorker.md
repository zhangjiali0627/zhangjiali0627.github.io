---
layout: post
categories: js
title: "web Worker（上）"
subtitle: "web worker 是运行在后台的 JavaScript，不会影响页面的性能。"
featured-image: /images/img/performance.jpeg
tags: ['性能优化']
date-string: Jun 23, 2020
---

JS采用的是单线程模型，所有任务只能在一个线程上完成，一次只能做一件事。前面的任务没有做完，后面的任务只能等着。单线程带来了很大的不便，无法充分发挥计算机的计算能力。

Web Worker的作用，就是为JS创造多线程环境，允许主线程创建Worker线程，将一些任务分配给后者运行。在主线程运行的同时，Worker线程在后台运行，两者互不干扰。等到Worker线程完成计算任务，再把结果返回给主线程。

#### 好处：一些计算密集型或高延迟的任务，被Worker线程承担了，主线程（主要负责UI交互）就会很流畅，不会被阻塞或拖慢。

Worker线程一旦新建成功，就会始终运行，不会被主线程上的活动（比如用户点击按钮、提交表单）打断。这样有利于随时响应主线程的通信。但是，这也造成了Worker比较耗费资源，不应该过度使用，而且一旦使用完毕，就应该关闭。

## 使用Web Worker时应该注意以下几点：

1. 同源限制
分配给Worker线程运行的脚本文件，必须与主线程脚本文件同源。

2. DOM限制
Worker线程所在的全局对象、与主线程不一样，无法读取主线程所在网页的DOM对象，也无法使用document、window、parent这些对象。但是，Worker线程可以使用navigator对象和location对象。

3. 通信联系
Worker线程和主线程不在同一个上下文环境，它们不能直接通信，必须通过消息完成。

4. 脚本限制
Worker线程不能执行alert()方法和confirm()方法，但Web Worker中的Javascript依然可以使用setTimeout(),setInterval()之类的函数，也可以使用XMLHttpRequest对象发出AJAX请求。

5. 文件限制
Worker 线程无法读取本地文件，即不能打开本机的文件系统（file://），它所加载的脚本，必须来自网络。

## 浏览器支持

所有主流浏览器均支持 web worker，除了 Internet Explorer。

## 检测 Web Worker 支持
在创建 web worker 之前，请检测用户的浏览器是否支持它：

  ```js
    if(typeof(Worker)!=="undefined"){
      // Yes! Web worker support!
      // Some code.....
    } else {
      // Sorry! No Web Worker support..
    }
  ```
## Web Worker的基本用法

### 主线程

1. 主线程采用new命令，调用Worker()构造函数，新建一个Worker线程。

  ```js
    let worker = new Worker('work.js'); 
  ```

  Worker()构造函数的参数是一个JS脚本文件，该文件就是Worker线程所要执行的任务。由于Worker不能读取本地文件，所以这个脚本必须来自网络。如果下载没有成功（如404错误），Worker就会默默失败。

2. 然后，主线程调用 worker.postMessage()方法，向Worker发消息。

  ```js
    worker.postMessage('Hello World');
    worker.postMessage({method: 'echo', args: ['Work']});
  ```

  worker.postMessage()方法的参数，就是主线程传给Worker的数据。它可以是各种数据类型，包括二进制数据。

3. 接着，主线程通过worker.onmessage指定监听函数，接收子线程发回来的消息。

  ```js
    worker.onmessage = (event) => {
      console.log('Received message ' + event.data);
      doSomething();
    }

    doSomething () => {
      // 执行任务
      worker.postMessage('Work done!');
    }
  ```

  上面代码中，事件对象的data属性可以获取Worker发来的数据。

4. Worker完成任务之后，主线程就可以把它关掉。
  ```js
    worker.terminate();
  ```

## Worker 线程

Worker线程内部需要有一个监听函数，监听message事件。
  ```js
    self.addEventListener('message', (e) => {
      self.postMessage('You said: ' + e.data);
    }, false);
  ```
  上面代码中，self代表子线程自身，即子线程的全局对象。等同于下面的两种写法。
  ```js
    // 写法一
    this.addEventListener('message', (e) => {
      this.postMessage('You said: ' + e.data);
    }, false);
    // 写法二
    addEventListener('message', (e) => {
      postMessage('You said: ' + e.data);
    }, false);
  ```
  除了使用self.addEventListener()指定监听函数，也可以使用self.onmessage指定。
  监听函数的参数是一个事件对象，它的data属性包含主线程发来的数据。
  self.postMessage()方法用来向主线程发送消息。  
  ```js
    self.addEventListener('message', (e) => {
      let data = e.data;
      switch (data.cmd) {
        case 'start':
          self.postMessage('WORKER STARTED: ' + data.msg);
          break;
        case 'stop':
          self.postMessage('WORKER STOPPED: ' + data.msg);
          self.close(); // Terminates the worker.
          break;
        default:
          self.postMessage('Unknown command: ' + data.msg);
      };
    }, false);
  ```
  代码中的self.close()用于在Worker内部关闭自身。

## Worker加载脚本

Worker内部如果要加载其他脚本，有一个专门的方法importScripts()。
  ```js
  importScripts('script1.js');
  // 该方法可以同时加载多个脚本。
  importScripts('script1.js', 'script2.js');
  ```

## 错误处理
主线程可以监听Worker是否发生错误。如果发生错误，Worker会触发主线程的error事件。
  ```js
    worker.onerror((event)=>{
      console.log([
        'ERROR: Line ', e.lineno, ' in ', e.filename, ': ', e.message
      ].join(''));
    })
    // 或者
    worker.addEventListener('error', (event) => {
      // ...
    })
  ```
  Worker 内部也可以监听error事件。

## 关闭Worker
使用完毕，为了节省系统资源，必须关闭Worker。
  ```js
  // 主线程
    worker.terminate();
    // worker 线程
    self.close();
  ```

## 说明

  以上是worker的一些用法，下一篇中会重点介绍Web Worker的应用。