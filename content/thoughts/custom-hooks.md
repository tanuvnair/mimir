---
title: "Custom Hooks"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
  - hooks
publish: false
---

# Custom Hooks

Custom hooks let you extract component logic into reusable functions. They're just JavaScript functions that use other hooks!

## What are Custom Hooks?

Custom hooks are functions that start with "`use`" and can call other hooks. They let you share stateful logic between components.

### Key Rules:

- Must start with "`use`" (e.g., `useWindowSize`)
- Can call other hooks (`useState`, `useEffect`, etc.)
- Follow the same rules as built-in hooks
- Share logic, not state (each call is independent)

## Example 1: useLocalStorage

Sync state with localStorage - persist data across page reloads.

```jsx
function useLocalStorage<T>(key: string, initialValue: T) {
  // Get initial value from localStorage or use default
  const [value, setValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });

  // Update localStorage when value changes
  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error('Error saving to localStorage:', error);
    }
  }, [key, value]);

  return [value, setValue] as const;
}

// Usage
function App() {
  const [name, setName] = useLocalStorage('name', '');
  return <input value={name} onChange={(e) => setName(e.target.value)} />;
}
```

## Example 2: useWindowSize

Track window dimensions - useful for responsive behavior.

```jsx
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    };

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return size;
}

// Usage
function App() {
  const { width, height } = useWindowSize();
  return <div>Window: {width} x {height}</div>;
}
```

## Example 3: useDebounce

Delay updating a value until user stops typing - perfect for search inputs.

```jsx
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);

  useEffect(() => {
    // Set up the timeout
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    // Cleanup: clear timeout if value changes
    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]);

  return debouncedValue;
}

// Usage
function SearchBar() {
  const [search, setSearch] = useState('');
  const debouncedSearch = useDebounce(search, 500);

  useEffect(() => {
    // API call only happens after user stops typing
    if (debouncedSearch) {
      fetchResults(debouncedSearch);
    }
  }, [debouncedSearch]);

  return <input value={search} onChange={(e) => setSearch(e.target.value)} />;
}
```

## Example 4: useToggle

Simple hook for boolean state - cleaner than useState for toggles.

```jsx
function useToggle(initialValue: boolean = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => {
    setValue(v => !v);
  }, []);

  const setTrue = useCallback(() => {
    setValue(true);
  }, []);

  const setFalse = useCallback(() => {
    setValue(false);
  }, []);

  return { value, toggle, setTrue, setFalse };
}

// Usage
function App() {
  const modal = useToggle(false);

  return (
    <>
      <button onClick={modal.toggle}>Toggle Modal</button>
      {modal.value && <Modal onClose={modal.setFalse} />}
    </>
  );
}
```

## Example 5: usePrevious

Get the previous value of state or props - useful for comparisons.

```jsx
function usePrevious<T>(value: T): T | undefined {
  const ref = useRef<T>();

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
}

// Usage
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCount}</p>
      <button onClick={() => setCount(c => c + 1)}>Increment</button>
    </div>
  );
}
```

## Best Practices

### Do's

- Always start hook names with "use"
- Extract logic that's used in multiple components
- Keep hooks focused on a single responsibility
- Return values in a consistent format (array or object)
- Document your custom hooks with examples

### Don'ts

- Don't call hooks conditionally
- Don't create hooks for everything (keep it simple)
- Don't share state between hook calls (each is independent)
- Don't forget to clean up side effects

## Key Takeaways

- Custom hooks extract reusable logic from components
- Must start with "use" and can call other hooks
- Each hook call has its own isolated state
- Common patterns: localStorage, window events, debouncing, toggles
- Keep hooks focused and well-documented
- Custom hooks make your code more maintainable and testable

## Related

- Series: [[react-katas]]
- Prev: [[use-ref-hook]]
- Next: [[use-reducer-hook]]
- Also: [[behavioral-hooks]], [[use-ref-hook]]
