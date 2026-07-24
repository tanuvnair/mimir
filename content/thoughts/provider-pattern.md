---
title: "Provider Pattern"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
  - hooks
  - performance
publish: false
---

# Provider Pattern Optimization

Context providers can cause performance issues if not optimized. Learn how to prevent unnecessary re-renders when using Context API.

![[provider-pattern-optimization.png]]

## The Problem with Context

When context value changes, ALL consumers re-render, even if they don't use the changed value.

### Bad Pattern:

```jsx
function App() {
    const [theme, setTheme] = useState('light');
    const [user, setUser] = useState(null);

    // New object on every render!
    const value = { theme, setTheme, user, setUser };

    return (
        <ThemeContext.Provider value={value}>
	        <Component /> {/* Re-renders on ANY state change! */}
        </ThemeContext.Provider>
    );
}
```

## Solution 1: Split Contexts

Separate state and updater functions into different contexts.

### Good Pattern:

```jsx
// Split into two contexts
const ThemeContext = createContext(null);
const ThemeUpdateContext = createContext(null);

function ThemeProvider({ children }) {
	const [theme, setTheme] = useState('light');

	return (
		<ThemeContext.Provider value={theme}>
			<ThemeUpdateContext.Provider value={setTheme}>
				{children}
			</ThemeUpdateContext.Provider>
		</ThemeContext.Provider>
	);
}

// Components only re-render if they use the changing value
function Display() {
	const theme = useContext(ThemeContext); // Re-renders on theme change
	return <div>{theme}</div>;
}

function Toggle() {
	const setTheme = useContext(ThemeUpdateContext); // Never re-renders!
	return <button onClick={() => setTheme('dark')}>Toggle</button>;
}
```

## Solution 2: Memoize Context Value

Use `useMemo` to prevent creating new objects on every render.

```jsx
function Provider({ children }) {
    const [state, setState] = useState(initialState);

    // Memoize the value object
    const value = useMemo(() => ({
        state,
        setState
    }), [state]);

	return (
		<Context.Provider value={value}>
			{children}
		</Context.Provider>
	);
}
```

## Solution 3: Use Composition

Pass expensive components as `children` to avoid re-renders.

```jsx
function App() {
	return (
		<ThemeProvider>
			<ExpensiveComponent /> {/* Doesn't re-render on theme change! */}
		</ThemeProvider>
	);
}

function ThemeProvider({ children }) {
	const [theme, setTheme] = useState('light');
	
	return (
		<ThemeContext.Provider value={theme}>
			<div className={theme}>
				{children} {/* Children don't re-render */}
			</div>
		</ThemeContext.Provider>
	);
}
```

## Best Practices

### Optimization Tips:

- Split state and updaters into separate contexts
- Memoize context values with useMemo
- Use composition to prevent re-renders
- Keep context values small and focused
- Consider state management libraries for complex state

## Key Takeaways

- Context updates cause all consumers to re-render
- Split state and updaters into separate contexts
- Memoize context values with `useMemo`
- Use composition to prevent unnecessary re-renders
- Keep context values small and focused
- Profile before optimizing - context may not be the bottleneck!

## Related

- Series: [[react-katas]]
- Prev: [[code-splitting]]
- Next: [[profiling-and-debugging]]
