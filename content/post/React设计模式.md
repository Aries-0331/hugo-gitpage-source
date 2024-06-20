---
title: "React 设计模式"
date: 2022-01-11T14:55:46+08:00
draft: false
categories: [React]
---

常见的 React 设计模式：

### 1. 组件化

将UI拆分成独立、可复用的组件是React开发的核心。每个组件只负责一部分的UI呈现，保持组件的职责单一。

- **原子设计**：将UI拆分成最小单元（原子），然后组合成复杂的UI结构（分子和生物）。

### 2. 容器组件和展示组件

这种模式通过将业务逻辑和UI呈现分离到两类不同的组件中来提高组件的复用性和可维护性。

- **展示组件**：关注于如何展示，通常只包含UI呈现逻辑。
- **容器组件**：关注于如何工作，包含业务逻辑和数据处理。

### 3. 高阶组件（HOC）

高阶组件是参数为组件，返回值为新组件的函数。HOC用于复用组件逻辑。

```jsx
function withSubscription(WrappedComponent) {
  return class extends React.Component {
    /* ... */
    render() {
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```

### 4. 函数作为子组件（Function as Child Component，FACC）

这种模式利用了React组件的`children`属性可以是任何类型，包括函数。这允许父组件向子组件传递数据和逻辑，而不是通过props。

```jsx
<Mouse render={mouse => (
  <Cat mouse={mouse} />
)}/>
```

### 5. 渲染道具（Render Props）

渲染道具是一种技术，通过一个函数prop（通常称为`render`），将组件的内部状态或逻辑传递给子组件。这与FACC非常相似，但更通用。

```jsx
<DataProvider render={data => (
  <h1>{data.title}</h1>
)}/>
```

### 6. 钩子（Hooks）

自React 16.8起，Hooks允许在无状态组件中使用状态和其他React特性，如`useState`和`useEffect`等。

`useState`允许函数组件拥有自己的状态。

```jsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
}
```

`useEffect`使得我们能够执行副作用操作，等同于类组件中的生命周期方法`componentDidMount`、`componentDidUpdate`和`componentWillUnmount`的组合。

```jsx
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // 类似于componentDidMount 和 componentDidUpdate:
  useEffect(() => {
    // 更新文档的标题
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

`useContext`让我们能够在函数组件中使用React的Context API来实现跨组件的数据传递。

```jsx
import React, { useContext } from 'react';
const ThemeContext = React.createContext('light');

function ThemedButton() {
  const theme = useContext(ThemeContext);
  return <button theme={theme}>I am styled by theme context!</button>;
}

```

自定义钩子允许我们将组件逻辑提取到可重用的函数中。

```jsx
import React, { useState, useEffect } from 'react';

function useCounter(initialCount = 0) {
  const [count, setCount] = useState(initialCount);

  useEffect(() => {
    document.title = `Count: ${count}`;
  });

  return [count, setCount];
}

function Counter() {
  const [count, setCount] = useCounter();

  return (
    <div>
      <p>{count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}

```

React Hooks 不仅仅是一种语法糖，它们引入了一种全新的组件设计模式，使得状态和逻辑更容易在组件之间共享。通过自定义钩子，开发者可以构建高度复用的逻辑单元，从而使代码更加干净和模块化。这种模式减少了高阶组件（HOC）和渲染道具（Render Props）模式的需要，提供了一种更简单直观的方式来共享逻辑。

### 7. 上下文（Context）API

Context提供了一种在组件树中传递数据的方式，无需通过每层组件显式传递props。

```jsx
<UserContext.Provider value={user}>
  <Component />
</UserContext.Provider>
```

### 8. 受控组件和非受控组件

在React中处理表单输入时，受控组件通过React的状态来管理输入的值，而非受控组件则使用DOM本身的状态。

### 9. 组件复合（Composition）

通过组合而非继承来构建组件之间的关系，以实现更灵活的代码复用。这通常涉及到将子组件作为props传递给父组件。

假设我们有几个基础UI组件：`Button`、`Dialog`。

```jsx
// Button 组件
function Button({ children, onClick }) {
  return (
    <button onClick={onClick}>{children}</button>
  );
}

// Dialog 组件
function Dialog({ children }) {
  return (
    <div className="dialog">
      {children}
    </div>
  );
}
```

现在，我们想要创建一个`WelcomeDialog`组件，它使用`Dialog`组件来展示欢迎信息，并且包含一个关闭按钮。

```jsx
function WelcomeDialog() {
  const handleClose = () => alert('Dialog will close now.');
  
  return (
    <Dialog>
      <h1>Welcome!</h1>
      <p>Thank you for visiting our spacecraft!</p>
      <Button onClick={handleClose}>Close</Button>
    </Dialog>
  );
}
```

在这个例子中，`WelcomeDialog`组件通过将`Button`组件作为子组件传递给`Dialog`组件来复合它们。这展示了组件复合的基本思想：通过将子组件嵌入到父组件中来构建复杂的组件。

### 10. 错误边界（Error Boundaries）

错误边界是React组件，它可以捕获其子组件树中JavaScript错误，记录这些错误，并显示一个备用UI。

```jsx
<ErrorBoundary>
  <MyComponent />
</ErrorBoundary>
```

这些设计模式并不是孤立使用的，实际工作中需要灵活选择结合使用。
