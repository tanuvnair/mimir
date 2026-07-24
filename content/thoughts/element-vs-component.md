---
title: "Element vs Component"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
publish: false
---

# Element vs Component

## What is a React Element?

A React element is a plain JavaScript object that describes what should appear on screen. It has a type, props, and a key. Elements are cheap to create and are immutable -- once created, you cannot change their children or attributes.

```js
// A React element is just an object like this:
{
  type: 'div',           // or a component function/class
  props: {
    className: 'card',
    children: 'Hello'
  },
  key: null,
  ref: null
}

// JSX creates elements:
const el = <div className="card">Hello</div>

// Which compiles to:
const el = React.createElement('div', { className: 'card' }, 'Hello')
```

## What is a Component?

A component is a function (or class) that accepts props and returns React elements. It is the blueprint or template for creating elements

```js
// This is a component (a function):
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>
}

// This is an element (an object created by JSX):
const element = <Greeting name="World" />

// Writing <Greeting /> does NOT call the function.
// It creates an element: { type: Greeting, props: { name: "World" } }
// React calls the function later during rendering.
```

## Key Distinction

- **Component** = the function definition (`function Greeting() {…}`)
- **Element** = the object JSX produces (`<Greeting />` becomes `{ type: Greeting, props: {} }`)
- **Instance** = what React creates internally when it renders the element

## JSX Compiles to createElement

JSX is not magic, it is syntactic sugar that compiles to React.createElement() calls. Both produce the same element object.

```js
// These are equivalent:
const a = <Greeting name="World" />
const b = createElement(Greeting, { name: "World" })

// Both produce:
// { type: Greeting, props: { name: "World" }, key: null, ref: null }

// For host elements (HTML tags):
const c = <div id="box">Hi</div>
const d = createElement('div', { id: 'box' }, 'Hi')
// { type: "div", props: { id: "box", children: "Hi" }, key: null, ref: null }
```

## Why This Matters: Element Identity and Re-rendering

React uses **element identity** (reference equality) during reconciliation. If the same element object is returned across renders, React skips reconciling that subtree. This is the principle behind patterns like storing elements in state or using `children` prop lifting.

```js
function Parent() {
  const [count, setCount] = useState(0)

  // Created once, same reference every render
  const [stored] = useState(() => <ExpensiveChild />)

  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>Re-render</button>

      {/* Same reference -> React skips reconciling */}
      {stored}

      {/* New reference each render -> React must reconcile */}
      <ExpensiveChild />
    </div>
  )
}
```

## Key Takeaways

- A **component** is a function (or class) that returns elements, it is the template.
- A **React element** is a plain object (`{ type, props, key })`, it is the description of what to render.
- JSX (`<Comp />`) compiles to `createElement(Comp, props)`, it creates an element, it does not call the function.
- React calls your component function during rendering, not at JSX evaluation time.
- Element identity matters for performance: same reference means React can skip re-reconciling that subtree.
- Never call components as functions directly -- let React manage them through elements.

## Related

- Series: [[react-katas]]
- Prev: [[jsx-basics]]
- Next: [[components-and-props]]
