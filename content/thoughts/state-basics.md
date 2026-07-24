---
title: "State Basics"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
publish: false
---

# State Basics

State lets components "remember" information and respond to user interactions. The `useState` hook is how we add state to function components.

## What is State?

State is data that changes over time. When state changes, React automatically re-renders the component to reflect the new data.

```jsx
// useState returns [currentValue, setterFunction]
const [count, setCount] = useState(0);

// Update state by calling the setter
<button onClick={() => setCount(count + 1)}>
  Increment
</button>
```

## Multiple State Variables

You can use multiple useState hooks in a single component for different pieces of state.

## State vs Props

| State                            | Props                       |
| -------------------------------- | --------------------------- |
| Owned by the component           | Passed from parent          |
| Can be changed by the component  | Read-only (immutable)       |
| Triggers re-renders when updated | Can be passed down          |
| Private and local                | Configuration for component |

## Conditional Rendering with State

State is perfect for controlling what gets rendered

## Under the Hood: State Batching

React doesn't update state immediately. Instead, it "batches" updates together for performance. This means multiple state updates in the same event handler might result in only ONE re-render!

### How Batching Works

```jsx
// Render Count: 0

const handleClick = () => {
  setCount(c => c + 1); // Update queued
  setName('Alice');     // Update queued
  setIsVisible(false);  // Update queued

  // React reconciles ALL changes in one go!
  // Component re-renders ONCE with all new values.
}
```

## Key Takeaways

- Use `useState` to add state to function components
- State changes trigger component re-renders
- You can have multiple state variables in one component
- State is local and private to the component
- Props are read-only, state is mutable
- Always use the setter function to update state, never mutate directly

## Related

- Series: [[react-katas]]
- Prev: [[components-and-props]]
- Next: [[event-handling]]
