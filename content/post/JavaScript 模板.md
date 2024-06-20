---
title: "JavaScript 模板"
date: 2022-12-30T11:06:43+08:00
draft: false
categories: [JavaScript]
---

在JavaScript中，模板是指结合数据来生成 HTML 标记或其他文本的技术，模板通过声明式的标记语法允许你指定如何将对象属性插入到文档中。这种方法在动态内容渲染、页面或应用部分的即时更新等方面很有用。

模板涉及两个主要部分：

1. **模板字符串**：定义了最终输出的结构和布局，其中包含了一些占位符，用于在运行时插入动态内容。
2. **数据对象**：一个JavaScript对象，包含了要插入到模板字符串中的动态内容。

ES6 引入了模板字符串（Template Strings）功能，让创建含有动态数据的字符串变得更加简单。模板字符串使用反引号标识，并允许嵌入表达式`${expression}`。

```js
const name = 'World';
const greeting = `Hello, ${name}!`;
console.log(greeting); // Hello, World!
```

模板字符串简化了动态字符串的创建，尤其是在生成HTML时：

```js
const user = { name: 'John', age: 30 };
const userInfo = `
  <div>
    <h1>${user.name}</h1>
    <p>Age: ${user.age}</p>
  </div>
`;
```

对于更复杂的模板和数据绑定，

通常会使用模板引擎（如Handlebars、Mustache、EJS等）。模板引擎提供了更丰富的文本替换功能，包括条件判断、循环、过滤器等。这里以 Handlebars 为例，借助它我们可以创建具有逻辑表达式的模板。

```js
<script id="user-template" type="text/x-handlebars-template">
  <div>
    <h1>{{name}}</h1>
    <p>Age: {{age}}</p>
  </div>
</script>
```

然后用 Handlebars 编译模板，并传入一个数据对象来渲染它：

```js
const source = document.getElementById("user-template").innerHTML;
const template = Handlebars.compile(source);

const context = { name: "John", age: 30 };
const html = template(context);

document.body.innerHTML = html;
```

使用模板时，应尽可能保持简单，不建议在模板中加入过多逻辑。对于使用模板引擎的情况，可以通过预编译模板来提高页面渲染性能。另外，要对动态插入到模板中的用户输入进行适当转义，以防 XSS 攻击。
