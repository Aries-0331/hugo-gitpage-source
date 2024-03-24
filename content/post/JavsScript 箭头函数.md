---
title: "JavsScript 箭头函数"
date: 2022-01-24T21:40:30+08:00
draft: false
tags: [javascript]
---

## 箭头函数

箭头函数是 ES6 (ECMAScript 2015) 引入的一个新增特性，提供了一种更简洁的方式来写函数表达式。它不仅语法简短，而且还有其他几个特别之处，特别是与`this`关键字的绑定相关的行为。

箭头函数的基本语法如下：

```
const functionName = (arg1, arg2, ...argN) => {
  // 函数体
};
```

如果箭头函数直接返回一个值，可以省略花括号和`return`关键字：

```

const functionName = (arg1, arg2) => arg1 + arg2;
```

如果只有一个参数，还可以省略参数周围的括号：

```

const functionName = arg => arg * 2;
```

### `this`的绑定

箭头函数不绑定自己的`this`，它会捕获其所在上下文的`this`值作为自己的`this`值，这使得在回调函数和闭包中处理`this`变得更加容易。

```
function Counter() {
  this.count = 0;
  setInterval(() => {
    this.count++;
    console.log(this.count);
  }, 1000);
}
```

在这个例子中，`setInterval`内部的箭头函数没有自己的`this`绑定，因此它可以直接访问外围`Counter`函数中的`this`对象。

### 使用限制

- **没有自己的`this`**：如上所述，箭头函数不绑定`this`，意味着不能用它来定义对象方法，如果尝试通过`new`关键字调用箭头函数，会抛出一个 TypeError 错误。
- **没有`arguments`对象**：箭头函数没有自己的`arguments`对象，但是可以通过剩余参数（`...args`）来访问函数的参数。
- **不能用作构造函数**：由于没有自己的`this`绑定，箭头函数不能用作构造函数。
- **没有`prototype`属性**：箭头函数没有`prototype`属性。

### 应用场景

箭头函数最适用于非方法函数和那些不能被当作构造函数使用的场景，非常适合用于简短的回调或函数式编程场景，比如`map`、`filter`、`reduce`等数组方法的参数，或者任何需要简洁语法的地方。

## 对比匿名函数

**箭头函数** 提供了一种更简洁的函数写法。通常用于单行表达式，但也可以用于更复杂的场景。箭头函数没有自己的 `this`, `arguments`, `super`, 或 `new.target` 绑定。这些值由外围最近一层非箭头函数决定。

```
const arrowFunction = (arg1, arg2) => arg1 + arg2;
```

如果箭头函数体包含了一个表达式，那么该表达式的结果会被自动返回。对于更复杂的逻辑，可以使用花括号 `{}` 来包围函数体，并且需要显式返回一个值（如果需要的话）。

**匿名函数** 是一种没有名称的函数表达式，通常用于传递给其他函数作为参数，或者定义立即执行的函数表达式（IIFE）。匿名函数可以有自己的 `this`, `arguments`, `super`, 和 `new.target` 绑定。

```
const anonymousFunction = function(arg1, arg2) {
  return arg1 + arg2;
};
```

### `this` 绑定的差异

**箭头函数** 没有自己的 `this` 值。箭头函数内的 `this` 值由外围最近一层非箭头函数的执行上下文决定。这使得箭头函数非常适合用作回调函数，尤其是在需要对 `this` 绑定进行词法跟踪的场合。

```
function Counter() {
  this.count = 0;
  setInterval(() => {
    this.count++; // `this` 与外围函数的 `this` 相同
    console.log(this.count);
  }, 1000);
}
```

**匿名函数** 有自己的 `this` 绑定。如果匿名函数被一个对象调用，`this` 将指向那个对象。在全局作用域中调用时，`this` 会指向全局对象（在非严格模式下）或者是 `undefined`（在严格模式下）。

```
function Counter() {
  this.count = 0;
  setInterval(function() {
    this.count++; // `this` 不指向 Counter 实例，除非显式绑定
    console.log(this.count);
  }, 1000);
}
```

### 用途和选择

- **箭头函数** 是首选的回调函数形式，特别是当需要在回调函数中访问外围函数的 `this` 上下文时。它们的简洁语法也使得代码更加清晰和简洁。
- **匿名函数** 更适合那些需要独立的 `this` 绑定的场合，或者是在需要将函数作为构造函数时（箭头函数不能用作构造函数）。
