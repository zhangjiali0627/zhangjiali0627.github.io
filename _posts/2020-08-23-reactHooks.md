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

## useState():状态钩子
useState()用于为函数组件引入状态（state）。纯函数不能有状态，所以把状态放在钩子里面。

上面提到的组件类，用户点击按钮，会导致按钮的文字改变，文字取决于用户是否点击，这就是状态。使用useState()重写如下。

```js
   import React, {useState} from 'react';
   export default function Button() {
      const [buttonText, setButtonText] = useState('click me');

      function handleClick() {
         return setButtonText('Thanks!');
      }
      return <button onClick={handleClick}>{buttonText}</button>;
   }
```

在上面的代码中，Button组件是一个函数，内部使用useState()钩子引入状态。

useSTate()这个函数接受状态的初始值，作为参数，上里的初始值为按钮的文字。

该函数返回一个数组，数组的第一个成员是一个变量（上例是buttonText）,指向状态的当前值。第二个成员是一个函数，用来更新状态，约定是set前缀加上状态的变量名（上例是setButtonText）。

## useContext():共享状态钩子
如果需要在组件之间共享状态，可以使用useContext();

现在有两个组件Navbar和Messages，我们希望它们之间共享状态。

```js
   <div className="App">
      <Navbar/>
      <Messages/>
   </div>
```
第一步就是使用 React Context API，在组件外部建立一个 Context。

```js
   const AppContext = React.createContext({});
```
组件封装代码如下。

```js
   <AppContext.Provider value={{
      username: 'superawesome'
   }}>
      <div className="App">
         <Navbar/>
         <Messages/>
      </div>
   </AppContext.Provider>
```

上面的代码中，AppContext.Provider提供了一个 Context 对象，这个对象可以被子组件共享。

Navbar 组件的代码如下。
```js
   const Navbar = () => {
      const { username } = useContext(AppContext);
      return (
         <div className="navbar">
            <p>AwesomeSite</p>
            <p>{username}</p>
         </div>
      );
   }
```

上面代码中，useContext()钩子函数用来引入 Context 对象，从中获取username属性。

Message 组件的代码也类似。
```js
   const Messages = () => {
      const { username } = useContext(AppContext)

      return (
         <div className="messages">
            <h1>Messages</h1>
            <p>1 message for {username}</p>
            <p className="message">useContext is awesome!</p>
         </div>
      )
   }
```

## useReducer(): action 钩子
React 本身不提供状态管理功能，通常需要使用外部库。这方面最常用的库是 Redux。

Redux 的核心概念是，组件发出action与状态管理器通信。状态管理器收到 action 以后，使用 Reducer 函数算出新的状态，Reducer 函数的形式是(state, action) => newState。

useReducers()钩子用来引入 Reducer 功能。

```js
   const [state, dispatch] = useReducer(reducer, initialState);
```

上面是useReducer()的基本用法，它接受 Reducer 函数和状态的初始值作为参数，返回一个数组。数组的第一个成员是状态的当前值，第二个成员是发送 action 的dispatch函数。

下面是一个计数器的例子。用于计算状态的 Reducer 函数如下。
```js
   const myReducer = (state, action) => {
      switch(action.type)  {
         case('countUp'):
            return  {
            ...state,
            count: state.count + 1
            }
         default:
            return  state;
      }
   }
```

组件代码如下。
```js
   function App() {
      const [state, dispatch] = useReducer(myReducer, { count: 0 });
      return  (
         <div className="App">
            <button onClick={() => dispatch({ type: 'countUp' })}>
            +1
            </button>
            <p>Count: {state.count}</p>
         </div>
      );
   }
```

由于 Hooks 可以提供共享状态和 Reducer 函数，所以它在这些方面可以取代 Redux。但是，它没法提供中间件（middleware）和时间旅行（time travel），如果你需要这两个功能，还是要用 Redux。