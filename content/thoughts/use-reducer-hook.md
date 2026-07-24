---
title: "useReducer Hook"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
  - hooks
publish: false
---

# useReducer Hook

`useReducer` is an alternative to `useState` for managing complex state logic. It's especially useful when state updates depend on previous state or when you have multiple sub-values.

## What is useReducer?

`useReducer` accepts a reducer function and initial state, returning the current state and a dispatch function.

```jsx
const [state, dispatch] = useReducer(reducer, initialState);

// Reducer function
function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    default:
      return state;
  }
}

// Dispatch an action
dispatch({ type: 'INCREMENT' });
```

When to Use useReducer:

- Complex state logic with multiple sub-values
- Next state depends on previous state
- Want to centralize state update logic
- Need to optimize performance with deep updates

## Example 1: Simple Counter

A basic example showing the reducer pattern.

```jsx
type Action =
  | { type: 'INCREMENT' }
  | { type: 'DECREMENT' }
  | { type: 'RESET' };

interface State {
  count: number;
}

function reducer(state: State, action: Action): State {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'RESET':
      return { count: 0 };
    default:
      return state;
  }
}

function Counter() {
  const [state, dispatch] = useReducer(reducer, { count: 0 });

  return (
    <>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
      <button onClick={() => dispatch({ type: 'RESET' })}>Reset</button>
    </>
  );
}
```

## Example 2: Form State Management

Managing multiple form fields with a single reducer.

```jsx
type Action =
  | { type: 'SET_FIELD'; field: string; value: string }
  | { type: 'RESET' };

interface FormState {
  name: string;
  email: string;
  message: string;
}

function formReducer(state: FormState, action: Action): FormState {
  switch (action.type) {
    case 'SET_FIELD':
      return { ...state, [action.field]: action.value };
    case 'RESET':
      return { name: '', email: '', message: '' };
    default:
      return state;
  }
}

function Form() {
  const [state, dispatch] = useReducer(formReducer, {
    name: '',
    email: '',
    message: '',
  });

  return (
    <input
      value={state.name}
      onChange={(e) => dispatch({
        type: 'SET_FIELD',
        field: 'name',
        value: e.target.value
      })}
    />
  );
}
```

## Example 3: Todo List

Complex state with arrays - adding, removing, and toggling items.

```jsx
interface Todo {
  id: number;
  text: string;
  completed: boolean;
}

type Action =
  | { type: 'ADD_TODO'; text: string }
  | { type: 'TOGGLE_TODO'; id: number }
  | { type: 'DELETE_TODO'; id: number };

function todoReducer(state: Todo[], action: Action): Todo[] {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, {
        id: Date.now(),
        text: action.text,
        completed: false
      }];
    case 'TOGGLE_TODO':
      return state.map(todo =>
        todo.id === action.id
          ? { ...todo, completed: !todo.completed }
          : todo
      );
    case 'DELETE_TODO':
      return state.filter(todo => todo.id !== action.id);
    default:
      return state;
  }
}
```

## useReducer vs useState

### useState

Best for:

- Simple state (strings, numbers, booleans)
- Independent state updates
- State that doesn't depend on previous state
- Quick prototyping

```jsx
const [count, setCount] = useState(0);
setCount(c => c + 1);
```

### useReducer

Best for:

- Complex state objects
- Multiple sub-values
- State transitions with logic
- Centralized state updates

```jsx
const [state, dispatch] = useReducer(reducer, init);
dispatch({ type: 'INCREMENT' });
```

## Best Practices

### Do's

- Use TypeScript to type actions and state
- Keep reducers pure (no side effects)
- Use action types as constants
- Return new state objects (don't mutate)
- Handle default case in switch

### Don'ts

- Don't mutate state directly
- Don't perform side effects in reducers
- Don't use for simple state (overkill)
- Don't forget the default case

## Key Takeaways

- `useReducer` is great for complex state logic
- Reducers are pure functions that take state and action, return new state
- Actions describe what happened, reducers describe how state changes
- Use `useState` for simple state, `useReducer` for complex state
- Reducers make state updates predictable and testable
- TypeScript makes reducers safer with action type checking

## Related

- Series: [[react-katas]]
- Prev: [[custom-hooks]]
- Next: [[memoization]]
