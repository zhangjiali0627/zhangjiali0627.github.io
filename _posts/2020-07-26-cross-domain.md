---
layout: post
categories: js
title: "跨域"
subtitle: "前端开发的过程中，经常会遇到跨域问题，本文将对跨域做一个简单的总结。"
featured-image: /images/2016-11-19/abstract-2.jpg
tags: ['JS']
date-string: Jul 26, 2020
---

## 什么是跨域

跨域，是指浏览器不能执行其他网站的脚本。跨域是由浏览器的通源策略造成的，是浏览器对Javascript实施的安全限制。

同源策略限制了以下行为：
- Cookie、LocalStorage和IndexDB无法读取
- DOM和JS对象无法获取
- Ajax请求发送不出去

## 常见的跨域场景

所谓的同源是指域名、协议、端口均相同。

```js
  http://www.baidu.com/a.html 调用 http://www.baidu.com/server.php 非跨域

  http://www.baidu.com/a.html 调用 http://www.baidu1.com/server.php 跨域，主域不同
  
  http://abc.baidu.com/a.html 调用 http://def.baidu1.com/server.php 跨域，子域名不同

  http://www.baidu.com:8000/a.html 调用 http://www.baidu.com/server.php 跨域，端口不同

  https://www.baidu.com/a.html 调用 http://www.baidu.com/server.php 跨域，协议不同

  localhost 调用 127.0.0.1 跨域
```

## 跨域的解决办法

### jsonp 跨域

jsonp 跨域其实也是Javascript设计模式中的一种代理模式。在html页面中通过相应的标签从不同域名下加载静态资源文件是被浏览器允许的，所以我们可以通过这个“漏洞”来进行跨域。一般，我们可以动态的创建script标签，再去请求一个带参网址来实现跨域通信。

```js
  // 原生的实现方式
  let script = document.createElement('script');
  script.src = 'http://www.baidu.com/login?username=Zhangsan&callback=callback';

  document.body.appendChild(script);

  let callback = (res) => {
    console.log(res);
  }
```

当然，jquery也支持jsonp的实现方式。

```js
  $.ajax({
    url:'http://www.baidu.com/login',
    type:'GET',
    dataType:'jsonp',//请求方式为jsonp
    jsonpCallback:'callback',
    data:{
        "username":"Zhangsan"
    },
  })
```
#### 虽然这种方式非常好用，兼容性也非常好，但是一个最大的缺陷是，只能够实现get请求。

### postMessage 跨域

这是由H5提出来的一个API，IE8+，chrome，FF都支持实现这个功能。这个功能非常的简单，其中包括接受信息的message时间，和发送信息的postMessage方法。

发送信息的postMessage方法是向外界窗口发送信息
```
  otheWindow.postMessage(message,targetOrigin);
```
- otherWindow 指的是目标窗口，也就是要给哪一个window发送消息，是window.frames属性的成员或者是window.open方法创建的窗口。
- message是要发送的消息，类型为String，Object（IE8、9不支持Obj），targetOrigin是限定消息接受范围，不限制就用星号*

接受信息的message事件

```js
  let onmessage = () => {
    let data = event.data;
    let origin =event.origin;
  }
  if(typeof window.addEventListener != 'undefined') {
    window.addEventListener('message', onmessage, false);
  }else if(typeof window.attachEvent != 'undefiner') {
    window.attachEvent('onmessage', onmessage);
  }
```

eg:
a.html(www.baidu.com/a/html)

```html
  <iframe id="iframe" src="http://www.bai.com/b.html" style="display:none;"></iframe>
  <script>       
      var iframe = document.getElementById('iframe');
      iframe.onload = function() {
          var data = {
              name: 'aym'
          };
          // 向bai传送跨域数据
          iframe.contentWindow.postMessage(JSON.stringify(data), 'http://www.bai.com');
      };

      // 接受domain2返回数据
      window.addEventListener('message', function(e) {
          alert('data from bai ---> ' + e.data);
      }, false);
  </script>
```

b.html(www.bai.com/b.html)

```html
  <script>
      // 接收domain1的数据
      window.addEventListener('message', function(e) {
          alert('data from baidu ---> ' + e.data);

          var data = JSON.parse(e.data);
          if (data) {
              data.number = 16;

              // 处理后再发回baidu
              window.parent.postMessage(JSON.stringify(data), 'http://www.baidu.com');
          }
      }, false);
  </script>
```


### document.domain + iframe 跨域

这种跨域的方式最主要的是要求主域名相同。

#### 什么是主域名相同？
www.baidu.com

aaa.baidu.com

ba.ad.baidu.com

以上这三个主域名都是baidu.com,而主域名不同的就不能用此方法。

假设目前a.baidu.com和b.baidu.com分别对应指向不同ip的服务器。

a.baidu.com下有一个test.html文件

```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>html</title>
        <script type="text/javascript" src = "jquery-1.12.1.js"></script>
    </head>
    <body>
        <div>A页面</div>
        <iframe 
        style = "display : none" 
        name = "iframe1" 
        id = "iframe" 
        src="http://b.baidu.com/1.html" frameborder="0"></iframe>
        <script type="text/javascript">
            $(function(){
                try{
                    document.domain = "baidu.com"
                }catch(e){}
                $("#iframe").load(function(){
                    var jq = document.getElementById('iframe').contentWindow.$
                    jq.get("http://baidu.com/test.json",function(data){
                        console.log(data);
                    });
                })
            })
        </script>
    </body>
  </html>
```

利用iframe加载了其他域下的文件（baidu.com/1.html），同时document.domain 设置成了 baidu.com，当iframe加载完毕后就可以获取baidu.com域下的全局对象，此时尝试着去请求baidu.com域名下的test.json(请求接口），就会发现数据请求失败了。

数据请求失败，是因为还缺少一步：

```html
  <!DOCTYPE html>
  <html lang="en">
    <head>
        <meta charset="UTF-8">
        <title>html</title>
        <script type="text/javascript" src = "jquery-1.12.1.js"></script>
        <script type="text/javascript">
            $(function(){
                try{
                    document.domain = "baidu.com"
                }catch(e){}
            })
        </script>
    </head>
    <body>
        <div id = "div1">B页面</div>
    </body>
  </html>
```
此时刷新浏览器，就会发现数据请求成功了~~~~~

### window.name + iframe 跨域

window.name属性可设置或者返回存放窗口名称的一个字符串。神奇之处在于name值在不同页面或者不同域下加载后依旧存在，没有修改就不会发生变化，并且可以存储非常长的name（2MB）

假设index页面请求远端服务器上的数据，我们在该页面下创建iframe标签，该iframe的src指向服务器文件的地址（iframe标签src可以跨域），服务器文件里设置好window.name的值，然后再在index.html里面读取改iframe中的window.name的值。

```html
  <body>
    <script type="text/javascript"> 
      iframe = document.createElement('iframe'),
      iframe.src = 'http://localhost:8080/data.php';
      document.body.appendChild(iframe);
      iframe.onload = function() {
        console.log(iframe.contentWindow.name)
      };
    </script>
  </body>
```
当然，这样还是不够的。

因为规定index.html页面和和该页面里的iframe框架的src如果不同源，则也无法操作框架里的任何东西，所以就取不到iframe框架的name值了。
既然要同源，那就换个src去指，前面说了无论怎样加载window.name值都不会变化，于是我们在index.html相同目录下，新建了个proxy.html的空页面，修改代码如下：

```html
  <body>
    <script type="text/javascript"> 
      iframe = document.createElement('iframe'),
      iframe.src = 'http://localhost:8080/data.php';
      document.body.appendChild(iframe);
      iframe.onload = function() {
        iframe.src = 'http://localhost:81/cross-domain/proxy.html';
        console.log(iframe.contentWindow.name)
      };
    </script>
  </body>
```

理想似乎很美好，在iframe载入过程中，迅速重置iframe.src的指向，使之与index.html同源，那么index页面就能去获取它的name值了！但是现实是残酷的，iframe在现实中的表现是一直不停地刷新，也很好理解，每次触发onload时间后，重置src，相当于重新载入页面，又触发onload事件，于是就不停地刷新了（但是需要的数据还是能输出的）。修改后代码如下：

```html
  <body>
    <script type="text/javascript"> 
      iframe = document.createElement('iframe');
      iframe.style.display = 'none';
      var state = 0;
      
      iframe.onload = function() {
        if(state === 1) {
            var data = JSON.parse(iframe.contentWindow.name);
            console.log(data);
            iframe.contentWindow.document.write('');
            iframe.contentWindow.close();
          document.body.removeChild(iframe);
        } else if(state === 0) {
            state = 1;
            iframe.contentWindow.location = 'http://localhost:81/cross-domain/proxy.html';
        }
      };

      iframe.src = 'http://localhost:8080/data.php';
      document.body.appendChild(iframe);
    </script>
  </body>
```
所以如上，我们就拿到了服务器返回的数据，但是有几个条件是必不可少的：
- iframe标签的跨域能力
- window.names属性值在文档刷新后依然存在的能力
