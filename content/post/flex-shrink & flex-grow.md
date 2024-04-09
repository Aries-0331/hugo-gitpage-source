---
title: "Flex Shrink & Flex Grow"
date: 2024-04-03T21:48:56+08:00
draft: false
categories: [css, frontend]
---

# flex-shrink

`flex-shrink` 是 CSS Flexbox 布局中的一个属性，它定义了当父容器的空间不足以容纳所有子项时，子项（flex项）如何收缩。简而言之，`flex-shrink` 控制了flex项在必要时的收缩比率。

### 语法

```css
.item {
  flex-shrink: <number>; /* 默认值是 1 */
}
```

- `<number>`: 是一个无单位的值，定义了flex项收缩的相对比率。默认值为`1`，表示flex项可以等比例收缩。如果设置为`0`，则表示flex项不会收缩。值越大，flex项收缩得越多。

### 工作原理

当flex容器（即具有`display: flex`或`display: inline-flex`的元素）的空间不足以放下所有子项时，`flex-shrink`属性决定了各个子项将如何根据其`flex-shrink`值相对于同一容器中其他子项的`flex-shrink`值收缩。

- 如果所有子项的`flex-shrink`值都相同，则它们将等比例收缩以适应容器空间。
- 如果某些子项的`flex-shrink`值比其他子项大，则这些子项将收缩得更多。

### 示例

假设有一个flex容器和三个子项，其空间不足以容纳所有子项：

```css
.container {
  display: flex;
}

.item1 {
  flex-shrink: 1; /* 默认值 */
}

.item2 {
  flex-shrink: 2; /* 在空间不足时，这个项会比 item1 收缩得更多 */
}

.item3 {
  flex-shrink: 0; /* 这个项不会收缩 */
}
```

在这个例子中，当容器空间不足以容纳所有子项时：
- `item1` 会收缩，因为其`flex-shrink`值为`1`。
- `item2` 会收缩得比 `item1` 更多，因为其`flex-shrink`值为`2`，表明它在空间不足时可以接受更多的收缩。
- `item3` 不会收缩，因为其`flex-shrink`值为`0`。

### 使用场景

- **等比例收缩**：当希望容器内的所有项在空间不足时等比例减少大小，可以将所有项的`flex-shrink`设置为相同的值（默认是`1`）。
- **防止某些关键内容被压缩**：在某些情况下，我们可能希望某些flex项（如包含关键信息或操作按钮的项）在容器空间不足时不被压缩，以确保用户界面的功能性和易用性。此时，可以将这些项的`flex-shrink`设置为`0`，防止它们在空间紧张时缩小。

# flex-grow

`flex-grow` 是 CSS Flexbox 布局中的一个属性，它定义了当父容器有多余的空间时，子项（flex项）如何增长（扩展）来填充那些空间。与 `flex-shrink` 控制收缩不同，`flex-grow` 关注于如何利用额外的可用空间。

### 语法

```css
.item {
  flex-grow: <number>; /* 默认值是 0 */
}
```

- `<number>`: 是一个无单位的值，定义了flex项增长的相对比率。默认值为`0`，表示flex项不会增长来填充额外的空间。如果所有子项的 `flex-grow` 值都为`0`，那么即使容器有额外的空间，子项也不会增长。

### 工作原理

当flex容器的空间大于所有子项正常所需的总空间时，`flex-grow`属性决定了各个子项将如何根据其`flex-grow`值相对于同一容器中其他子项的`flex-grow`值增长。

- 如果所有子项的`flex-grow`值都为`0`，那么它们将保持原始大小，不会增长来占据额外的空间。
- 如果某些子项的`flex-grow`值大于`0`，这些子项会增长以填充容器的剩余空间。值越大，增长的比例也越大。

### 示例

假设有一个flex容器和三个子项，容器的空间大于所有子项正常所需的总空间：

```css
.container {
  display: flex;
}

.item1 {
  flex-grow: 1;
}

.item2 {
  flex-grow: 2;
}

.item3 {
  flex-grow: 0; /* 这个项不会增长 */
}
```

在这个例子中，当容器有额外空间时：
- `item1` 会增长，因为其`flex-grow`值为`1`。
- `item2` 会比 `item1` 增长得更多，因为其`flex-grow`值为`2`，表明它可以占据更多的额外空间。
- `item3` 不会增长，因为其`flex-grow`值为`0`。

### 使用场景

- **平均分配空间**：如果希望所有项在容器有额外空间时平均分配那些空间，可以将所有项的`flex-grow`设置为相同的非零值。
- **保持原始大小**：对于不希望在容器有额外空间时增长的项，可以将其`flex-grow`设置为`0`。
- **优先增长某些项**：如果希望某些项在容器有额外空间时占据更多空间，可以给它们设置更高的`flex-grow`值。

通过合理利用`flex-grow`，可以灵活地控制flex布局中子项如何响应容器的额外空间，实现多样化的布局需求。
