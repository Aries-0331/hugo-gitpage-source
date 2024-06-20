---
title: "CSS Units"
date: 2023-06-19T11:07:43+08:00
draft: false
categories: [CSS]
---

# Recommendations

| Media  | Recommended | Occasional use     | Infrequent use             | Not recommended        |
| ------ | ----------- | ------------------ | -------------------------- | ---------------------- |
| Screen | em, rem, %  | px                 | ch, ex, vw, vh, vmin, vmax | cm, mm, in, pt, pc     |
| Print  | em, rem, %  | cm, mm, in, pt, pc | ch, ex                     | px, vw, vh, vmin, vmax |

# Relative Units

## rem

The `rem` unit in CSS stands for "root `em`". It is a relative unit of measurement that is relative to the font size of the root element. One `rem` is equal to the font size of the root element. The root element defaults to `16px` in many browsers, so `1rem` is equal to `16px`.

For example:

```js
<div class="container">
  <h1>Best Practices for CSS units</h1>
  <p>This is a paragraph with font size set to 2rem</p>
</div>
```

```css
html {
    font-size: 16px;
}
.container {
   margin: 20px;
   padding: 20px;
   border: 1px solid #ddd;
}
h1 {
    font-size: 2rem;
    color: #0077cc;
}
p {
    font-size: 1rem;
    color: #0077cc;
}
```

The `h1` element is set to `2rem` which means that it's two times the root element, the html element. 2 x `16px` = 32, so the `h1` element is `32px`.

By using `rem` units, we can easily scale the font size of the body elements proportionally to the HTML font size.

It's a good idea to use `rem` to set font sizes because it is designed to adapt to the user's browser preferences. This helps with accessibility. It is also good for consistent scaling, as changing the HTML font size will affect the elements with rem units.

Using `rem` or `em` for padding or margin also offers advantages in terms of providing a scalable and maintainable design. When we change the font size at the top level, all `rem` values automatically gets updated and adjusts according to the base front size. `rem` or `em` should be used for margin or padding depending on if we want the element to be relative to the root element or the parent.

## em

Similar to `rem`, `em` is a relative unit of measurement. But unlike `rem`, `em` is relative to the font size of the parent element or the font-size of the nearest parent with a defined font size.

For example:

```js
<div class="container">
  <p>This is a paragraph with the font size set to 2em</p>
</div>
```

```css
body {
     font-size: 20px;
}
.container {
   margin: 20px;
   padding: 20px;
   border: 1px solid #ddd;
}
p {
    font-size: 2em;
    color: #0077cc;
}
```

The `p` element is set to `2em` which means it is 2 times the font size of the parent element. 2 x 20 is `40px` so the p element is `40px`.

If we need to scale an element to be consistent with the parent, then `em` is the right direction to go in. `em` is good for creating scalable and responsive designs.

It is recommended to use `em` for setting margin and padding. When we use `em` for margin and padding, they adjust according to the font size of the parent element. This creates a consistent design, especially when users adjust their browser's default font size. When we set margin or padding with `em`, the layouts also become more flexible and adaptable, and elements can scale in proportion.

`em` is also important for media queries to enhance responsiveness and adaptability.

## %

Percentages are relative units that render an element relative to the size of the element's parent. They serve as a percentage of their containing block and are always relative to their nearest parent.

For example:

```js
<div class="container">
    <div class="box"></div>
</div>
```

```css
.container {
  width: 80%;
  height: 80%;
  margin: auto;
  position: absolute;
  top: 0;
  bottom: 0;
  left: 0;
  right: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #000000;
}
.box {
  width: 50%;
  height: 50%;
  background-color: #0077cc;
}
```

Here, the box element is `50%` the width and height of the parent container, so it takes up half of the container.

The container is `80%` of the viewport by width and height. If we try to resize the browser, we will notice that the percentage is maintained. Similarly, the box within the container is `50%` of the container by width and height.

When we want an element to take up a certain amount of the containing block, then we should go with percentages.

Percentages can be used for positioning elements relative to their containing element. This comes in handy when creating layouts where elements need to be positioned based on a percentage of the parent.

Setting widths and heights in percentages also allows elements to scale relative to their containing element.

# Viewport Units

## vh

`vh` is a relative unit of measurement that represents the visible height area of the browser window.

For example:

```htmlembedded
 <div class="viewport-height">
.viewport-height {
    height: 100vh;
    background-color: bisque;
}
```

This code snippet shows the `viewport-height` element set t0 `100vh` so it covers all of the viewport height.

Using `vh` for height allows elements to maintain the set height of the viewport. This is particularly useful when we want an element to take up a certain percentage of the screen's height.

`vh` can also be used to set font sizes that are responsive to the viewport height. This is useful for responsive typography. When we don't want an element to be relative to the parent, we can use `vh`.

## vw

`vw` is similar to `vh` but `vw` refers to the width of the browser window. It is used to set elements based on the horizontal axis of the browser window.

For example:

```htmlembedded
  <body>
    <div class="box"></div>
  </body>
```

Here, the `box` is specified as `50vw` of the overall viewport width. As the browser shrinks and increases, watch how the box adjusts to maintain the specified width.

`vw` is particularly useful and encouraged when we want an element to take a certain width of the viewport. `vw` is also used for font sizes in media query for responsive typography.

# Absolute Units

## px

`px` is an absolute unit of measurement used to specify length and sizes. `px` is a fixed unit and should be used sparingly and with caution for responsive design.

For example:

```js
<div class="container">
  <p>This element has a fixed size using px units.</p>
</div>
```

```css
.container {
	width: 300px; 
    padding: 20px;
    background-color: #f0f0f0;
}

p {
    font-size: 16px;
    color: #333;
}
```

In this example, the size of the `<p>` text remains a certain size. It remains fixed at `16px` and doesn't change regardless of whether the size of the browser changes.

When we resize the browser, we will observe that the container doesn't scale according to the viewport, and neither does the text scale according to the container.

The main reason why using `px` is not always recommended for responsive design lies in its fixed nature. Unlike relative units, such as percentages, `em`, `rem`, and viewport units (`vw`, `vh`), `px` does not adjust based on the user's preferences or the size of the viewport.

`px` is useful when we want to specify a fixed size of an element, such as a border size or an image size.

For example:

```css
img {
    width: 200px;
    height: 150px;
}
```

# To Sum Up

To set font sizes, we'll commonly use `rem` . For width, percentages are often recommended. We can also consider using `vw` . For height, consider using `%`, `rem`, or `vh`.

For padding or margin, we'll typically use `rem` or `em` depending on our specific requirements.
