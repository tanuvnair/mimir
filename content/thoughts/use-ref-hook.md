---
title: "useRef Hook"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
  - hooks
publish: false
---

# useRef Hook

The `useRef` hook lets you reference values that don't trigger re-renders when changed. It's perfect for accessing DOM elements and storing mutable values.

## What is useRef?

`useRef` returns a mutable object with a `.current` property that persists across renders.

```jsx
const ref = useRef(initialValue);

// Access the value
console.log(ref.current);

// Update the value (doesn't cause re-render!)
ref.current = newValue;
```

Key Difference:

Changing `ref.current` does NOT trigger a re-render, unlike `setState`!

## Use Case 1: Accessing DOM Elements

The most common use - directly access and manipulate DOM elements.

### Focus Input Example:

```jsx
function FocusInput() {
  const inputRef = useRef<HTMLInputElement>(null);

  const handleFocus = () => {
    // Access the DOM element directly
    inputRef.current?.focus();
  };

  return (
    <>
      <input ref={inputRef} type="text" />
      <button onClick={handleFocus}>Focus Input</button>
    </>
  );
}
```

## Use Case 2: Storing Mutable Values

Use refs to store values that change but shouldn't trigger re-renders (timers, previous values, etc.)

```jsx
function RenderCounter() {
  const [count, setCount] = useState(0);
  const renderCount = useRef(0);

  // Increment on every render (doesn't cause re-render)
  renderCount.current += 1;

  return (
    <div>
      <p>State: {count}</p>
      <p>Renders: {renderCount.current}</p>
      <button onClick={() => setCount(c => c + 1)}>
        Increment
      </button>
    </div>
  );
}
```

## Use Case 3: Tracking Previous Values

Store the previous value of state or props using refs.

```jsx
function PreviousValue() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef(0);

  useEffect(() => {
    // Update previous value after render
    prevCountRef.current = count;
  }, [count]);

  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCountRef.current}</p>
      <button onClick={() => setCount(c => c + 1)}>
        Increment
      </button>
    </div>
  );
}
```

## Use Case 4: Managing Timers

Store timer IDs in refs so you can clear them from any function.

```jsx
function Stopwatch() {
  const [time, setTime] = useState(0);
  const intervalRef = useRef<number | null>(null);

  const start = () => {
    if (intervalRef.current) return; // Already running

    intervalRef.current = setInterval(() => {
      setTime(t => t + 1);
    }, 1000);
  };

  const stop = () => {
    if (intervalRef.current) {
      clearInterval(intervalRef.current);
      intervalRef.current = null;
    }
  };

  return (
    <>
      <p>Time: {time}s</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
    </>
  );
}
```

## Use Case 5: Scrolling to Elements

Use refs to programmatically scroll to specific elements.

```jsx
function ScrollToElement() {
  const targetRef = useRef<HTMLDivElement>(null);

  const scrollToTarget = () => {
    targetRef.current?.scrollIntoView({
      behavior: 'smooth',
      block: 'center'
    });
  };

  return (
    <>
      <button onClick={scrollToTarget}>Scroll to Target</button>
      {/* ... lots of content ... */}
      <div ref={targetRef}>Target Element</div>
    </>
  );
}
```

## useRef vs useState

### useState

- Triggers re-render when updated
- Use for UI data
- Asynchronous updates
- Immutable update pattern

```jsx
const [count, setCount] = useState(0);
setCount(c => c + 1); // Re-renders
```

### useRef

- No re-render when updated
- Use for non-UI data
- Synchronous updates
- Mutable .current property

```jsx
const countRef = useRef(0);
countRef.current += 1; // No re-render
```

## Common Patterns

### When to Use useRef

- Accessing DOM elements (focus, scroll, measure)
- Storing timer/interval IDs
- Tracking previous values
- Storing mutable values that don't affect rendering
- Avoiding re-renders for internal state

## Key Takeaways

- `useRef` creates a mutable object that persists across renders
- Changing `ref.current` doesn't trigger re-renders
- Perfect for accessing DOM elements with the ref attribute
- Use refs for values that change but don't affect the UI
- Common uses: DOM access, timers, previous values, instance variables
- Use `useState` for UI data, `useRef` for everything else

## Related

- Series: [[react-katas]]
- Prev: [[use-effect-cleanup]]
- Next: [[custom-hooks]]
