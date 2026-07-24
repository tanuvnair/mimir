---
title: "Event Handling"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
publish: false
---

# Event Handling

React uses **synthetic events** - a cross-browser wrapper around the browser's native event system. This ensures events work consistently across all browsers.

## Click Events

The most common event in React. Use `onClick` to handle clicks on buttons, divs, and other elements.

```jsx
// Event handler function
const handleClick = () => {
  setClickCount(clickCount + 1);
};

// Attach to button
<button onClick={handleClick}>
  Click Me!
</button>
```

## Mouse Events

React supports various mouse events: `onMouseMove`, `onMouseEnter`, `onMouseLeave`, etc.

```jsx
const handleMouseMove = (e: MouseEvent<HTMLDivElement>) => {
  setMousePosition({ x: e.clientX, y: e.clientY });
};

<div onMouseMove={handleMouseMove}>
  Move your mouse here!
</div>
```

## Form Events

Forms are central to web apps. Handle `onChange` for inputs and `onSubmit` for forms.

```jsx
// Handle input changes
const handleInputChange = (e: ChangeEvent<HTMLInputElement>) => {
  const { name, value } = e.target;
  setFormData(prev => ({ ...prev, [name]: value }));
};

// Handle form submission
const handleSubmit = (e: FormEvent<HTMLFormElement>) => {
  e.preventDefault(); // Prevent page reload!
  console.log(formData);
};

<form onSubmit={handleSubmit}>
  <input
    name="name"
    value={formData.name}
    onChange={handleInputChange}
  />
</form>
```

## Keyboard Events

Handle keyboard input with `onKeyDown`, `onKeyUp`, and `onKeyPress`.

## The Event Object

Event handlers receive a **synthetic event object** with useful properties:

### Common Event Properties

- `e.target` - The element that triggered the event
- `e.currentTarget` - The element the handler is attached to
- `e.preventDefault()` - Prevent default behavior (e.g., form submission)
- `e.stopPropagation()` - Stop event from bubbling up
- `e.key`, `e.clientX`, `e.clientY` - Event-specific data

## Event Handler Patterns

### Don't Call Immediately:

```jsx
// This calls the function immediately!
<button onClick={handleClick()}>
  Wrong
</button>
```

### Pass Function Reference:

```jsx
// This passes the function to be called later
<button onClick={handleClick}>
  Correct
</button>
```

### Inline Arrow Function:

```jsx
// Use when you need to pass arguments
<button onClick={() => handleClick(id)}>
  With Args
</button>
```

### With Event Object:

```jsx
// Access event object
<button onClick={(e) => {
  console.log(e.target);
}}>
  With Event
</button>
```

## Under the Hood: Event Delegation

React doesn't attach event listeners to every single element. Instead, it uses **Event Delegation**.

### How It Works

React attaches a SINGLE event listener for each event type (click, change, etc.) to the **root** of your app (usually the div you mounted React into).

- When you click a button, the browser's native event bubbles up to the root.
- React catches it at the root.
- React figures out which component fired it.
- React creates a "Synthetic Event" wrapper.
- React calls your `onClick` handler.

This is why `e.stopPropagation()` works in React even if you stop propagation to parent DOM nodes - React handles its own bubbling logic!

## Key Takeaways

- React uses synthetic events for cross-browser compatibility
- Common events: `onClick`, `onChange`, `onSubmit`, `onMouseMove`
- Always use `e.preventDefault()` in form submit handlers to prevent page reload
- Pass function references, not function calls, to event handlers
- Use arrow functions when you need to pass arguments to handlers
- Event handlers receive a synthetic event object with useful properties

## Related

- Series: [[react-katas]]
- Prev: [[state-basics]]
- Next: [[conditional-rendering]]
