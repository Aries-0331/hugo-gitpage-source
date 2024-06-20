---
title: "Dom"
date: 2021-11-25T14:14:09+08:00
lastmod: 2024-03-25T14:14:09+08:00
draft: false
categories: [HTML]
---

DOM（文档对象模型 Document Object Model）是一个跨平台和语言独立的接口，允许程序和脚本动态地访问和更新文档的内容、结构和样式。DOM将HTML和XML文档呈现为树结构，其中每个节点代表文档中的一部分，例如文本、注释、元素等。

### 核心概念

- **节点（Node）**：DOM将所有文档元素表示为节点树。文档中的每个部分都是一个节点，包括元素节点、文本节点、属性节点等。
- **元素（Element）**：特定类型的节点，代表文档中的HTML或XML元素。元素节点可以有属性节点和子节点。
- **属性（Attribute）**：定义在元素上的附加信息，如`class`、`id`、`style`等。
- **文本（Text）**：元素或属性中的文本内容，被视为文本节点。

### DOM操作

1. **查找元素**：
   - `document.getElementById(id)`：通过元素ID查找元素。
   - `document.getElementsByTagName(name)`：通过标签名查找元素。
   - `document.getElementsByClassName(name)`：通过类名查找元素。
   - `document.querySelector(selector)`：返回文档中匹配指定CSS选择器的第一个元素。
   - `document.querySelectorAll(selector)`：返回文档中匹配指定CSS选择器的所有元素的NodeList。
2. **修改文档结构**：
   - `createElement(tagName)`：创建一个新的元素节点。
   - `createTextNode(data)`：创建一个新的文本节点。
   - `appendChild(child)`：将一个节点添加到指定父节点的子节点列表末尾。
   - `removeChild(child)`：从DOM中删除一个子节点。
   - `replaceChild(newChild, oldChild)`：替换子节点。
3. **操作属性和样式**：
   - 通过`element.style.property`直接修改样式。
   - 使用`element.setAttribute(name, value)`和`element.getAttribute(name)`操作属性。
4. **事件处理**：
   - `element.addEventListener(event, function, useCapture)`：为元素添加事件监听器。
   - `element.removeEventListener(event, function, useCapture)`：移除事件监听器。

### 注意事项

- **性能考虑**：频繁地操作DOM可以导致页面性能问题，因为每次修改DOM都可能引发页面的重排（reflow）和重绘（repaint）。为了优化性能，可以减少DOM操作的次数，比如使用`documentFragment`或合并多次DOM操作。
- **跨浏览器兼容性**：虽然现代浏览器对DOM的支持趋于一致，但仍然存在一些兼容性问题。在开发过程中，需要注意这些差异，或使用库（如jQuery）来抹平这些差异。
- **安全性**：在处理DOM时，应当注意防止跨站脚本攻击（XSS），特别是在插入用户输入内容到DOM时。应该对用户输入进行适当的清理和转义。

### 虚拟DOM

虚拟DOM是对真实DOM的抽象，主要用于前端框架中，以提高应用性能。虚拟DOM允许我们在内存中进行DOM操作，通过比较两个虚拟DOM树的差异，只更新真实DOM树中需要变更的部分，从而避免了不必要的DOM操作。
