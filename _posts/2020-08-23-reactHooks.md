---
layout: post
categories: js
title: "React Hooks"
subtitle: "React 是主流的前端框架，v16.8 版本引入了全新的 API，叫做 React Hooks，颠覆了以前的用法。"
featured-image: /images/2016-11-19/abstract-4.jpg
tags: ['JS']
date-string: Aug 23, 2020
---

## 前言
这个 API 是 React 的未来，有必要深入理解。接下来我们一起来学习一下！

## 背景
React 的核心是组件。v16.8 版本之前，组件的标准写法是类（class）。

### 组件类的缺点
下面是一个简单的组件类。

```js
   import React, {Component} from "react";
   export default class Button extends Component {
      constructor() {
         super();
         this.state = {buttonText: 'click me'};
         this.handleClick = this.handleClick.bind(this);
      }
      handleClick() {
         this.setState(() => {
            return { buttonText: "Thanks!" };
         });
      }
      render() {
         const { buttonText } = this.state;
         return <button onClick={this.handleClick}>{buttonText}</button>;
      }
   }
```
这个组件类仅仅是一个按钮，但可以看到，它的代码已经很"重"了。真实的 React 项目由多个类按照层级，一层层构成，复杂度成倍增长。再加入 Redux，就变得更复杂。

Redux 的作者 Dan Abramov 总结了组件类的几个缺点。

- 大型组件很难拆分和重构，也很难测试。
- 业务逻辑分散在组件的各个方法之中，导致重复逻辑或关联逻辑。
- 组件类引入了复杂的编程模式，比如 render props 和高阶组件。

### 函数组件

React 团队希望，组件不要变成复杂的容器，最好只是数据流的管道。开发者根据需要，组合管道即可。 组件的最佳写法应该是函数，而不是类。

React 早就支持函数组件，但是，这种写法有重大限制，必须是纯函数，不能包含状态，也不支持生命周期方法，因此无法取代类。

**React Hooks 的设计目的，就是加强版函数组件，完全不使用"类"，就能写出一个全功能的组件。**

## Hook的含义
Hook是“钩子”的意思！

React Hooks 的意思是，组件尽量写成纯函数，如果需要外部功能和副作用，就用钩子把外部代码"钩"进来。React Hooks 就是那些钩子。

我们需要什么功能，就使用什么钩子。React 默认提供了一些常用钩子，另外我们也可以封装自己的钩子。

所有的钩子都是为函数引入外部功能，所以 React 约定，钩子一律使用use前缀命名，便于识别。我们要使用 xxx 功能，钩子就命名为 usexxx。

下面介绍 React 默认提供的四个最常用的钩子。

- useState()
- useContext()
- useReducer()
- useEffect()