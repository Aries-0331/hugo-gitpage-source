---
title: "Object.is in JavaScript"
date: 2024-04-03T17:56:05+08:00
draft: false
categories: [javascript, frontend]
---

`Object.is` is a method introduced in ECMAScript 2015 (ES6) that determines whether two values are the same value. It's a more precise comparison than the traditional `==` (loose equality) or `===` (strict equality) operators in JavaScript. `Object.is` aims to provide an accurate comparison algorithm, particularly useful for distinguishing between values like `+0` and `-0`, and for correctly identifying `NaN` values, which traditional comparisons cannot.

### Syntax

```javascript
Object.is(value1, value2);
```

- `value1`: The first value to compare.
- `value2`: The second value to compare.
- Returns: A Boolean indicating whether the two arguments are the same value.

### Key Differences from `==` and `===`

- **`NaN` comparison**: `Object.is(NaN, NaN)` returns `true`, whereas `NaN === NaN` returns `false`. This is useful because `NaN` is the only JavaScript value that is not equal to itself when using traditional comparison operators.
- **`+0` and `-0` comparison**: `Object.is(+0, -0)` returns `false`, whereas `+0 === -0` returns `true`. This distinction is important in certain mathematical contexts.

### Examples

```javascript
Object.is('foo', 'foo');     // true
Object.is(window, window);   // true
Object.is('foo', 'bar');     // false
Object.is([], []);           // false, because they are distinct objects

Object.is(null, null);       // true
Object.is(0, -0);            // false
Object.is(NaN, 0/0);         // true, both represent NaN
Object.is(NaN, NaN);         // true

// Loose and strict equality comparisons for contrast
NaN === NaN;                 // false
+0 === -0;                   // true
```

### When to Use `Object.is` vs. `===`

- Use `Object.is` when you need to distinguish between `+0` and `-0`, or when you need to check if a value is `NaN`.
- Use `===` for most other comparisons, as it is the standard equality operator and covers the majority of use cases without the peculiarities handled by `Object.is`.

`Object.is` is particularly useful in functional programming and mathematical computations where these distinctions are significant. However, for the majority of equality checks, the strict equality operator `===` remains the most appropriate and efficient choice.
