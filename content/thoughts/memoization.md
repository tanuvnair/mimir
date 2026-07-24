---
title: "Memoization"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
  - hooks
  - performance
publish: false
---

# Memoization

In React 19, the **React Compiler** automatically optimizes most components. Manual memoization with `useMemo` and `useCallback` is now optional in many cases!

## React 19 & The Compiler

React 19 introduces an automatic compiler that memoizes components and values behind the scenes. This means:

### What's Automatic:

- Component re-renders are optimized automatically
- Simple calculations are memoized by the compiler
- Object and array references are stable when possible
- Most cases don't need manual memoization!

### When You Still Need Manual Memoization:

- Expensive computations (heavy calculations, large data processing)
- Referential equality for dependencies (useEffect, custom hooks)
- Preventing unnecessary child re-renders in large lists
- When profiling shows a performance issue

## useMemo - Memoizing Values

`useMemo` caches the result of a calculation between re-renders. Use it for expensive computations.

```jsx
function ExpensiveCalculation() {
  const [count, setCount] = useState(0);
  const [input, setInput] = useState('');

  // Without useMemo: runs on every render
  // const result = expensiveOperation(count);

  // With useMemo: only runs when count changes
  const result = useMemo(() => {
    console.log('Computing...');
    return expensiveOperation(count);
  }, [count]); // Only recompute when count changes

  return (
    <>
      <p>Result: {result}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
      <input value={input} onChange={(e) => setInput(e.target.value)} />
    </>
  );
}
```

## useCallback - Memoizing Functions

`useCallback` returns a memoized callback function. Useful when passing callbacks to optimized child components.

```jsx
function Parent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');

  // Without useCallback: new function on every render
  // const handleClick = () => setCount(c => c + 1);

  // With useCallback: same function reference
  const handleClick = useCallback(() => {
    setCount(c => c + 1);
  }, []); // Empty deps - function never changes

  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <MemoizedChild onClick={handleClick} />
    </>
  );
}

// Child only re-renders if onClick changes
const MemoizedChild = memo(({ onClick }) => {
  console.log('Child rendered');
  return <button onClick={onClick}>Click</button>;
});
```

## React.memo - Memoizing Components

`React.memo` prevents re-renders if props haven't changed. Less needed in React 19, but still useful for expensive components.

```jsx
// Without memo: re-renders on every parent render
function ExpensiveChild({ value }) {
  console.log('Rendering ExpensiveChild');
  // ... expensive rendering logic
  return <div>{value}</div>;
}

// With memo: only re-renders if value changes
const ExpensiveChild = memo(function ExpensiveChild({ value }) {
  console.log('Rendering ExpensiveChild');
  // ... expensive rendering logic
  return <div>{value}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [other, setOther] = useState(0);

  return (
    <>
      <button onClick={() => setOther(o => o + 1)}>
        Update Other
      </button>
      <ExpensiveChild value={count} />
      {/* Child doesn't re-render when 'other' changes */}
    </>
  );
}
```

## When NOT to Use Memoization

### Don't Memoize These:

- Simple calculations (addition, string concatenation)
- Components that always re-render anyway
- Premature optimization without profiling
- Small lists (under 100 items)
- When the memoization cost exceeds the benefit

## The Modern React 19 Approach

### Best Practice in React 19:

1. **Start without memoization** - Let the compiler optimize
2. **Profile your app** - Use React DevTools Profiler
3. **Identify bottlenecks** - Find actual performance issues
4. **Add memoization selectively** - Only where needed
5. **Focus on composition** - Better architecture beats memoization

## Key Takeaways

- React 19 compiler automatically optimizes most cases
- `useMemo` for expensive calculations, `useCallback` for stable function references
- `React.memo` for preventing unnecessary component re-renders
- Don't memoize prematurely - profile first!
- Component composition is often better than memoization
- In React 19, memoization is an optimization, not a requirement
- Next section: Performance patterns that work better than memoization!

## Related

- Series: [[react-katas]]
- Prev: [[use-reducer-hook]]
- Next: [[behavioral-hooks]]
- Also: [[react-memo]], [[component-composition]]
