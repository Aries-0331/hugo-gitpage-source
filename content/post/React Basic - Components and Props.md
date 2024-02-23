---
title: "React Basic   Components and Props"
date: 2024-02-05T15:58:02+08:00
draft: true
categories: [react]
---

# Components

## What is components?

Components are one of the core concepts of React. They are the foundation upon which we build user interfaces(UI) - UI building blocks.

Taking traditional Web development as an example, HTML lets us create rich structured documents with its built-in set of tags like `<h1>` and `<li>`:

```html
<article>
  <h1>My First Component</h1>
  <ol>
    <li>Components: UI Building Blocks</li>
    <li>Defining a Component</li>
    <li>Using a Component</li>
  </ol>
</article>
```

This markup combined with CSS for style, and then JavaScript for interactivity, lies behind every sidebar, avatar, modal, dropdown -- every piece of UI we see on the Web.

However, React lets us combine our markup, CSS and JavaScript into custom "components", **reusable UI elements for app**. Just like with HTML tags, we can compose, order and nest components to design whole pages. For example, the react documentation page is make out of React components:

```react
<PageLayout>
  <NavigationHeader>
    <SearchBar />
    <Link to="/docs">Docs</Link>
  </NavigationHeader>
  <Sidebar />
  <PageContent>
    <TableOfContents />
    <DocumentationText />
  </PageContent>
</PageLayout>
```

As projects grows, we will notice that many of our designs can be composed by reusing components we already wrote, or even jumpstart a project with the thousands of components shared by the React open source community like [Chakra UI](https://chakra-ui.com/) and [Material UI](https://material-ui.com/).

React puts interactivity first: **a React component is a JavaScript function that you can sprinkle with markup**. 

**Nesting and organizing**

Components are regular JavaScript functions, so you can keep multiple components in the same file. This is convenient when components are relatively small or tightly related to each other.  We can define a component once, and then use it in as many places and as many times as we like.

**Recap**

- Components are reusable UI elements for app
- In a React app, every piece of UI is a component
- React components are regular JavaScript functions except:
  - Their names always begin with a capital letter
  - They return JSX markup

# Props

## What are Props?

Props simply stands for properties. 

They are what make components reusable. Because they perform an essential function – they pass data from one component to another.  Basically, these props components are read-only components. 

Props act as a channel for component communication. Props are passed from parent to child and help your child access properties that made it into the parent's tree.

## How to use props?

There are two ways to use props, without destructuring and with destructuring.

**Using Props With Destructuring**

```react
//MyProducts.js
import React from "react";
 
function MyProducts(props) {
  return (
    <div>
      <h1>{props.name}</h1>
      <p>{props.description}</p>
      <p>{props.price}</p>
    </div>
  );
}
 
export default MyProducts;
```

`props` is an argument in function, we can access it by writing `props.name`, `props.price`, and `props.description`.

Then we can render the product and pass some data to there three props in `App.js`. Props are passed in like HTML attributes.

```react
//App.js
import "./App.css";
import MyProducts from "./MyProducts";
function App() {
  return (
    <div className="App">
      <MyProducts
        name="temitope"
        description="the product has fantastic features"
        price={1000}
      />
    </div>
  );
}
 
export default App;
```

**Using Props With Destructuring**

Destructuring is a JavaScript feature that allows us to extract sections of data from an array or object.

```react
//traditionally
const todo = ["bath","sleep","eat"];
// old way
const firstTodo = todo[0];//bath
const secondTodo = todo[1];//sleep
 
console.log(firstTodo);
console.log(secondTodo);

//destructuring offers a easier way to do so
const todo = ["bath","sleep","eat"];
 
// destructuring
const [firstTodo, secondTodo] = todo;// bath, sleep
 
console.log(firstTodo);
console.log(secondTodo);
```

So in react, we can use destructuring to pull apart or unpack props, which lets us access props and use them more readable. Just like this:

```react
import React from "react";
 
function MyProducts({ name, description, price }) {
  return (
    <div>
      <h1>{name}</h1>
      <p>{description}</p>
      <p>{price}</p>
    </div>
  );
}
 
export default MyProducts;
```

And in this way, we can create another child component in which we can pass down the parent's attributes.

```react
import React from "react";
 
function AdditionalDescription([name, description]) {
  return (
  <div>
      <p>{name}</p>
      <p>{description}</p>
  </div>
  )
}
 
export default AdditionalDescription;
```

Then render the `AdditionalDescription` in `App.js` and pass down some props to it:

```react
<AdditionalDescription name={name} description={description} />
```

So the child can take the props of the parent.

### About prop drilling?

Prop drilling is basically a situation when the same data is being sent at almost every level due to requirements in the final level.

### Why not to use prop drilling?

1. **Code Complexity:** As components grow, prop drilling increases code complexity as it is difficult to track the flow of data through various components.
2. **Reduced Maintainability:** It will become very challenging to maintain the code with prop drilling. When changes are required in the data flow, you need to make changes in many components.
3. **Performance Overhead:** We have to pass props through unnecessary intermediary components which can impact performance.
4. **Decreased Component Reusability:** Components used in prop drilling become tightly coupled to the structure of the parent components, so it very difficult to use it on other parts of the application.
5. **Increased Development Time:** Prop drilling often requires additional planning to implement. This can slow down the development process, especially when the component hierarchies is complex.

### How to avoid prop drilling?

The problem with Prop Drilling is that whenever data from the Parent component will be needed, it would have to come from each level, Regardless of the fact that it is not needed there and simply needed in last.

A better alternative to this is using **useContext** hook. The **useContext** hook is based on Context API and works on the mechanism of Provider and Consumer. Provider needs to wrap components inside Provider Components in which data have to be consumed. Then in those components, using the **useContext** hook that data needs to be consumed.
