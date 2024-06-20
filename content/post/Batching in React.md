---
title: "Batching in React"
date: 2024-01-20T11:34:29+08:00
draft: false
categories: [React]
---

# Batching in React.

Batch updates in React are a mechanism to group multiple state updates into a single re-render for performance optimization. This helps in reducing the number of renders and thus improves performance, especially during complex state updates or when multiple updates happen in quick succession.

## Details of Batch Updates in React

1. **Concept**:
   - React batches state updates by default inside event handlers and lifecycle methods to avoid excessive re-renders.
   - When multiple state updates occur within an event handler, React schedules a single re-render to reflect all the changes at once.
2. **Mechanism**:
   - When a state update is triggered, React checks if it is currently batching updates.
   - If batching is enabled, the updates are collected and processed together at the end of the event handler.
   - If batching is not enabled (e.g., updates triggered from asynchronous callbacks like `setTimeout`), each update may result in a separate re-render.

## React Code for Batch Updates

React's implementation of batch updates involves internal functions and mechanisms that manage the update lifecycle. Let's look at some relevant parts of the React source code to understand how it works.

**Note**: React codebase is large and complex. The following snippets are simplified for explanation.

### Batch Updates in React 17 and earlier

In React 17 and earlier versions, React used a module called `ReactUpdates` for managing updates. The core of the batching mechanism is implemented in this module.

```javascript
// Simplified version from React 17 source code
let isBatchingUpdates = false;
let updateQueue = [];

function batchedUpdates(callback, a, b, c, d, e) {
	const alreadyBatchingUpdates = isBatchingUpdates;
	isBatchingUpdates = true;

	// Perform the callback (e.g., an event handler)
	try {
		return callback(a, b, c, d, e);
	} finally {
		isBatchingUpdates = alreadyBatchingUpdates;
		if (!isBatchingUpdates) {
			// Process the queued updates
			flushBatchedUpdates();
		}
	}
}

function flushBatchedUpdates() {
	// Process all updates in the queue
	while (updateQueue.length) {
		const update = updateQueue.shift();
		update.perform();
	}
}
```

In this simplified version:

- `isBatchingUpdates` flag determines if React is currently batching updates.
- `batchedUpdates` function enables batching by setting `isBatchingUpdates` to true, running the callback, and then flushing the queued updates.
- `flushBatchedUpdates` processes the collected updates at the end of the batching period.

### Batch Updates in React 18 and later

React 18 introduces a more sophisticated concurrency model with the new `ReactDOM.createRoot` API, which improves the ability to batch updates, especially in concurrent mode.

```javascript
// Simplified version from React 18 source code
import { unstable_batchedUpdates } from "react-dom";

function batchedUpdates(callback, a, b, c, d, e) {
	unstable_batchedUpdates(() => {
		callback(a, b, c, d, e);
	});
}

// Usage in an event handler
function handleClick() {
	batchedUpdates(() => {
		setState1(newState1);
		setState2(newState2);
	});
}
```

In this simplified version:

- `unstable_batchedUpdates` is imported from `react-dom` and used to batch updates.
- It wraps the callback, ensuring that all state updates within the callback are batched together.

### Explanation of Batch Updates

1. **Batched Updates**:
   - React batches updates to minimize the number of re-renders.
   - It ensures that multiple state updates within the same event handler are processed together, resulting in a single re-render.
2. **Event Handlers**:
   - Inside event handlers, React batches updates automatically.
   - This prevents excessive rendering during user interactions.
3. **Asynchronous Updates**:
   - By default, updates triggered by asynchronous code (e.g., `setTimeout`, `fetch` callbacks) are not batched.
   - In React 18, the `use` function and the automatic batching feature improve handling of asynchronous updates, allowing them to be batched more effectively.

### Practical Example

Here’s an example demonstrating batched updates:

```javascript
import React, { useState } from "react";

function Counter() {
	const [count1, setCount1] = useState(0);
	const [count2, setCount2] = useState(0);

	function handleClick() {
		setCount1(count1 + 1);
		setCount2(count2 + 1);
		// Both updates are batched, resulting in a single re-render
	}

	return (
		<div>
			<p>Count 1: {count1}</p>
			<p>Count 2: {count2}</p>
			<button onClick={handleClick}>Increment</button>
		</div>
	);
}

export default Counter;
```

In this example, clicking the button will increment both `count1` and `count2`. Despite the two state updates, React will batch them together, resulting in a single re-render of the `Counter` component.

## Automatic Batching in React 18

**Automatic batching** means that React will group multiple state updates into a single re-render, even if they occur in different contexts, such as asynchronous callbacks. This is a departure from earlier versions of React, where batching was primarily confined to event handlers.

Consider the following example where state updates are triggered within a `setTimeout`:

### React 17 Behavior

In React 17, each state update within an asynchronous callback would cause a separate re-render:

```javascript
import React, { useState } from "react";

function Counter() {
	const [count1, setCount1] = useState(0);
	const [count2, setCount2] = useState(0);

	function handleAsyncUpdate() {
		setTimeout(() => {
			setCount1((c) => c + 1); // Causes a re-render
			setCount2((c) => c + 1); // Causes another re-render
		}, 1000);
	}

	return (
		<div>
			<p>Count 1: {count1}</p>
			<p>Count 2: {count2}</p>
			<button onClick={handleAsyncUpdate}>Increment Async</button>
		</div>
	);
}

export default Counter;
```

### React 18 Behavior

In React 18, the same example will result in a single re-render due to automatic batching:

```javascript
import React, { useState } from "react";
import ReactDOM from "react-dom/client";

function Counter() {
	const [count1, setCount1] = useState(0);
	const [count2, setCount2] = useState(0);

	function handleAsyncUpdate() {
		setTimeout(() => {
			setCount1((c) => c + 1); // Batched with the next update
			setCount2((c) => c + 1); // Batched with the previous update
		}, 1000);
	}

	return (
		<div>
			<p>Count 1: {count1}</p>
			<p>Count 2: {count2}</p>
			<button onClick={handleAsyncUpdate}>Increment Async</button>
		</div>
	);
}

// Create a root to enable concurrent features and automatic batching
const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<Counter />);
```

In this example, even though the state updates happen within a `setTimeout`, React 18 batches them together and performs only one re-render after both state updates are applied.

### Benefits of Automatic Batching

1. **Improved Performance**:
   - By reducing the number of re-renders, automatic batching significantly improves the performance of applications, especially those with frequent state updates.
2. **Consistency**:
   - Developers do not need to worry about whether updates are happening synchronously or asynchronously. React handles batching consistently across different contexts.
3. **Simpler Code**:
   - Automatic batching simplifies the mental model for developers. You no longer need to use special APIs or workarounds to ensure updates are batched in asynchronous code.

## Additional Enhancements in React 18

Alongside automatic batching, React 18 introduces other concurrent features that improve the user experience and performance:

1. **Concurrent Rendering**:
   - React 18 allows rendering to be interrupted and resumed, making it possible to handle complex updates without blocking the main thread.
2. **Transition API**:
   - The new `startTransition` API lets developers mark updates as non-urgent, allowing React to prioritize important updates while deferring less critical ones.

## Conclusion

Automatic batching in React 18 is a powerful feature that enhances performance by minimizing unnecessary re-renders. It extends batching behavior to asynchronous contexts, making state management more efficient and consistent.
