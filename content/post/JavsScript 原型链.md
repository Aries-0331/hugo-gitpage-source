---
title: "JavsScript 原型链"
date: 2022-10-24T20:36:26+08:00
draft: false
tags: [javascript, prototype]
---

每个 JavaScript 对象都有一个指向其他对象的链接，这个链接就是我们所说的“原型”。当我们试图访问一个对象的属性或方法时，如果它自身没有这个属性或方法，JavaScript 就会沿着这条原型链向上查找，直到找到为止，这就是原型链。

初次接触这一概念时，很容易想到 C++ 之类 OOP 语言中的继承，对于这类语言严格区分类和实例，继承实际上是类的扩展。但是 JavaScript 的面向对象编程方式和其它大多语言都不太一样，JavaScript 不区分类和实例的概念，而是通过原型（prototype）来实现面向对象编程。

每个 JavaScript 对象都有一个内置的 `__proto__` 属性（非标准，但广泛支持），指向创建它的构造函数的原型对象 `prototype`。使用构造函数创建一个对象时，新对象的 `__proto__` 会指向构造函数的 `prototype` 属性，这是原型链的基础。

```js
function Person(name) {
  this.name = name;
}

Person.prototype.sayName = function() {
  console.log(this.name);
};

const person1 = new Person('Alice');
person1.sayName(); // Alice
console.log(person1.__proto__ === Person.prototype); // true
```

在这个例子中，`Person.prototype` 是 `person1` 的原型。当我们调用 `person1.sayName()` 时，虽然 `person1` 自己没有 `sayName` 方法，但它会通过原型链在 `Person.prototype` 中找到这个方法。

原型链的一个重要应用是实现对象之间的继承。

```js
function Employee(name, jobTitle) {
  Person.call(this, name); // 调用 Person 构造函数
  this.jobTitle = jobTitle;
}

// 继承 Person
Employee.prototype = Object.create(Person.prototype);
Employee.prototype.constructor = Employee;

Employee.prototype.sayJob = function() {
  console.log(this.jobTitle);
};

const employee1 = new Employee('Bob', 'Developer');
employee1.sayName(); // Bob
employee1.sayJob(); // Developer
```

这个示例中实现了使用 `Employee` 从 `Person` 的继承，将 `Employee.prototype` 设置为 `Person.prototype` 的一个实例，这样 `Employee` 的实例就可以访问 `Person` 原型上的方法。

总的来说，JavaScript 的原型链主要用于实现继承、多态、给对象添加方法、实现一些性能优化（比如通过原型共享方法、减少每个实例的内存占用）等。

另外，需要注意避免链过长和原型污染。链过长可能导致性能问题，而多个实例共享同一个原型时，对原型对象的修改会影响所有通过这个原型创建的对象，可能会导致意想不到的问题，需谨慎使用。

对于 `__proto__` 属性来说，它允许对一个对象的原型进行读/写访问。这是一个非常强大的特性，因为它让开发者可以直接操作对象的原型链。不过，正因为它这么强大，直接操作`__proto__`也可能引发代码的可维护性和性能问题，这也是为什么它不被包含在ECMAScript 2015（ES6）及以后版本的标准中的原因之一。

尽管如此，考虑到现有代码库中广泛使用了`__proto__`，以及它在一些特定情况下的用途，大多数现代JavaScript环境仍然支持它。但是，官方推荐使用`Object.getPrototypeOf()`和`Object.setPrototypeOf()`这两个方法来代替直接访问`__proto__`属性，因为这两个方法是标准中定义的，因此使用它们编写的代码会更加可靠和兼容。

---

1. https://zh.wikipedia.org/wiki/%E5%9F%BA%E4%BA%8E%E5%8E%9F%E5%9E%8B%E7%BC%96%E7%A8%8B

2. https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Inheritance_and_the_prototype_chain#%E5%9F%BA%E4%BA%8E%E5%8E%9F%E5%9E%8B%E9%93%BE%E7%9A%84%E7%BB%A7%E6%89%BF

3. https://www.liaoxuefeng.com/wiki/1022910821149312/1023021997355072
