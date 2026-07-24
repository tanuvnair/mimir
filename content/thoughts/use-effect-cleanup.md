---
title: "useEffect Cleanup"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
  - hooks
publish: false
---

# useEffect Cleanup

## Why Cleanup Matters

Without proper cleanup, you can have:

- **Memory leaks:** Timers and listeners that keep running
- **Stale closures:** Effects using old state values
- **Race conditions:** Multiple async operations conflicting
- **Performance issues:** Unnecessary computations

**Critical Rule:** If your effect sets up something (timer, subscription, listener), it MUST clean it up!

## Pattern 1: Cleaning Up Timers

Always clear intervals and timeouts to prevent them from running after the component unmounts.

```jsx
function TimerComponent() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    // Set up the interval
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);

    // Cleanup: clear the interval
    return () => {
      clearInterval(interval);
      console.log('Timer cleaned up!');
    };
  }, []);

  return <div>Seconds: {seconds}</div>;
}
```

## Pattern 2: Cleaning Up Event Listeners

Remove event listeners to prevent memory leaks, especially with window/document events.

```jsx
function MouseTracker() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    // Define the handler
    const handleMouseMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };

    // Add listener
    window.addEventListener('mousemove', handleMouseMove);

    // Cleanup: remove listener
    return () => {
      window.removeEventListener('mousemove', handleMouseMove);
      console.log('Mouse listener removed!');
    };
  }, []);

  return <div>Mouse: {position.x}, {position.y}</div>;
}
```

## Pattern 3: Cleaning Up Subscriptions

WebSocket connections, Firebase listeners, and other subscriptions must be cleaned up.

```jsx
function ChatComponent() {
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    // Simulate WebSocket connection
    const ws = new WebSocket('wss://chat.example.com');

    ws.onmessage = (event) => {
      setMessages(prev => [...prev, event.data]);
    };

    // Cleanup: close connection
    return () => {
      ws.close();
      console.log('WebSocket closed!');
    };
  }, []);

  return <div>Messages: {messages.length}</div>;
}
```

## Pattern 4: Canceling Async Operations

Use `AbortController` to cancel fetch requests and prevent state updates on unmounted components.

```jsx
function SearchResults({ searchTerm }) {
  const [results, setResults] = useState([]);

  useEffect(() => {
    // Create abort controller
    const controller = new AbortController();

    async function search() {
      try {
        const response = await fetch(
          `/api/search?q=${searchTerm}`,
          { signal: controller.signal } // Pass signal
        );
        const data = await response.json();
        setResults(data);
      } catch (error) {
        if (error.name === 'AbortError') {
          console.log('Fetch aborted');
        }
      }
    }

    if (searchTerm) search();

    // Cleanup: abort the fetch
    return () => {
      controller.abort();
    };
  }, [searchTerm]);

  return <div>Results: {results.length}</div>;
}
```

## Pattern 5: Debouncing with Cleanup

Use cleanup to implement debouncing - delay execution until user stops typing.

**Immediate value:** (empty)

**Debounced value (500ms delay):** (empty)

The debounced value updates 500ms after you stop typing

```jsx
function DebouncedSearch() {
  const [input, setInput] = useState('');
  const [debouncedValue, setDebouncedValue] = useState('');

  useEffect(() => {
    // Set up timeout
    const timer = setTimeout(() => {
      setDebouncedValue(input);
    }, 500); // Wait 500ms after user stops typing

    // Cleanup: clear timeout if input changes again
    return () => {
      clearTimeout(timer);
    };
  }, [input]); // Re-run when input changes

  return (
    <input
      value={input}
      onChange={(e) => setInput(e.target.value)}
    />
  );
}
```

## Common Cleanup Pitfalls

### Wrong

```jsx
useEffect(() => {
  setInterval(() => {
    console.log('tick');
  }, 1000);
  // No cleanup!
}, []);
```

Interval keeps running forever!

### Correct

```jsx
useEffect(() => {
  const id = setInterval(() => {
    console.log('tick');
  }, 1000);

  return () => clearInterval(id);
}, []);
```

Cleanup clears the interval!

## Key Takeaways

- Cleanup functions run before the next effect and on unmount
- Always clean up: timers (`clearInterval`, `clearTimeout`)
- Always clean up: event listeners (`removeEventListener`)
- Always clean up: subscriptions (WebSocket `close()`, unsubscribe functions)
- Use `AbortController` to cancel fetch requests
- Cleanup enables debouncing and other timing patterns
- Forgetting cleanup leads to memory leaks and bugs!

## Related

- Series: [[react-katas]]
- Prev: [[use-effect-fundamentals]]
- Next: [[use-ref-hook]]
- Also: [[custom-hooks]]
