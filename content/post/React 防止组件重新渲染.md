---
title: "React 防止组件重新渲染"
date: 2023-05-03T18:33:42+08:00
draft: false
tags: [react, performance, frontend]
---

在React中，组件的重新渲染通常由状态（state）或属性（props）的变化触发。虽然React通过虚拟DOM和高效的对比算法（diffing algorithm）优化了性能，但在某些情况下，不必要的重新渲染仍然会导致性能问题。因此，防止不必要的重新渲染是优化React应用性能的关键策略之一。以下是几种常用的方法：

### 1. 使用PureComponent

在类组件中，可以通过继承`React.PureComponent`而不是`React.Component`来减少不必要的渲染。`PureComponent`通过浅比较（shallow comparison）`props`和`state`来自动检查是否需要重新渲染组件。

```jsx
import React, { PureComponent } from 'react';

class MyComponent extends PureComponent {
  render() {
    return <div>{this.props.message}</div>;
  }
}
```

注意：`PureComponent`进行的是浅比较，所以如果`props`或`state`的结构比较复杂，可能需要其他策略。

浅比较（Shallow Comparison）是指对两个对象的属性进行表层比较。在浅比较中，如果对象的所有属性值都相等（对于基本数据类型比较值，对于引用数据类型比较内存地址），那么这两个对象被认为是相等的。浅比较并不会递归比较对象的属性值中可能包含的对象或数组，即它不检查嵌套的对象或数组。

### 2. 使用React.memo

对于函数组件，可以使用`React.memo`来避免不必要的渲染。`React.memo`是一个高阶组件，它类似于`PureComponent`，但用于函数组件，它也会对组件的`props`进行浅比较。

```jsx
const MyComponent = React.memo(function MyComponent(props) {
  return <div>{props.message}</div>;
});
```

### 3. 使用shouldComponentUpdate

在类组件中，可以通过定义`shouldComponentUpdate`方法来决定组件是否需要更新。此方法接受新的`props`和`state`作为参数，并返回一个布尔值，指示组件是否应该更新。

```jsx
class MyComponent extends React.Component {
  shouldComponentUpdate(nextProps, nextState) {
    // 自定义的比较逻辑
    return nextProps.message !== this.props.message;
  }

  render() {
    return <div>{this.props.message}</div>;
  }
}
```

### 4. 使用React.useMemo和React.useCallback

对于函数组件，可以使用`useMemo`和`useCallback`钩子来缓存计算结果和回调函数，避免在每次渲染时都重新计算或创建。

```jsx
import React, { useMemo, useCallback } from 'react';

const MyComponent = ({ value }) => {
  const expensiveResult = useMemo(() => computeExpensiveValue(value), [value]);
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []); // 依赖列表为空，回调不会在重新渲染时改变

  return <button onClick={handleClick}>Click me</button>;
};
```

### 5. 避免在渲染方法中创建新的对象或函数

在类组件的`render`方法或函数组件中直接定义对象、数组或函数，会在每次渲染时创建新的引用，可能会触发子组件的不必要渲染。

错误示例：

```jsx
const MyComponent = ({ value }) => {
  const handleClick = () => { // 每次渲染都会创建新的handleClick
    console.log(value);
  };

  return <ChildComponent onClick={handleClick} />;
};
```

正确做法是使用`useCallback`钩子缓存函数，以及将对象和数组等数据结构放在组件的状态中，或使用`useMemo`钩子缓存。
