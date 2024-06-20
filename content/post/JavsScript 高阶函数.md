---
title: "JavsScript 高阶函数"
date: 2022-04-24T21:14:54+08:00
draft: false
categories: [JavaScript]
---

在 JavaScript 中，如果一个函数能接收一个或多个函数作为参数，或返回另一个函数，那么该函数被称为**高阶函数**。

高阶函数是函数式编程的一个重要概念，提供了极大的灵活性和表达力。以下是 JavaScript 中常见的高阶函数：

### Array.prototype.map

`map` 方法创建一个新数组，该数组中的每个元素是调用对象中每个元素调用一次提供的函数后的返回值。

```
const numbers = [1, 2, 3, 4];
const doubled = numbers.map(x => x * 2); // [2, 4, 6, 8]
```

### Array.prototype.filter

`filter` 方法创建一个新数组，包含通过所提供函数实现的测试的所有元素。

```
const numbers = [1, 2, 3, 4];
const evens = numbers.filter(x => x % 2 === 0); // [2, 4]
```

### Array.prototype.reduce

`reduce` 方法对数组中的每个元素执行一个由您提供的reducer函数(升序执行)，将其结果汇总为单个返回值。

```
const numbers = [1, 2, 3, 4];
const sum = numbers.reduce((accumulator, currentValue) => accumulator + currentValue, 0); // 10
```

### Array.prototype.forEach

虽然`forEach`不返回任何值（返回`undefined`），但它接收一个函数作为参数，对数组的每个元素执行该函数，因此可以被视为一个高阶函数。

```
const numbers = [1, 2, 3, 4];
numbers.forEach(x => console.log(x)); // 依次打印 1, 2, 3, 4
```

### Function.prototype.bind

`bind` 方法创建一个新的函数，在调用时设置`this`关键字为提供的值，并在调用新函数时前置给定的参数序列。

```
const module = {
  x: 42,
  getX: function() {
    return this.x;
  }
};

const unboundGetX = module.getX;
console.log(unboundGetX()); // 函数被调用在全局作用域中 - 返回 `undefined`

const boundGetX = unboundGetX.bind(module);
console.log(boundGetX()); // 42
```

### Promise.then 和 Promise.catch

这些`Promise`的方法也是高阶函数，因为它们接收函数作为参数，并返回一个新的`Promise`。

```
Promise.resolve(1)
  .then(x => x + 1)
  .catch(err => console.error(err))
  .then(x => console.log(x)); // 2
```
