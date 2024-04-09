---
title: "React Router"
date: 2022-05-06T14:31:14+08:00
draft: false
tags: [react, router, frontend]
---

React Router 是一个用于React的路由库，它允许我们在React应用中实现客户端路由。这意味着用户可以在不重新加载页面的情况下导航到不同的URL，同时应用的状态会相应更新以反映新的位置。这是单页面应用（SPA）的一个核心特性，能够提供更快的用户体验和更流畅的页面切换效果。

### 核心组件

React Router 提供了几个核心组件来定义和管理路由：

- **BrowserRouter**：一个使用HTML5 history API的Router，用于支持干净的URL。
- **HashRouter**：一个使用URL的hash部分（#）来保持UI和URL同步的Router，主要用于不支持HTML5 history API的旧版浏览器。
- **Route**：用于定义UI和路径之间的映射关系。它接受一个`path`属性和一个渲染方法（如`component`、`render`或`children`），当路径匹配时渲染相应的UI。
- **Switch**：仅渲染第一个与当前路径匹配的`<Route>`或`<Redirect>`。在包含多个`<Route>`的情况下非常有用，可以确保一次只渲染一个路由。
- **Link** 和 **NavLink**：允许在应用内导航到不同的路径。`NavLink`是`Link`的一个特殊版本，它可以为当前匹配的路由添加样式。

### 路由参数

React Router 允许通过在路径定义中使用`:`后跟参数名来指定动态片段。然后，你可以在组件中通过`match.params`属性访问这些参数。

```

<Route path="/user/:userId" component={User} />
```

在这个例子中，任何匹配`/user/:userId`形式的路径都会渲染`User`组件，其中`:userId`是一个动态参数，可以在`User`组件内通过`this.props.match.params.userId`访问。

### 编程式导航

除了使用`<Link>`和`<NavLink>`组件进行声明式导航外，React Router还提供了编程式导航的能力。这可以通过访问`history`对象来实现，它作为props传递给所有由`<Route>`渲染的组件。

```
class Home extends React.Component {
  handleClick = () => {
    this.props.history.push("/about");
  };

  render() {
    return (
      <button onClick={this.handleClick}>Go to About</button>
    );
  }
}
```

在这个例子中，点击按钮将会使用`history.push`方法导航到`/about`路径。

### 嵌套路由

React Router 允许路由嵌套，这意味着一个路由的组件内部可以包含自己的`<Route>`组件来进一步定义更深层次的路径结构。

### 重定向

`<Redirect>`组件用于路由的重定向，它可以将用户从一个路径自动导航到另一个路径。

```

<Redirect from="/old-path" to="/new-path" />
```

在这个例子中，任何访问`/old-path`的请求都会被重定向到`/new-path`。
