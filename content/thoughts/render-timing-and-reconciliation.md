---
title: "Render Timing and Reconciliation"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
  - performance
publish: false
---

# Render Timing & Reconciliation

Understanding when React re-renders and how reconciliation works is essential for building performant applications. This lesson covers the render triggers, the diffing algorithm, and how the `key` prop controls element identity.

## When Does React Re-render?

A React component re-renders in three situations:

### 1. State Change

When `setState` (or a dispatch from `useReducer`) is called, the component and its entire sub-tree re-render.

```jsx
const [count, setCount] = useState(0)
setCount(1) // triggers re-render of this component + children
```

### 2. Parent Re-render

When a parent component re-renders, all its children re-render too -- regardless of whether their props changed. This is the default behavior.

```jsx
function Parent() {
  const [count, setCount] = useState(0)
  return (
    <div>
      <Child /> {/* re-renders even though it has no props */}
    </div>
  )
}
```

### 3. Context Change

When a context value changes, all components consuming that context re-render -- even if they only use a portion of the context value.

```jsx
const ThemeContext = createContext('light')

function Child() {
  const theme = useContext(ThemeContext) // re-renders when value changes
  return <div className={theme}>...</div>
}
```

## What Reconciliation Does

After a component renders, React does not immediately update the DOM. Instead, it creates a new virtual element tree and **diffs** it against the previous tree. This process is called **reconciliation**.

```jsx
// Simplified reconciliation algorithm:
function reconcile(oldTree, newTree) {
  // 1. If types differ -> unmount old, mount new
  if (oldTree.type !== newTree.type) {
    unmount(oldTree)
    mount(newTree)
    return
  }

  // 2. Same type -> update props, recurse on children
  updateProps(oldTree, newTree)

  // 3. For children, use keys to match old and new children
  reconcileChildren(oldTree.children, newTree.children)
}
```

### The Two Rules:

- **Same type** = update. React keeps the DOM node and updates only the changed attributes. Component instances are preserved, state is maintained.
- **Different type** = unmount + mount. React destroys the entire subtree (including DOM nodes and state) and builds a new one from scratch.

## Demo: Same Type vs Different Type

When the element type changes (e.g., from `<div>` to `<section>`), React unmounts the entire old subtree and mounts a new one. All internal state is lost.

```jsx
// These produce DIFFERENT element types:
{condition ? <div><input /></div> : <section><input /></section>}
// Toggling condition -> unmount div subtree, mount section subtree
// Input value is LOST

// These produce the SAME element type:
{condition ? <div className="a"><input /></div> : <div className="b"><input /></div>}
// Toggling condition -> update className attribute
// Input value is PRESERVED
```

## The Key Prop: Element Identity

The `key` prop tells React which element in a list corresponds to which element from the previous render. Without keys (or with index keys), React matches elements by position. With stable unique keys, React matches by identity.

```jsx
// WITH stable keys (key={item.id}):
// React: "Apple(id=1) moved from index 0 to index 2"
// -> Reuses the same DOM nodes, preserves state (input values)

// WITHOUT stable keys (key={index}):
// React: "Item at index 0 changed from Apple to Cherry"
// -> Updates props in place, DOM state (input values) stays at position
```

## Why Index as Key is Dangerous

Using array index as key is the default behavior and is problematic when the list can be reordered, filtered, or items can be added/removed from the beginning or middle.

### Problems with index keys:

- **State mismatch:** Uncontrolled inputs, focus, scroll position, and component state get associated with the wrong items after reordering
- **Unnecessary DOM mutations:** Adding an item to the top updates every single item in the list instead of just inserting one node
- **Broken animations:** Exit/enter animations fire for the wrong items

### When index keys are OK:

- Static lists that never reorder, filter, or change
- Items have no internal state (no inputs, no local state)
- Items are never added to the beginning or middle of the list

## The Key Reset Trick

Since changing a key forces React to unmount and remount, you can use this intentionally to reset a component's internal state. This is a common and useful pattern.

```jsx
// The key reset trick:
<TimerWidget key={timerKey} />

// Changing timerKey forces unmount + remount
// All internal state (elapsed time) resets to initial
```

```JSX
// Common use cases for the key reset trick:

// 1. Reset a form when switching between records
<EditForm key={selectedUser.id} user={selectedUser} />

// 2. Reset animation when content changes
<FadeIn key={slideIndex}>
  <SlideContent index={slideIndex} />
</FadeIn>

// 3. Force re-initialization of a third-party widget
<MapWidget key={region} center={coordinates} />
```

## Demo: Render Counter

Use `useRef` to count how many times a component renders. This technique is invaluable for debugging performance issues.

```jsx
// Render counter pattern using useRef:
function MyComponent() {
  const renderCount = useRef(0)
  renderCount.current += 1

  return <div>Renders: {renderCount.current}</div>
}

// Why useRef and not useState?
// useState would cause an infinite loop:
// setState -> re-render -> setState -> re-render -> ...
// useRef.current mutation does not trigger a re-render.
```

## Key Takeaways

- React re-renders when: **state changes**, **parent re-renders**, or **context changes**
- Reconciliation diffs old and new element trees: **same type = update**, **different type = unmount + mount**
- The `key` prop tells React about element identity in lists -- use stable, unique IDs, not array indices
- Index keys cause state mismatches when lists are reordered, filtered, or modified
- The **key reset trick**: change a key to force remount and reset all internal state
- Use `useRef` to count renders for debugging (not useState, which would cause infinite loops)
- Parent re-renders cascade to all children by default -- use React.memo or composition patterns to optimize

## Related

- Series: [[react-katas]]
- Prev: [[behavioral-hooks]]
- Next: [[component-composition]]
