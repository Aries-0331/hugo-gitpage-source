---
title: "Go Basic - Interface as Parameter to Function"
date: 2023-03-01T17:50:27+08:00
draft: false
tags: [go, gpt]
---

> https://stackoverflow.com/questions/20314604/go-syntax-and-interface-as-parameter-to-function

Go uses interfaces for generalization of types. So if you want a function that takes a specific interface you write

```golang
func MyFunction(t SomeInterface) {...}
```

Every type that satisfies `SomeInterface` can be passed to `MyFunction`.

Now, `SomeInterface` can look like this:

```golang
type SomeInterface interface {
    SomeFunction()
}
```

To satisfy `SomeInterface`, the type implementing it must implement `SomeFunction()`.

If you, however, require an empty interface (`interface{}`) the object does not need to implement any method to be passed to the function:

```golang
func MyFunction(t interface{}) { ... }
```

This function above will take every type as [all types implement the empty interface](http://golang.org/ref/spec#Interface_types).

### Getting the type back

Now that you can have every possible type, the question is how to get the type back that was actually put in before. The empty interface does not provide any methods, thus you can't call anything on such a value.

For this you need type assertions: let the runtime check whether there is type X in value Y and convert it to that type if so.

Example:

```golang
func MyFunction(t interface{}) {
    v, ok := t.(SomeConcreteType)
    // ...
}
```

In this example the input parameter `t` is asserted to be of type `SomeConcreteType`. If `t` is in fact of type `SomeConcreteType`, `v` will hold the instance of that type and `ok` will be true. Otherwise, `ok` will be false. See [the spec on type assertions](http://golang.org/ref/spec#Type_assertions) for details.
