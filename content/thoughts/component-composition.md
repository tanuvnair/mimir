---
title: Component Composition
date: 2026-01-07
tags:
  - react-katas
  - react
  - learning
  - performance
publish: true
---

# Component Composition

Component composition is the **most powerful performance optimization** in React. It's better than memoization and works automatically with React 19's compiler!

## The Problem: Unnecessary Re-renders

When a parent component's state changes, all its children re-render by default, even if they don't use that state.

### Bad Pattern

```jsx
function App() {
    const [count, setCount] = useState(0);

    return (
        <div>
        <button onClick={() => setCount(c => c + 1)}>
            Count: {count}
        </button>
        <ExpensiveComponent /> {/* Re-renders on every count change! */}
        </div>
    );
    }
```

Problem: ExpensiveComponent re-renders even though it doesn't use count!

## Solution 1: Lift Content Up (Children Prop)

Move state down to the component that needs it, and pass expensive components as `children`.

```jsx
function App() {
    return (
        <Layout>
        <ExpensiveComponent /> {/* Doesn't re-render! */}
        </Layout>
    );
    }

    function Layout({ children }) {
    const [count, setCount] = useState(0);

    return (
        <div>
        <button onClick={() => setCount(c => c + 1)}>
            Count: {count}
        </button>
        {children} {/* Children don't re-render when count changes! */}
        </div>
    );
    }
```

Children are created by the parent, so they don't re-render when Layout's state changes!

## Solution 2: Slot Pattern (Multiple Props)

Pass multiple components as props to create flexible, performant layouts.

```jsx
function Dashboard({ sidebar, header, content }) {
    const [isOpen, setIsOpen] = useState(false);

    return (
        <div>
        <button onClick={() => setIsOpen(!isOpen)}>Toggle</button>
        {header}    {/* Doesn't re-render */}
        {sidebar}   {/* Doesn't re-render */}
        {content}   {/* Doesn't re-render */}
        </div>
    );
    }

    // Usage
    <Dashboard
    header={<Header />}
    sidebar={<Sidebar />}
    content={<Content />}
    />
```

## Solution 3: State Co-location

Keep state as close as possible to where it's used. Don't lift state up unless you need to!

```jsx
// Bad: State too high
    function App() {
    const [formData, setFormData] = useState({});
    return (
        <>
        <Form data={formData} onChange={setFormData} />
        <OtherComponent /> {/* Re-renders on every form change! */}
        </>
    );
    }

    // Good: State colocated
    function App() {
    return (
        <>
        <Form /> {/* State is inside Form */}
        <OtherComponent /> {/* Never re-renders! */}
        </>
    );
    }

    function Form() {
    const [formData, setFormData] = useState({});
    // ... form logic
    }
```

## Solution 4: Extract Stateful Components

Extract the part that changes into its own component to isolate re-renders.

```jsx
// Bad: Everything re-renders
    function Page() {
    const [count, setCount] = useState(0);
    return (
        <>
        <button onClick={() => setCount(c => c + 1)}>
            {count}
        </button>
        <ExpensiveList /> {/* Re-renders! */}
        <ExpensiveChart /> {/* Re-renders! */}
        </>
    );
    }

    // Good: Extract counter
    function Page() {
    return (
        <>
        <Counter /> {/* Only this re-renders */}
        <ExpensiveList />
        <ExpensiveChart />
        </>
    );
    }

    function Counter() {
    const [count, setCount] = useState(0);
    return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
    }
```

## Why Composition Beats Memoization

### Composition

- Works automatically
- No extra code needed
- Better architecture
- More maintainable
- React 19 compiler optimizes it

### Memoization

- Requires manual work
- More code to maintain
- Can be misused
- Dependency tracking
- Use as last resort

## Preventing re-renders with extraction

Whenever state changes, the parent re-renders and so do its children.

![[react-re-render.png]]

```javascript
const Component = () => {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <Something>
      {isOpen && <ModalDialog />}
      <Button onClick={() => setIsOpen(true)}>Open Dialog</Button>
      <VerySlowComponent />
      <BunchOfSlowStuff />
    </Something>
  );
};
```

Prefer extracting state-related UI into its own component instead of wrapping everything in `React.memo`:

```javascript
const ButtonWithDialog = () => {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      {isOpen && <ModalDialog />}
      <Button onClick={() => setIsOpen(true)}>Open Dialog</Button>
    </>
  );
};

const Component = () => {
  return (
    <Something>
      <ButtonWithDialog />
      <VerySlowComponent />
      <BunchOfSlowStuff />
    </Something>
  );
};
```

## Key Takeaways

- **Composition is the primary performance pattern** - use it first!
- Use `children` prop to prevent unnecessary re-renders
- Slot pattern for multiple component props
- Keep state colocated - as close to where it's used as possible
- Extract stateful components to isolate re-renders
- Composition works automatically with React 19 compiler
- Better architecture beats memoization every time!

%% ## Related

- Series: [[react-katas]]
- Prev: [[render-timing-and-reconciliation]]
- Next: [[react-memo]]
- Also: [[memoization]], [[components-and-props]] %%
