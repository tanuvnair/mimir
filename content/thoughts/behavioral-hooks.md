---
title: "Behavioral Hooks"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
  - hooks
publish: false
---

# Behavioral Hooks

Behavioral hooks encapsulate complex DOM interactions and event handling into reusable functions. They isolate browser-level concerns from your component logic, keeping components clean and focused on rendering.

![[behavioral-hooks-logic-diagram.png]]

## What are Behavioral Hooks?

Behavioral hooks are custom hooks that manage complex interactions with the DOM and browser APIs. Instead of scattering event listeners and refs across your components, you encapsulate them in a hook with a clean, declarative API.

### Common Behavioral Hooks:

- **useClickOutside** -- Detect clicks outside a ref element
- **useMediaQuery** -- Reactive CSS media query matching
- **useKeyboardShortcut** -- Listen for specific key combos
- **useFocusTrap** -- Keep focus within a container (modals)
- **useIntersectionObserver** -- Detect when elements enter the viewport

```jsx
// The pattern: wrap DOM interactions in a hook
function useBehavior(ref, options) {
  useEffect(() => {
    // 1. Set up event listeners / observers
    const handler = (event) => { /* ... */ }
    document.addEventListener('event', handler)

    // 2. Clean up on unmount or dependency change
    return () => document.removeEventListener('event', handler)
  }, [ref, ...dependencies])
}
```

## Example 1: useClickOutside

Detect clicks that happen outside a specified element. Essential for dropdowns, modals, tooltips, and any UI that should close when clicking away.

```jsx
function useClickOutside<T extends HTMLElement>(
  ref: RefObject<T | null>,
  handler: () => void
) {
  useEffect(() => {
	    function handleClick(event: MouseEvent) {
	      // Check if click target is outside the ref element
	      if (ref.current && !ref.current.contains(event.target as Node)) {
	        handler()
	      }
	    }

    // Use 'mousedown' instead of 'click' for better UX
    // (fires before 'click', preventing race conditions)
    document.addEventListener('mousedown', handleClick)
    return () => document.removeEventListener('mousedown', handleClick)
  }, [ref, handler])
}

// Usage
function Dropdown() {
  const [isOpen, setIsOpen] = useState(false)
  const ref = useRef<HTMLDivElement>(null)

  const close = useCallback(() => setIsOpen(false), [])
  useClickOutside(ref, close)

  return (
    <div ref={ref}>
      <button onClick={() => setIsOpen(o => !o)}>Menu</button>
      {isOpen && <DropdownMenu />}
    </div>
  )
}
```

### Implementation Details:

- Use `mousedown` instead of `click` to fire before focus changes
- `ref.current.contains(event.target)` checks if the click was inside the element
- The handler should be stable (use `useCallback`) to avoid re-attaching listeners
- Always clean up the event listener in the useEffect return

## Example 2: useMediaQuery

Subscribe to CSS media query changes reactively. Useful for responsive logic that cannot be achieved with CSS alone, such as conditionally rendering entirely different component trees.

```jsx
function useMediaQuery(query: string): boolean {
  const [matches, setMatches] = useState(() => {
    return window.matchMedia(query).matches
  })

  useEffect(() => {
    const mql = window.matchMedia(query)

    const handleChange = (e: MediaQueryListEvent) => {
      setMatches(e.matches)
    }

    // Use addEventListener (addListener is deprecated)
    mql.addEventListener('change', handleChange)
    // Sync in case it changed between render and effect
    setMatches(mql.matches)

    return () => mql.removeEventListener('change', handleChange)
  }, [query])

  return matches
}

// Usage
function Layout() {
  const isMobile = useMediaQuery('(max-width: 640px)')
  const prefersDark = useMediaQuery('(prefers-color-scheme: dark)')

  if (isMobile) {
    return <MobileLayout />
  }
  return <DesktopLayout theme={prefersDark ? 'dark' : 'light'} />
}
```

### When to Use useMediaQuery vs CSS:

Use CSS media queries for:

- Showing/hiding elements
- Changing layout and spacing
- Adjusting font sizes

Use useMediaQuery for:

- Rendering different component trees
- Changing behavior (not just styles)
- Loading different data based on screen size

## Example 3: useKeyboardShortcut

Listen for specific keyboard combinations. Useful for command palettes, navigation shortcuts, and accessibility enhancements.

```jsx
function useKeyboardShortcut(
  key: string,
  callback: () => void,
  modifiers: {
    ctrl?: boolean
    shift?: boolean
    alt?: boolean
    meta?: boolean
  } = {}
) {
  useEffect(() => {
    function handleKeyDown(event: KeyboardEvent) {
      const { ctrl = false, shift = false, alt = false, meta = false } = modifiers

      if (
        event.key.toLowerCase() === key.toLowerCase() &&
        event.ctrlKey === ctrl &&
        event.shiftKey === shift &&
        event.altKey === alt &&
        event.metaKey === meta
      ) {
        event.preventDefault()
        callback()
      }
    }

    document.addEventListener('keydown', handleKeyDown)
    return () => document.removeEventListener('keydown', handleKeyDown)
  }, [key, callback, modifiers])
}

// Usage
function App() {
  const [searchOpen, setSearchOpen] = useState(false)

  const toggle = useCallback(() => setSearchOpen(p => !p), [])
  const close = useCallback(() => setSearchOpen(false), [])

  useKeyboardShortcut('k', toggle, { ctrl: true })
  useKeyboardShortcut('Escape', close)

  return searchOpen ? <SearchPanel /> : null
}
```

### Implementation Notes:

- Always call `event.preventDefault()` to prevent browser defaults (e.g., Ctrl+K opens browser search)
- Compare modifier keys explicitly (ctrlKey, shiftKey, altKey, metaKey)
- Use `.toLowerCase()` for case-insensitive key matching
- Ensure callback is stable with `useCallback` to avoid re-registering listeners

## Composing Behavioral Hooks

The real power of behavioral hooks is that they compose naturally. A single component can use multiple behavioral hooks without any conflicts.

```JSX
function CommandPalette() {
  const [isOpen, setIsOpen] = useState(false)
  const panelRef = useRef<HTMLDivElement>(null)

  // Compose multiple behavioral hooks
  useKeyboardShortcut('k', () => setIsOpen(true), { ctrl: true })
  useKeyboardShortcut('Escape', () => setIsOpen(false))
  useClickOutside(panelRef, () => setIsOpen(false))

  const isMobile = useMediaQuery('(max-width: 640px)')

  if (!isOpen) return null

  return (
    <div ref={panelRef} className={isMobile ? 'fullscreen' : 'centered'}>
      <SearchInput />
      <ResultsList />
    </div>
  )
}

// Each hook manages its own concern:
// - useKeyboardShortcut: opening/closing via keyboard
// - useClickOutside: closing when clicking outside
// - useMediaQuery: adapting layout to screen size
```

## Best Practices

### Do:

- Always clean up event listeners and observers in the useEffect return
- Accept refs as parameters rather than creating them internally (more flexible)
- Stabilize callbacks with `useCallback` before passing to hooks
- Handle SSR by checking `typeof window !== 'undefined'`

### Avoid:

- Creating unstable callbacks that cause constant re-subscriptions
- Forgetting to clean up listeners (memory leaks)
- Using behavioral hooks for things CSS can handle (e.g., hover states)
- Making hooks that do too many things -- keep each hook focused on one behavior

## Key Takeaways

- Behavioral hooks encapsulate complex DOM/event interactions into reusable, composable functions
- **useClickOutside**: uses `useEffect` + event listener + `ref.current.contains()`
- **useMediaQuery**: uses `useState` + `window.matchMedia` + change listener
- **useKeyboardShortcut**: uses `useEffect` + keydown listener + modifier checks
- They compose naturally -- a single component can use multiple behavioral hooks without conflicts
- Always clean up listeners, stabilize callbacks, and handle SSR gracefully

## Related

- Series: [[react-katas]]
- Prev: [[memoization]]
- Next: [[render-timing-and-reconciliation]]
