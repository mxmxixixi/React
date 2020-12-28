## 核心概念

#### 第四章 组件 & Props

##### 4.1 函数组件

暂未记录

##### 4.2 继承组件

- React.Component

- React.PureComponent

  React.PureComponent 与 React.Component 很相似。两者的区别在于 React.Component 并未实现 shouldComponentUpdate()，而 React.PureComponent 中以浅层对比 prop 和 state 的方式来实现了该函数。如果赋予 React 组件相同的 props 和 state，render() 函数会渲染相同的内容，那么在某些情况下使用 React.PureComponent 可提高性能。

#### 第五章 State & 生命周期

##### state 不可变值

> 为什么 React 推荐组件的状态是不可变对象呢？一方面是因为不可变对象方便管理和调试；另一方面是出于性能考虑，当对象组件状态都是不可变对象时，我们在组件的`shouldComponentUpdate`方法中，仅需要比较状态的引用就可以判断状态是否真的改变，从而避免不必要的`render`调用。当我们使用 React 提供的`PureComponent`时，更是要保证组件状态是不可变对象，否则在组件的`shouldComponentUpdate`方法中，状态比较就可能出现错误，因为`PureComponent`执行的是浅比较（比较对象的引用）。

> 在`shouldComponentUpdate()`(用在子组件中，接受参数有父组件传过来的参数)判断中，有一个有意思的问题，解释为什么 React `setState()` 要用不可变值
>
> 子组件将始终不会渲染，因为在`shouldComponentUpdate()`中，`this.state.list.push()`已经修改了`this.props.list`，而`this.setState()`修改了`nextProps.list`所以两个值深度比较，将始终相同。

> state 中的值在 this.setState 之前都是保持不变的，render 之前覆盖成新值，这样在 this.setState 之后，更新触发的钩子函数可以拿到更改前后的值进行比较

```javascript
// 父组件中
changeList () {
    this.state.list.push({id: 2})
    this.setState({
        list: this.state.list
    })
}
// 子组件中
import _ from 'lodash'
shouldComponentUpdate(nextProps, nextState) {
    // 数组深度比较（一次性递归到底，耗费性能，工作中慎用）
    if (_.isEqual(nextProps.list, this.props.list)) {
        return false // 相等，不渲染
    }
    return true // 不相等，渲染
}
```

- #### 状态的类型是不可变类型（数字，字符串，布尔值，null， undefined）

> 这种情况最简单，因为状态是不可变类型，直接给要修改的状态赋一个新值即可。如要修改 count（数字类型）、title（字符串类型）、success（布尔类型）三个状态：

```
this.setState({
  count: 1,
  title: 'Redux',
  success: true
})
```

- #### 状态的类型是数组

> 如有一个数组类型的状态 books，当向 books 中增加一本书时，使用数组的 concat 方法或 ES6 的数组扩展语法（spread syntax）：

```javascript
// 方法一：将state先赋值给另外的变量，然后使用concat创建新数组
var books = this.state.books;
this.setState({
  books: books.concat(['React Guide']);
})

// 方法二：使用preState、concat创建新数组
this.setState(preState => ({
  books: preState.books.concat(['React Guide']);
}))

// 方法三：ES6 spread syntax
this.setState(preState => ({
  books: [...preState.books, 'React Guide'];
}))
```

> 当从 books 中截取部分元素作为新状态时，使用数组的 slice 方法：

```javascript
// 方法一：将state先赋值给另外的变量，然后使用slice创建新数组
var books = this.state.books;
this.setState({
  books: books.slice(1,3);
})

// 方法二：使用preState、slice创建新数组
this.setState(preState => ({
  books: preState.books.slice(1,3);
}))
```

> 当从 books 中过滤部分元素后，作为新状态时，使用数组的 filter 方法

```javascript
// 方法一：将state先赋值给另外的变量，然后使用filter创建新数组
var books = this.state.books;
this.setState({
  books: books.filter(item => {
    return item != 'React';
  });
})

// 方法二：使用preState、filter创建新数组
this.setState(preState => ({
  books: preState.books.filter(item => {
    return item != 'React';
  });
}))
```

> 注意不要使用 push、pop、shift、unshift、splice 等方法修改数组类型的状态，因为这些方法都是在原数组的基础上修改，而 concat、slice、filter 会返回一个新的数组。

- #### 状态的类型是普通对象（不包含字符串、数组）

> 使用 ES6 的 Object.assgin 方法

```javascript
// 方法一：将state先赋值给另外的变量，然后使用Object.assign创建新对象
var owner = this.setState.owner;
this.setState({
  owner: Object.assign({}, owner, {name: 'Jason'});
})

// 方法二：使用preState、Object.assign创建新对象
this.setState(preState => ({
  owner: Object.assign({}, preState.owner, {name: 'Jason'});
}))
```

> 使用对象扩展语法（[object spread properties](https://github.com/sebmarkbage/ecmascript-rest-spread)）

```javascript
// 方法一：将state先赋值给另外的变量，然后使用对象扩展语法创建新对象
var owner = this.setState.owner;
this.setState({
  owner: {...owner, {name: 'Jason'}};
})

// 方法二：使用preState、对象扩展语法创建新对象
this.setState(preState => ({
  owner: {...preState.owner, {name: 'Jason'}};
}))
```

##### 钩子函数

- componentDidCatch

```javascript
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { error: null, errorInfo: null };
  }

  componentDidCatch(error, errorInfo) {
    // Catch errors in any components below and re-render with error message
    this.setState({
      error: error,
      errorInfo: errorInfo,
    });
    // You can also log error messages to an error reporting service here
  }

  render() {
    if (this.state.errorInfo) {
      // Error path
      return (
        <div>
          <h2>Something went wrong.</h2>
          <details style={{ whiteSpace: "pre-wrap" }}>
            {this.state.error && this.state.error.toString()}
            <br />
            {this.state.errorInfo.componentStack}
          </details>
        </div>
      );
    }
    // Normally, just render children
    return this.props.children;
  }
}

class BuggyCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { counter: 0 };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(({ counter }) => ({
      counter: counter + 1,
    }));
  }

  render() {
    if (this.state.counter === 5) {
      // Simulate a JS error
      throw new Error("I crashed!");
    }
    return <h1 onClick={this.handleClick}>{this.state.counter}</h1>;
  }
}

function App() {
  return (
    <div>
      <p>
        <b>
          This is an example of error boundaries in React 16.
          <br />
          <br />
          Click on the numbers to increase the counters.
          <br />
          The counter is programmed to throw when it reaches 5. This simulates a
          JavaScript error in a component.
        </b>
      </p>
      <hr />
      <ErrorBoundary>
        <p>
          These two counters are inside the same error boundary. If one crashes,
          the error boundary will replace both of them.
        </p>
        <BuggyCounter />
        <BuggyCounter />
      </ErrorBoundary>
      <hr />
      <p>
        These two counters are each inside of their own error boundary. So if
        one crashes, the other is not affected.
      </p>
      <ErrorBoundary>
        <BuggyCounter />
      </ErrorBoundary>
      <ErrorBoundary>
        <BuggyCounter />
      </ErrorBoundary>
    </div>
  );
}
ReactDOM.render(<App />, document.getElementById("root"));
```

## 高级指引

### 使用 PropTypes 进行类型检查

> `PropTypes` 提供一系列验证器，可用于确保组件接收到的数据类型是有效的;出于性能方面的考虑，`propTypes` 仅在开发模式下进行检查;

#### 1.基本使用方法

```javascript
import PropTypes from "prop-types";

class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}

Greeting.propTypes = {
  name: PropTypes.string,
};
```

#### 2.检验类型

```javascript
import PropTypes from "prop-types";

MyComponent.propTypes = {
  // 你可以将属性声明为 JS 原生类型，默认情况下
  // 这些属性都是可选的。
  optionalArray: PropTypes.array,
  optionalBool: PropTypes.bool,
  optionalFunc: PropTypes.func,
  optionalNumber: PropTypes.number,
  optionalObject: PropTypes.object,
  optionalString: PropTypes.string,
  optionalSymbol: PropTypes.symbol,

  // 任何可被渲染的元素（包括数字、字符串、元素或数组）
  // (或 Fragment) 也包含这些类型。
  optionalNode: PropTypes.node,

  // 一个 React 元素。
  optionalElement: PropTypes.element,

  // 一个 React 元素类型（即，MyComponent）。
  optionalElementType: PropTypes.elementType,

  // 你也可以声明 prop 为类的实例，这里使用
  // JS 的 instanceof 操作符。
  optionalMessage: PropTypes.instanceOf(Message),

  // 你可以让你的 prop 只能是特定的值，指定它为
  // 枚举类型。
  optionalEnum: PropTypes.oneOf(["News", "Photos"]),

  // 一个对象可以是几种类型中的任意一个类型
  optionalUnion: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.instanceOf(Message),
  ]),

  // 可以指定一个数组由某一类型的元素组成
  optionalArrayOf: PropTypes.arrayOf(PropTypes.number),

  // 可以指定一个对象由某一类型的值组成
  optionalObjectOf: PropTypes.objectOf(PropTypes.number),

  // 可以指定一个对象由特定的类型值组成
  optionalObjectWithShape: PropTypes.shape({
    color: PropTypes.string,
    fontSize: PropTypes.number,
  }),

  // An object with warnings on extra properties
  optionalObjectWithStrictShape: PropTypes.exact({
    name: PropTypes.string,
    quantity: PropTypes.number,
  }),

  // 你可以在任何 PropTypes 属性后面加上 `isRequired` ，确保
  // 这个 prop 没有被提供时，会打印警告信息。
  requiredFunc: PropTypes.func.isRequired,

  // 任意类型的数据
  requiredAny: PropTypes.any.isRequired,

  // 你可以指定一个自定义验证器。它在验证失败时应返回一个 Error 对象。
  // 请不要使用 `console.warn` 或抛出异常，因为这在 `onOfType` 中不会起作用。
  customProp: function (props, propName, componentName) {
    if (!/matchme/.test(props[propName])) {
      return new Error(
        "Invalid prop `" +
          propName +
          "` supplied to" +
          " `" +
          componentName +
          "`. Validation failed."
      );
    }
  },

  // 你也可以提供一个自定义的 `arrayOf` 或 `objectOf` 验证器。
  // 它应该在验证失败时返回一个 Error 对象。
  // 验证器将验证数组或对象中的每个值。验证器的前两个参数
  // 第一个是数组或对象本身
  // 第二个是他们当前的键。
  customArrayProp: PropTypes.arrayOf(function (
    propValue,
    key,
    componentName,
    location,
    propFullName
  ) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        "Invalid prop `" +
          propFullName +
          "` supplied to" +
          " `" +
          componentName +
          "`. Validation failed."
      );
    }
  }),
};
```

#### 3.默认值

```javascript
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}

// 指定 props 的默认值：
Greeting.defaultProps = {
  name: "Stranger",
};

// 渲染出 "Hello, Stranger"：
ReactDOM.render(<Greeting />, document.getElementById("example"));
```

## Hook

### 1. Hook 简介

#### 1.1 没有破坏性改动

- 完全可选；
- 100%向后兼容；
- 现在可用(Hook 发布于 16.8 版本)；

#### 1.2 动机

- 组件中复用状态逻辑很难(现在主要是用 render props 或高阶组件)；

  使用 hook 从组件中提取状态逻辑，使得这些逻辑可以单独测试并使用、hook 可以在无需改变组件结构的情况下复用状态逻辑；

- 复杂组件变得难以理解(组件中需要处理的逻辑很多，不同的钩子函数处理不同的逻辑，维护比较困难)；

  Hook 将组件中相互关联的部分拆成更小的函数，而非强制按照生命周期划分；

- 难以理解的 class(绑定 this、class 组件不好压缩，并且会使热重载出现不稳定的情况)；

  Hook 在你非 class 的情况下可以使用更多的 React 特性；

#### 1.3 渐进策略

**没有计划从 React 中移除 class**

Hook 与现在有的代码可以同时工作，可以渐进式使用他们；

#### 1.4 FAQ

### 2. Hook 概览

> 什么是 Hook？

Hook 是一些可以让你在函数组件里"钩入"React State 以及生命周期等特性的函数，Hook 不能在 class 组件中使用。React 中内置了一些像 useState 这样的 Hook，也可以创建自己的 Hook 来复用组件之间的逻辑；Hook 使用了 javascript 的闭包机制；

#### 2.1 State Hook

```javascript
import React, { useState } from "react";
function Example() {
  // 声明一个叫 “count” 的 state 变量， 使用数组解构方式
  const [count, setCount] = useState(0);
  //可以同时申明多个state，渲染时顺序不变
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState("banana");
  const [todos, setTodos] = useState([{ text: "Learn Hooks" }]);
  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

##### 2.1.1 声明 State 变量

- **`调用 useState` 方法的时候做了什么?**

  定义了一个 state 变量，在函数退出后变量就会”消失”，而 state 中的变量会被 React 保留；

- **`useState` 需要哪些参数？**

  `useState()` 方法里面唯一的参数就是初始 state。不同于 class 的是，我们可以按照需要使用数字、字符串、函数对其进行赋值，而不一定是对象(this.state 是一个对象)；

- **`useState` 方法的返回值是什么?**

  useState 会返回一对值：当前状态和一个让你更新它的函数，它不会把新的 state 和旧的 state 进行合并，需要成对的获取它们；React 会在重复渲染时记住它当前的值，并且提供最新的值给我们的函数

##### 2.1.2 读取 State

```javascript
//class组件
<p>You clicked {this.state.count} times</p>
//Hook
<p>You clicked {count} times</p>
```

##### 2.1.3 更新 State

```javascript
//class组件
<button onClick={() => this.setState({ count: this.state.count + 1 })}>
  Click me
</button>
//hook,不需要使用this
<button onClick={() => setCount(count + 1)}>
    Click me
  </button>
```

##### 2.1.4 渲染问题

- 当我们在使用 `useState` 时，修改值时传入同样的值，不会重复渲染；

  ```javascript
  function App() {
    const [count, setCount] = useState(0);
    console.log("component render");
    return (
      <div>
        点击次数: {count}
        <button
          onClick={() => {
            setCount(count);
          }}
        >
          点击
        </button>
      </div>
    );
  }
  ```

- 组件重复渲染过程中，`useState` 中的函数只会执行一次

  ```javascript
  function App(props) {
    const [count, setCount] = useState(() => {
      console.log("useState default value function is call");
      return props.count || 0;
    });
    return (
      <div>
        点击次数: {count}
        <button
          onClick={() => {
            setCount(count + 1);
          }}
        >
          点击
        </button>
      </div>
    );
  }
  ```

- 多个 useState(HOOK)使用情况

  不要在循环，条件或嵌套函数中调用 `Hook`， 确保总是在你的 `React` 函数的最顶层调用他们。遵守这条规则，你就能确保 `Hook` 在每一次渲染中都按照同样的顺序被调用。这让 `React` 能够在多次的 `useState` 和 `useEffect` 调用之间保持 `hook` 状态的正确

#### 2.2 Effect Hook

`useEffect` 就是一个 Effect Hook，给函数组件增加了操作副作用的能力(与`componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 具有相同的作用)，副作用是除了与状态相关的逻辑，如网络请求，监听事件，查找 DOM；effect 发生在组件渲染之后，不用考虑挂载和更新的问题

```javascript
import React, { useState, useEffect } from "react";

function Example() {
  const [count, setCount] = useState(0);

  // 相当于 componentDidMount 和 componentDidUpdate:
  useEffect(() => {
    // 使用浏览器的 API 更新页面标题
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

##### 2.2.1 无需清除的 effect

##### 2.2.2 需要清除的 effect

React 会在组件卸载的时候执行清除操作；`useEffect` 可以在组件渲染后实现各种不同的副作用。有些副作用可能需要清除，所以需要返回一个函数；

```javascript
import React, { useState, useEffect } from "react";

function FriendStatus(props) {
  const [isOnline, setIsOnline] = useState(null);

  useEffect(() => {
    function handleStatusChange(status) {
      setIsOnline(status.isOnline);
    }
    ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
    // Specify how to clean up after this effect:
    return function cleanup() {
      ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
    };
  });

  if (isOnline === null) {
    return "Loading...";
  }
  return isOnline ? "Online" : "Offline";
}
```

##### 2.2.3 使用多个 Effect 实现关注点分离

**Hook 允许我们按照代码的用途分离他们**；

##### 2.2.4 通过跳过 Effect 进行性能优化

通过 effect 的第二个参数进行控制；

- 如果想执行只运行一次的 effect（仅在组件挂载和卸载时执行），可以传递一个空数组（`[]`）作为第二个参数；这就告诉 React 你的 effect 不依赖于 props 或 state 中的任何值，所以它永远都不需要重复执行；
- 如果数组中有多个元素，即使只有一个元素发生变化，React 也会执行 effect；
- 什么都不传，组件每次 `render` 之后 `useEffect` 都会调用，

#### 2.3 useCallback

#### 2.4 useImperativeHandle

可以在使用 ref 时自定义暴露给父组件的实例值，useImperativeHandle 一般与 forwardref 使用

```javascript
//forwardref接收组件，组件的入参分为两个，props为一般的参数，ref为传递的父级调用子组件时设置的ref
const child = forwardref((props,ref)=>{
  const inputRef = useRef();
  //第一个参数：暴露哪个ref；第二个参数：暴露什么
  useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus();
    }
  }));
  return <input ref={inputRef} ... />;
})
const parent = ()=>{
  const childRef = useRef();
  useEffect(()=>{
    childRef.current.focus
  })
  return <div>
    <child ref={childRef}/>
    </div>
}
```

## Hook 规则

### 3.1 只在最顶层使用 Hook

**不要在循环，条件或嵌套函数中调用 Hook，** 确保总是在你的 React 函数的最顶层调用他们。遵守这条规则，你就能确保 Hook 在每一次渲染中都按照同样的顺序被调用。这让 React 能够在多次的 `useState` 和 `useEffect` 调用之间保持 hook 状态的正确。如果我们想要有条件地执行一个 effect，可以将判断放到 Hook 的*内部*：
