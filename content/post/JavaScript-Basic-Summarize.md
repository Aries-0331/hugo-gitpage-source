---
title: "JavaScript Basic Summarize"
date: 2024-01-30T15:48:57+08:00
draft: false
categories: [javascript]
---

Typically something that are easy to forget or misunderstand.

## Data types

JavaScript is a dynamic language with dynamic types. Variables in JavaScript are not directly associated with any particular value type, and any variable can be assigned (and re-assigned) values of all types.

JavaScript is also a weakly typed language, which means it allows implicit type conversion when an operation involves mismatched types, instead of throwing type errors.

Implicit coercions are very convenient, but can create subtle bugs when conversions happen where they are not expected, or where they are expected to happen in the other direction (for example, string to number instead of number to string).

### Primitive values

| Type                                                         | `typeof` return value | Object wrapper                                               |
| :----------------------------------------------------------- | :-------------------- | :----------------------------------------------------------- |
| [Null](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#null_type) | `"object"`            | N/A                                                          |
| [Undefined](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#undefined_type) | `"undefined"`         | N/A                                                          |
| [Boolean](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#boolean_type) | `"boolean"`           | [`Boolean`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Boolean) |
| [Number](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#number_type) | `"number"`            | [`Number`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Number) |
| [BigInt](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#bigint_type) | `"bigint"`            | [`BigInt`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/BigInt) |
| [String](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#string_type) | `"string"`            | [`String`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String) |
| [Symbol](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures#symbol_type) | `"symbol"`            | [`Symbol`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Symbol) |

### Differences between null and undefined

`undefined` means a variable has been declared but has not yet been assigned a value :

```javascript
var testVar;
console.log(testVar); //shows undefined
console.log(typeof testVar); //shows undefined
```

`null` is an assignment value. It can be assigned to a variable as a representation of no value :

```javascript
var testVar = null;
console.log(testVar); //shows null
console.log(typeof testVar); //shows object
```

From the preceding examples, it is clear that `undefined` and `null` are two distinct types: `undefined` is a type itself (undefined) while `null` is an object.

```javascript
console.log(null === undefined) // false (not the same type)
console.log(null == undefined) // true (but the "same value")
console.log(null === null) // true (both type and value are the same)
null = 'value' // Uncaught SyntaxError: invalid assignment left-hand side
undefined = 'value' // 'value'
```

### Differences between var/let

**Scoping rules**

The main difference is scoping rules. Variables declared by `var` keyword are scoped to the immediate function body (hence the function scope) while `let` variables are scoped to the immediate *enclosing* block denoted by `{ }` (hence the block scope).

```javascript
function run() {
  var foo = "Foo";
  let bar = "Bar";

  console.log(foo, bar); // Foo Bar

  {
    var moo = "Mooo"
    let baz = "Bazz";
    console.log(moo, baz); // Mooo Bazz
  }

  console.log(moo); // Mooo
  console.log(baz); // ReferenceError
}
```

The reason why `let` keyword was introduced to the language was function scope is confusing and was one of the main sources of bugs in JavaScript.

**Hoisting**

Variables declared with `var` keyword are hoisted and initialized which means they are accessible in their enclosing scope even before they are declared, however their value is `undefined` before the declaration statement is reached:

```javascript
function checkHoisting() {
  console.log(foo); // undefined
  var foo = "Foo";
  console.log(foo); // Foo
}
```

`let` variables are hoisted but not initialized until their definition is evaluated. Accessing them before the initialization results in a `ReferenceError`. The variable is said to be in the temporal dead zone from the start of the block until the declaration statement is processed.

```javascript
function checkHoisting() {
  console.log(foo); // ReferenceError
  let foo = "Foo";
  console.log(foo); // Foo
}
```

**Redeclaration**

In strict mode, `var` will let you re-declare the same variable in the same scope while `let` raises a SyntaxError.

```javascript
'use strict';
var foo = "foo1";
var foo = "foo2"; // No problem, 'foo1' is replaced with 'foo2'.

let bar = "bar1"; 
let bar = "bar2"; // SyntaxError: Identifier 'bar' has already been declared
```

- Creating global object property

At the top level, `let`, unlike `var`, does not create a property on the global object:

```javascript
var foo = "Foo"; // globally scoped
let bar = "Bar"; // globally scoped but not part of the global object

console.log(window.foo); // Foo
console.log(window.bar); // undefined
```

## Error Handling

Common errors in JavaScript: 

- ReferenceError

  A ReferenceError gets thrown when, for example, one tries to use variables that haven't been declared anywhere.

- SyntaxError

  Any kind of invalid JavaScript code will cause a SyntaxError.

  There's an interesting caveat regarding the SyntaxError in JavaScript: **it cannot be caught using the try-catch block.  **

- TypeError

  A TypeError is thrown when, for example, trying to run a method on a non-supported data type.

- RangeError

  A RangeError is thrown when we're giving a value to a function, but that value is out of the allowed range of acceptable input values.

Some other errors:

- AggregateError
- Error
- InternalError
- URIError

## For of && For in

`for in` loops over enumerable property names of an object.

`for of` (new in ES6) does use an object-specific iterator and loops over the values generated by that.

Both `for..of` and `for..in` statements iterate over lists; the values iterated on are different though, `for..in` returns a list of keys on the object being iterated, whereas `for..of` returns a list of values of the numeric properties of the object being iterated.

```javascript
let list = [4, 5, 6];

for (let i in list) {
   console.log(i); // "0", "1", "2",
}

for (let i of list) {
   console.log(i); // "4", "5", "6"
}
```

Another distinction is that `for..in` operates on any object; it serves as a way to inspect properties on this object. `for..of` on the other hand, is mainly interested in values of iterable objects. Built-in objects like `Map` and `Set` implement `Symbol.iterator` property allowing access to stored values.

```javascript
let pets = new Set(["Cat", "Dog", "Hamster"]);
pets["species"] = "mammals";

for (let pet in pets) {
   console.log(pet); // "species"
}

for (let pet of pets) {
    console.log(pet); // "Cat", "Dog", "Hamster"
}
```

## Data Structures

### Arrays

- `forEach()`

  The forEach() method accepts **a function that will work on each array item**. That function's first parameter is the current array item itself, and the second (optional) parameter is the index.

  ```javascript
  const fruits = ['kiwi','mango','apple','pear'];
  function appendIndex(fruit, index) {
      console.log(`${index}. ${fruit}`)
  }
  fruits.forEach(appendIndex);
  
  output:
  0. kiwi
  1. mango
  2. apple
  3. pear
  ```

  Very often, the function that the `forEach()` method needs to use is passed in directly into the method call, like this:

  ```javascript
  const veggies = ['onion', 'garlic', 'potato'];
  veggies.forEach( function(veggie, index) {
      console.log(`${index}. ${veggie}`);
  });
  ```

- `filter`

  It filters arrays **based on a specific test**. Those array items that pass the test are returned.

  ```javascript
  const nums = [0,10,20,30,40,50];
  nums.filter( function(num) {
      return num > 20;
  })
  
  output:[30,40,50]
  ```

- `map`

  This method is used to map each array item over to another array's item, based on whatever work is performed inside the function that is passed-in to the map as a parameter. 

  ```javascript
  [0,10,20,30,40,50].map( function(num) {
      return num / 10
  })
  
  output:[0,1,2,3,4,5]
  ```

### Map

To make a new Map, we can use the `Map` constructor: `new Map();`

```javascript
let bestBoxers = new Map();
bestBoxers.set(1, "The Champion");
bestBoxers.set(2, "The Runner-up");
bestBoxers.set(3, "The third place");

console.log(bestBoxers);

output:
Map(3) {1 => 'The Champion', 2 => 'The Runner-up', 3 => 'The third place'}
```

To get a specific value, we need to use the `get()` method. For example, `bestBoxers.get(1);//The Champion`

### Set

A set is a collection of **unique** values.

To build a new set, we can use the `Set` constructor: `new Set();`

The `Set` constructor can, for example, accept an array.

This means that we can use it to quickly filter an array for unique members.

```javascript
const repetitiveFruits = ['apple','pear','apple','pear','plum', 'apple'];
const uniqueFruits = new Set(repetitiveFruits);
console.log(uniqueFruits);

output:{'apple', 'pear', 'plum'}
```

### Others

- Queues
- Linked lists (singly-linked and doubly-linked)
- Trees
- Graphs

## Spread && Rest

When using spread, we are expanding a single variable into more:

```javascript
var abc = ['a', 'b', 'c'];
var def = ['d', 'e', 'f'];
var alpha = [ ...abc, ...def ];
console.log(alpha)// alpha == ['a', 'b', 'c', 'd', 'e', 'f'];
```

When using rest arguments, we are collapsing all remaining arguments of a function into one array:

```javascript
function sum( first, ...others ) {
    for ( var i = 0; i < others.length; i++ )
        first += others[i];
    return first;
}
console.log(sum(1,2,3,4))// sum(1, 2, 3, 4) == 10;
```

