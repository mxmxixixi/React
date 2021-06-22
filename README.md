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

#### 第六章 事件处理

##### 特点

- 在 React 中另一个不同点是你不能通过返回 `false` 的方式阻止默认行为。你必须显式的使用 `preventDefault`

##### 事件列表 

- onWheel
  - 主要是鼠标进行滚动、触摸板进行滑动，没有滚动条也会触发此事件
- onScroll
  - 含有滚动条时，滚动触发的事件

#### 第八章 列表和Keys

- React使用key属性来标记列表中每一个元素，当数据发生变化时，React就可以通过key知道哪些元素发生了变化，从而只重新渲染发生变化的元素，提高渲染效率。key 不需要全局唯一，但在列表中需要保持唯一
- Key 应该具有稳定，可预测，以及列表内唯一的特质。不稳定的 key（比如通过 `Math.random()` 生成的）会导致许多组件实例和 DOM 节点被不必要地重新创建，这可能导致性能下降和子组件中的状态丢失
- key 只是在兄弟节点之间必须唯一
- key 会传递信息给 React ，但不会传递给你的组件。如果你的组件中需要使用 `key` 属性的值，请用其他属性名显式传递这个值

#### 第九章 表单

```javascript
class Reservation extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isGoing: true,
      numberOfGuests: 2
    };

    this.handleInputChange = this.handleInputChange.bind(this);
  }

  handleInputChange(event) {
    const target = event.target;
    const value = target.name === 'isGoing' ? target.checked : target.value;
    const name = target.name;

    this.setState({
      [name]: value
    });
  }

  render() {
    return (
      <form>
        <label>
          参与:
          <input
            name="isGoing"
            type="checkbox"
            checked={this.state.isGoing}
            onChange={this.handleInputChange} />
        </label>
        <br />
        <label>
          来宾人数:
          <input
            name="numberOfGuests"
            type="number"
            value={this.state.numberOfGuests}
            onChange={this.handleInputChange} />
        </label>
      </form>
    );
  }
}
```

#### 第十章 React哲学

- 页面划分组件，根据ui以及功能进行划分，根据**单一功能原则**来判定组件的范围

## 高级指引

### [Context](https://react.docschina.org/docs/context.html)

- 数据是通过 props 属性自上而下（由父及子）进行传递的，但这种做法对于某些类型的属性而言是极其繁琐的（例如：地区偏好，UI 主题）
- Context 设计目的是为了共享那些对于一个组件树而言是“全局”的数据，例如当前认证的用户、主题或首选语言
- Context 主要应用场景在于很多不同层级的组件需要访问同样一些的数据。请谨慎使用，因为这会使得组件的复用性变差。

### [错误边界](https://react.docschina.org/docs/error-boundaries.html)

- **错误边界仅可以捕获其子组件的错误**，它无法捕获其自身的错误。如果一个错误边界无法渲染错误信息，则错误会冒泡至最近的上层错误边界，这也类似于 JavaScript 中 catch {} 的工作机制

- 错误边界的粒度由你来决定，可以将其包装在最顶层的路由组件并为用户展示一个 “Something went wrong” 的错误信息，就像服务端框架经常处理崩溃一样。你也可以将单独的部件包装在错误边界以保护应用其他部分不崩溃

  ```javascript
  static getDerivedStateFromError(error) {
      // 更新 state 使下一次渲染能够显示降级后的 UI
      return { hasError: true };
    }
  
    componentDidCatch(error, errorInfo) {
      // 你同样可以将错误日志上报给服务器
      logErrorToMyService(error, errorInfo);
    }

### [Refs 转发](https://react.docschina.org/docs/forwarding-refs.html)

- 第二个参数 `ref` 只在使用 `React.forwardRef` 定义组件时存在。常规函数和 class 组件不接收 `ref` 参数，且 props 中也不存在 `ref`。
- Ref 转发不仅限于 DOM 组件，你也可以转发 refs 到 class 组件实例中

- refs 将不会透传下去。这是因为 `ref` 不是 prop 属性。就像 `key` 一样，其被 React 进行了特殊处理。如果你对 HOC 添加 ref，该 ref 将引用最外层的容器组件，而不是被包裹的组件

### [Fragments](https://react.docschina.org/docs/fragments.html)

- React 中的一个常见模式是一个组件返回多个元素。Fragments 允许你将子列表分组，而无需向 DOM 添加额外节点
- 短语法`<> </>`，除了它不支持 key 或属性；
- 显式 `<React.Fragment>` 语法声明的片段可能具有 key

### [深入 JSX](https://react.docschina.org/docs/jsx-in-depth.html)

- ```javascript
  import React from 'react';
  import { PhotoStory, VideoStory } from './stories';
  
  const components = {
    photo: PhotoStory,
    video: VideoStory
  };
  
  function Story(props) {
    // 错误！JSX 类型不能是一个表达式。
    return <components[props.storyType] story={props.story} />;
  }
  ```

- ```javascript
  import React from 'react';
  import { PhotoStory, VideoStory } from './stories';
  
  const components = {
    photo: PhotoStory,
    video: VideoStory
  };
  
  function Story(props) {
    // 正确！JSX 类型可以是大写字母开头的变量。
    const SpecificStory = components[props.storyType];
    return <SpecificStory story={props.story} />;
  }
  ```

- Props 默认值为 “True”

  ```javascript
  <MyTextBox autocomplete />
  
  <MyTextBox autocomplete={true} />
  ```

### [性能优化](https://react.docschina.org/docs/optimizing-performance.html)

- 打包优化
- 虚拟化长列表：react-virtualized
- 避免调停:shouldComponentUpdate(),React.PureComponent
- 不可变数据:在赋值时不改变原对象或者数组

### [Portals](https://react.docschina.org/docs/portals.html)

- Portal 提供了一种将子节点渲染到存在于父组件以外的 DOM 节点的优秀的方案
- 在父组件里捕获一个来自 portal 冒泡上来的事件，使之能够在开发时具有不完全依赖于 portal 的更为灵活的抽象。例如，如果你在渲染一个 `<Modal />` 组件，无论其是否采用 portal 实现，父组件都能够捕获其事件

### [Profiler](https://react.docschina.org/docs/profiler.html)

- `Profiler` 测量渲染一个 React 应用多久渲染一次以及渲染一次的“代价”。 它的目的是识别出应用中渲染较慢的部分，或是可以使用[类似 memoization 优化](https://react.docschina.org/docs/hooks-faq.html#how-to-memoize-calculations)的部分，并从相关优化中获益

### [协调](https://react.docschina.org/docs/reconciliation.html)

#### Diffing 算法

- 该算法不会尝试匹配不同组件类型的子树。如果你发现你在两种不同类型的组件中切换，但输出非常相似的内容，建议把它们改成同一类型。在实践中，我们没有遇到这类问题。
- Key 应该具有稳定，可预测，以及列表内唯一的特质。不稳定的 key（比如通过 `Math.random()` 生成的）会导致许多组件实例和 DOM 节点被不必要地重新创建，这可能导致性能下降和子组件中的状态丢失

### [Render Props](https://react.docschina.org/docs/render-props.html)

- render prop 是一个用于告知组件需要渲染什么内容的函数 prop

  ```javascript
  function withMouse(Component) {
    return class extends React.Component {
      render() {
        return (
          <Mouse render={mouse => (
            <Component {...this.props} mouse={mouse} />
          )}/>
        );
      }
    }
  }
  ```

- render prop 是因为模式才被称为 *render* prop ，你不一定要用名为 `render` 的 prop 来使用这种模式。事实上， [*任何*被用于告知组件需要渲染什么内容的函数 prop 在技术上都可以被称为 “render prop”](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce)

- 由于render直接写函数回在每次渲染时生成新的函数，所以Mouse组件继承React.PureComponent是无法优化性能的，要想继续用React.PureComponent，可以把render的函数变成外部写，而不是每次生成新的函数

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

### React.memo

- 性能优化特别好

- 如果你的组件在相同 props 的情况下渲染相同的结果，那么你可以通过将其包装在 `React.memo` 中调用，以此通过记忆组件渲染结果的方式来提高组件的性能表现。这意味着在这种情况下，React 将跳过渲染组件的操作并直接复用最近一次渲染的结果
- 默认情况下其只会对复杂对象做浅层对比，如果你想要控制对比过程，那么请将自定义的比较函数通过第二个参数传入来实现
- 当 context 发生变化时，它仍会重新渲染

## Hook

### 1. Hook 简介

#### 1.1 没有破坏性改动

- 完全可选；
- 100%向后兼容；
- 现在可用(Hook 发布于 16.8 版本)；

#### 1.2 动机(Hook的优点,为什么要在 React 中引入 Hook 的原因)

- 组件中复用状态逻辑很难(现在主要是用 render props 或高阶组件，这样会形成嵌套地域)；
  - hook 可以在无需改变组件结构的情况下复用状态逻辑**(使用方便)**
  - 使用 hook 从组件中**提取状态逻辑**，使得这些逻辑可以单独测试并使用**(全局使用hook存储用户的信息，Hook 和现有代码可以同时工作，可以渐进式地使用他们)**；
- 复杂组件变得难以理解(组件中需要处理的逻辑很多，同一个钩子函数处理不同的逻辑，维护比较困难)；

  - Hook 将组件中相互关联的部分拆成更小的函数，而非强制按照生命周期划分**(可以在每一个hook中完成一个功能点的逻辑)**；
- 难以理解的 class(绑定 this、class 组件不好压缩，并且会使热重载出现不稳定的情况)；

  - Hook 在你非 class 的情况下可以使用更多的 React 特性**(不需要绑定this)**
  - **不能在 class 组件*内部*使用 Hook

#### 1.3 渐进策略

**没有计划从 React 中移除 class**

Hook 与现在有的代码可以同时工作，可以渐进式使用他们；

#### 1.4 FAQ

### 2. Hook 概览

- 什么是 Hook？
  - Hook 是一些可以让你在函数组件里"钩入"React State 以及生命周期等特性的函数，Hook 不能在 class 组件中使用。React 中内置了一些像 useState 这样的 Hook，也可以创建自己的 Hook 来复用组件之间的逻辑；Hook 使用了 javascript 的闭包机制；

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

  定义了一个 state 变量，在函数退出后变量就会”消失”，而 state 中的变量会被 React 保留，运用闭包原理实现变量保存；

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

- **React 会在组件卸载的时候执行清除操作**；`useEffect` 可以在组件渲染后实现各种不同的副作用。有些副作用可能需要清除，所以需要返回一个函数；
- effect 的清除阶段在每次**重新渲染**时都会执行，而不是只在卸载组件的时候执行一次，这样设计替代componentDidUpdate生命周期，重新渲染代表着第一次渲染不执行

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

#### 2.3 useContext

- 定义以及用途同Context【高级指引中】

#### 2.3 useCallback

- 当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（例如 `shouldComponentUpdate`）的子组件时，它将非常有用
- `useCallback(fn, deps)` 相当于 `useMemo(() => fn, deps)`

#### 2.4 useMemo

- 把“创建”函数和依赖项数组作为参数传入 `useMemo`，它仅会在某个依赖项改变时才重新计算 memoized 值。这种优化有助于避免在每次渲染时都进行高开销的计算。
- 传入 `useMemo` 的函数会在渲染期间执行。请不要在这个函数内部执行与渲染无关的操作，诸如副作用这类的操作属于 `useEffect` 的适用范畴，而不是 `useMemo`
- 如果没有提供依赖项数组，`useMemo` 在每次渲染时都会计算新的值

#### 2.5 useRef

- `useRef` 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传入的参数（`initialValue`）。返回的 ref 对象在组件的整个生命周期内保持不变
- `useRef()` 比 `ref` 属性更有用。它可以很方便地保存任何可变值
- 当 ref 对象内容发生变化时，`useRef` 并*不会*通知你。变更 `.current` 属性不会引发组件重新渲染。如果想要在 React 绑定或解绑 DOM 节点的 ref 时运行某些代码，则需要使用回调 ref来实现

#### 2.5 useImperativeHandle

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

#### 2.6 useRequest【ahooks】

- ready只能是布尔值，不能是表达式的布尔值

#### 自定义Hook

- 自定义 Hook 是一个函数，其名称以 “`use`” 开头，函数内部可以调用其他的 Hook。
- **自定义 Hook 必须以 “`use`” 开头吗？**必须如此
- **在两个组件中使用相同的 Hook 会共享 state 吗？**不会。自定义 Hook 是一种重用*状态逻辑*的机制(例如设置为订阅并存储当前值)，所以每次使用自定义 Hook 时，其中的所有 state 和副作用都是完全隔离的。
- 

## Hook 规则

### 3.1 只在最顶层使用 Hook

**不要在循环，条件或嵌套函数中调用 Hook，** 确保总是在你的 React 函数的最顶层调用他们。遵守这条规则，你就能确保 Hook 在每一次渲染中都按照同样的顺序被调用。这让 React 能够在多次的 `useState` 和 `useEffect` 调用之间保持 hook 状态的正确。如果我们想要有条件地执行一个 effect，可以将判断放到 Hook 的*内部*：

## HOOK总结

- 优点

  - 有什么是 Hook 能做而 class 做不到的

    > Hook 提供了强大而富有表现力的方式来在组件间复用功能。通过 [「自定义 Hook」](https://react.docschina.org/docs/hooks-custom.html) 这一节可以了解能用它做些什么

  - Hook 避免了 class 需要的额外开支，像是创建类实例和在构造函数中绑定事件处理器的成本。

  - **符合语言习惯的代码在使用 Hook 时不需要很深的组件树嵌套**。这个现象在使用高阶组件、render props、和 context 的代码库中非常普遍。组件树小了，React 的工作量也随之减少

- 缺点
  - 容易造成函数中拿不到最新的值【闭包造成】
  - hook的依赖项手动添加，非常繁琐
  - 目前暂时还没有对应不常用的 `getSnapshotBeforeUpdate`，`getDerivedStateFromError` 和 `componentDidCatch` 生命周期的 Hook 等价写法