+++
author = "Jeff Chang"
title = "Tips for invoke child component function and force rerendering"
date = "2023-09-23"
description = "Have you ever face an issue in React where you wanted to invoke an inner function from the child component for rerendering? In this artcle, we will be going through some real world example on how the issue happened and solution for it."
tags = [
    "react"
]
categories = [
    "React"
]
image = "cover.jpg"
+++

## Table of contents

- [Introduction](#introduction)
- [React Keys](#react-keys)
- [useImperativeHandle hook](#useimperativehandle)

### Introduction<a name="introduction"></a>

Say we have 2 components in our React application. One is the Parent component while the other is a Child component, and the parent component is supposed to pass in a prop for initial value to the Child component.

The Child component will then takes that prop value as a initial value for an [useState](https://react.dev/reference/react/useState) hook.

For example, we have a counter value and setCounter function from a `useState` hook that takes a prop value from Parent component as a initial counter value.

```jsx
// Parent component
import { useState } from "react";
import "./App.css";
import Child from "./Child";

function App() {
  const [initialCount, setInitialCount] = useState(5);
  return (
    <>
      <div>
        <Child initialCount={initialCount} />
        <button onClick={() => setInitialCount(0)}>Parent reset</button>
      </div>
    </>
  );
}

export default App;
```

```jsx
// Child component
import { useState } from "react";

const childCounterDisplay = {
  display: "flex",
  alignItems: "center",
};

export default function Child({ initialCount }) {
  console.log("initialCount", initialCount);
  const [count, setCount] = useState(initialCount);

  const reset = () => {
    setCount(0);
  };

  return (
    <div>
      <div>
        Child counter: <span>{count}</span>
      </div>
      <div style={childCounterDisplay}>
        <button onClick={() => setCount((prevCount) => prevCount + 1)}>
          +
        </button>
        <button onClick={reset}>Reset</button>
      </div>
    </div>
  );
}
```

<iframe src="https://codesandbox.io/embed/1-tips-for-invoke-child-component-function-and-rerendering-f3pwgs?expanddevtools=1&fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="(1) Tips for invoke child component function and rerendering"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

#### Explaination:

We first pass in the initial value as 5, the increment function works fine in the Child component **but** if we click on the **Parent reset** button, the counter value will not set to 0 even the `initialValue` have changed from 5 to 0 in the console

The **most common way** is to move the useState hook from Child component out to the parent component. For example:

```jsx
// Parent component
import { useState } from "react";
import "./App.css";
import Child from "./Child";

function App() {
  const [count, setCount] = useState(5);
  return (
    <>
      <div>
        <Child count={count} setCount={setCount} />
        <button onClick={() => setCount(0)}>Parent reset</button>
      </div>
    </>
  );
}

export default App;
```

```jsx
// Child component
const childCounterDisplay = {
  display: "flex",
  alignItems: "center",
};

export default function Child({ count, setCount }) {
  return (
    <div>
      <div>
        Child counter: <span>{count}</span>
      </div>
      <div style={childCounterDisplay}>
        <button onClick={() => setCount((prevCount) => prevCount + 1)}>
          +
        </button>
        <button onClick={() => setCount(0)}>Reset</button>
      </div>
    </div>
  );
}
```

<iframe src="https://codesandbox.io/embed/2-tips-for-invoke-child-component-function-and-rerendering-757wzf?expanddevtools=1&fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="(2) Tips for invoke child component function and rerendering"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

> What if today's the Child component is imported from a library where you can't really move the useState hook out of this component?

### React Keys<a name="react-keys"></a>

React Keys are used in React to identify which items in the list are changed, updated, or deleted. In most cases, it is being used as an identifier when we are creating lists of elements. However, we could also utilize this field to actually "tell" React to rerender the selected component.

```jsx
// Parent component
import { useState } from "react";
import "./App.css";
import Child from "./Child";

function App() {
  const [childKey, setChildKey] = useState(0);
  return (
    <>
      <div>
        <Child initialCount={0} key={childKey} />
        <button onClick={() => setChildKey((prevKey) => prevKey + 1)}>
          Parent reset
        </button>
      </div>
    </>
  );
}

export default App;
```

<iframe src="https://codesandbox.io/embed/3-tips-for-invoke-child-component-function-and-rerendering-5pvdw4?expanddevtools=1&fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="(3) Tips for invoke child component function and rerendering"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>

### useimperativehandle hook<a name="useimperativehandle"></a>

Alternatively, we can utilize the [useImperativeHandle](https://react.dev/reference/react/useImperativeHandle) hook by passing a React ref as a props and configure a reset function to be callable from the Parent component.

```jsx
// Parent component
import { useRef } from "react";
import "./styles.css";
import { Child } from "./Child";

function App() {
  const childRef = useRef(null);
  return (
    <>
      <div>
        <Child initialCount={0} ref={childRef} />
        <button onClick={() => childRef.current.reset()}>Parent reset</button>
      </div>
    </>
  );
}

export default App;
```

```jsx
// Child component
/* eslint-disable react/display-name */
import { forwardRef, useState, useImperativeHandle } from "react";

const childCounterDisplay = {
  display: "flex",
  alignItems: "center",
};

export const Child = forwardRef(({ initialCount }, forwardedRef) => {
  const [count, setCount] = useState(initialCount);
  useImperativeHandle(
    forwardedRef,
    () => {
      return {
        reset: () => {
          console.log("reset function invoked");
          setCount(0);
        },
      };
    },
    []
  );

  return (
    <div ref={forwardedRef}>
      <div>
        Child counter: <span>{count}</span>
      </div>
      <div style={childCounterDisplay}>
        <button onClick={() => setCount((prevCount) => prevCount + 1)}>
          +
        </button>
        <button onClick={() => setCount(0)}>Reset</button>
      </div>
    </div>
  );
});
```

<iframe src="https://codesandbox.io/embed/4-tips-for-invoke-child-component-function-and-rerendering-5mtx6x?expanddevtools=1&fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="(4) tips for invoke child component function and rerendering"
     allow="accelerometer; ambient-light-sensor; camera; encrypted-media; geolocation; gyroscope; hid; microphone; midi; payment; usb; vr; xr-spatial-tracking"
     sandbox="allow-forms allow-modals allow-popups allow-presentation allow-same-origin allow-scripts"
   ></iframe>
