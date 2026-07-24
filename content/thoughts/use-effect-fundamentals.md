---
title: "useEffect Fundamentals"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
  - hooks
publish: false
---

# useEffect Fundamentals

The `useEffect` hook lets you perform **side effects** in function components. Side effects include data fetching, subscriptions, timers, and manually changing the DOM.

## What are Side Effects?

Side effects are operations that affect something outside the component's scope:

- Fetching data from an API
- Setting up subscriptions or timers
- Manually updating the DOM
- Logging to console
- Setting up event listeners

**Key Concept:** Effects run _after_ React updates the DOM. This ensures your UI is always in sync before side effects execute.

## Basic Syntax

```jsx
useEffect(() => {
  // Your side effect code here
  console.log('Effect ran!');

  // Optional: return cleanup function
  return () => {
    console.log('Cleanup!');
  };
}, [dependencies]); // Dependency array
```

The dependency array controls **when** the effect runs:

- **No array:** Runs after every render
- **Empty array `[]`:** Runs only once (on mount)
- **`[dep1, dep2]`:** Runs when dependencies change

## Example 1: Updating Document Title

This effect runs after every render and updates the browser tab title with the current count.

```jsx
// Runs after EVERY render (no dependency array)
useEffect(() => {
  document.title = `Count: ${count}`;
});
```

## Example 2: Run Once on Mount

Use an empty dependency array `[]` to run an effect only once when the component mounts.

```jsx
// Runs only ONCE when component mounts
useEffect(() => {
  console.log('Component mounted!');
  // Perfect for initial data fetching
}, []); // Empty dependency array
```

## Example 3: Timer with Cleanup

Effects can return a **cleanup function** that runs before the next effect and when the component unmounts.

```jsx
useEffect(() => {
  if (isRunning) {
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);

    // Cleanup: clear interval when effect re-runs or unmounts
    return () => {
      clearInterval(interval);
    };
  }
}, [isRunning]); // Re-run when isRunning changes
```

## Example 4: Window Resize Listener

Always clean up event listeners to prevent memory leaks!

```jsx
useEffect(() => {
  const handleResize = () => {
    setWindowWidth(window.innerWidth);
  };

  // Add event listener
  window.addEventListener('resize', handleResize);

  // Cleanup: remove event listener
  return () => {
    window.removeEventListener('resize', handleResize);
  };
}, []); // Empty array - set up once on mount
```

## Example 5: Data Fetching Pattern

A common pattern: fetch data on mount or when dependencies change. We'll explore this more in the next lesson!

```jsx
useEffect(() => {
  // Fetch data when component mounts
  async function fetchData() {
    const response = await fetch('/api/user');
    const data = await response.json();
    setUser(data);
  }

  fetchData();
}, []); // Empty array - fetch once on mount
```

## Common Mistakes

### Watch Out For:

- **Missing dependencies:** Always include all values used in the effect
- **Infinite loops:** Updating state that's in the dependency array
- **Forgetting cleanup:** Always clean up timers, subscriptions, listeners
- **Async effects:** Can't make the effect function itself async

## Key Takeaways

- `useEffect` runs side effects after React updates the DOM
- Effects run after every render by default
- Use dependency array to control when effects run: `[]` for once, `[dep]` for when `dep` changes
- Return a cleanup function to clean up subscriptions, timers, and listeners
- Common uses: data fetching, subscriptions, timers, event listeners
- Always include all dependencies used in the effect

## Related

- Series: [[react-katas]]
- Prev: [[conditional-rendering]]
- Next: [[use-effect-cleanup]]
