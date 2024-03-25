---
title: "SVG 与 Canvas"
date: 2023-03-29T14:38:32+08:00
draft: false
---

SVG（Scalable Vector Graphics）和Canvas是两种主要的 Web 技术，用于在网页上绘制图形，但它们在使用方式、性能和适用场景上有所不同。

### SVG

SVG是一种使用XML描述2D图形的语言。SVG图形是矢量的，这意味着它们可以在不失真的情况下缩放到任何尺寸。SVG适合用于那些图形细节丰富、结构复杂、但不频繁重绘的应用场景。

SVG在React中的使用非常直观，因为SVG元素可以直接在JSX中声明，就像其他任何React元素一样。这意味着可以很容易地将SVG图形集成到我们的组件中，还可以利用React的状态和属性来动态更新SVG的属性，从而实现丰富的交互效果，例如：

```jsx
function MyComponent() {
  const [color, setColor] = React.useState('blue');

  return (
    <svg width="100" height="100" onClick={() => setColor('red')}>
      <circle cx="50" cy="50" r="40" stroke="black" strokeWidth="3" fill={color} />
    </svg>
  );
}
```

在这个示例中，一个圆形的颜色会在点击后从蓝色变为红色。React的状态和事件处理使得与SVG的交互变得简单。

**优点**：

- **矢量图形**：放大不失真，适合需要高品质图形输出的场景。
- **DOM接口**：SVG元素是DOM元素，可以通过JavaScript和CSS轻松控制。
- **搜索引擎优化**（SEO）：由于SVG图形是文本文件，搜索引擎可以索引它们。

**缺点**：

- **性能问题**：在图形非常复杂或大量使用时，可能会导致性能问题，尤其是在动画和交互方面。
- **兼容性**：虽然大多数现代浏览器都支持SVG，但在旧浏览器上可能需要额外的兼容性处理。

### Canvas

Canvas是一个HTML元素，用于通过JavaScript来绘制图形（包括线条、形状、图像甚至视频）。与SVG不同，Canvas提供的是像素级操作，主要用于图形密集型的游戏、图像处理和实时视频处理等领域。

相比SVG，Canvas在React中的使用稍微复杂一些，因为Canvas是通过命令式的API进行绘图的。通常，需要在组件的生命周期方法中或使用`useEffect`钩子来处理Canvas的绘图逻辑，例如：

```jsx
function MyCanvasComponent() {
  const canvasRef = React.useRef(null);

  React.useEffect(() => {
    const canvas = canvasRef.current;
    const context = canvas.getContext('2d');

    // 绘制蓝色矩形
    context.fillStyle = 'blue';
    context.fillRect(10, 10, 150, 100);
  }, []); // 依赖数组为空，表示仅在组件挂载时执行

  return <canvas ref={canvasRef} width="170" height="120"></canvas>;
}
```

**优点**：

- **性能**：对于大量动画和图形密集的应用，Canvas通常能提供更好的性能，因为它是基于像素绘制的。
- **灵活性**：Canvas提供了低级API，开发者可以绘制复杂的2D和3D图形。

**缺点**：

- **不是矢量图形**：Canvas绘制的是位图，如果放大，会出现像素化。
- **不易于访问和SEO**：Canvas绘制的内容不是DOM元素，不被搜索引擎索引，也不易于进行无障碍访问。
- **需要更多代码**：相比SVG，绘制复杂图形可能需要更多的JavaScript代码。

### 选择

选择SVG还是Canvas，主要取决于应用场景：

- 如果我们的应用需要复杂的图形，如图表、图形编辑器或需要搜索引擎优化（SEO），SVG可能是更好的选择。
- 如果我们的应用需要大量的动画，如游戏、图像编辑或其他需要密集图形处理的场景，Canvas可能更合适。
