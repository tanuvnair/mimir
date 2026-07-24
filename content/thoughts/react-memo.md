---
title: "React.memo"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
  - hooks
  - performance
publish: false
---

# React.memo

`React.memo` is a higher-order component that prevents re-renders when props haven't changed. In React 19, it's less critical due to automatic optimizations, but still useful for expensive components.

## What is React.memo?

`React.memo` memoizes a component, re-rendering only when props change.

```jsx
// Without memo
function ExpensiveComponent({ data }) {
    return <div>{data}</div>;
}

// With memo
const ExpensiveComponent = memo(function ExpensiveComponent({ data }) {
    return <div>{data}</div>;
});

// Component only re-renders if 'data' prop changes
```

When to Use React.memo:

- Component renders often with same props
- Expensive rendering logic
- Large lists with many items
- After profiling shows it helps

## Custom Comparison Function

Provide a custom comparison function for complex prop comparisons.

```jsx
const MemoizedComponent = memo(
    function Component({ user, settings }) {
        return <div>{user.name}</div>;
    },
    (prevProps, nextProps) => {
        // Return true if props are equal (skip re-render)
        // Return false if props changed (re-render)
        return prevProps.user.id === nextProps.user.id;
    }
);
```

## Common Pitfalls

### Breaks memo:

```jsx
// New object/array every render
<MemoComponent data={{ value: 1 }} />
<MemoComponent items={[1, 2, 3]} />
<MemoComponent onClick={() => {}} />

// These create new references, breaking memo!
```

### Works with memo:

```jsx
// Stable references
const data = useMemo(() => ({ value: 1 }), []);
const items = useMemo(() => [1, 2, 3], []);
const handleClick = useCallback(() => {}, []);

<MemoComponent data={data} />
<MemoComponent items={items} />
<MemoComponent onClick={handleClick} />
```

## Under the Hood: Shallow Comparison

React.memo uses **shallow comparison** (`Object.is`) to check if props have changed. It does NOT dig deep into objects.

### How Shallow Compare Actually Works

```jsx
// Simplified implementation of what React does:
function shallowEqual(objA, objB) {
	if (Object.is(objA, objB)) return true;
	
	if (typeof objA !== 'object' || objA === null ||
		typeof objB !== 'object' || objB === null) {
		return false;
	}
	
	const keysA = Object.keys(objA);
	const keysB = Object.keys(objB);
	
	if (keysA.length !== keysB.length) return false;
	
	// Only checks if the values of the keys are strictly equal
	for (let i = 0; i < keysA.length; i++) {
		if (!Object.prototype.hasOwnProperty.call(objB, keysA[i]) ||
			!Object.is(objA[keysA[i]], objB[keysA[i]])) {
		return false;
		}
	}
	
	return true;
}
```

This is why `{a: 1} === {a: 1}` is false in JavaScript, and why `memo` re-renders if you pass a new object!

## Key Takeaways

- `React.memo` prevents re-renders when props are unchanged
- Use for expensive components that render often
- Shallow comparison by default - compares prop references
- Custom comparison for complex props
- Breaks with new object/array/function references
- Combine with useMemo/useCallback for stable props
- In React 19, try composition first!

## Related

- Series: [[react-katas]]
- Prev: [[component-composition]]
- Next: [[code-splitting]]
- Also: [[memoization]], [[element-vs-component]]
