---
title: "Conditional Rendering"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
publish: false
---

# Conditional Rendering

In React, you can render different UI based on conditions. There are several patterns for conditional rendering, each with its own use case.

## Pattern 1: If-Else with Variables

Use regular JavaScript if-else statements before the return to decide what to render.

```jsx
function LoginStatus({ isLoggedIn }: { isLoggedIn: boolean }) {
  // Use if-else before return
  if (isLoggedIn) {
    return <h3>Welcome back!</h3>;
  } else {
    return <h3>Please log in to continue</h3>;
  }
}
```

## Pattern 2: Ternary Operator (? :)

The most common pattern for inline conditional rendering. Use when you have two alternatives.

```jsx
// Ternary operator: condition ? true : false
{userRole === 'admin' ? (
  <p>Admin Dashboard Access</p>
) : (
  <p>Limited Access</p>
)}
```

## Pattern 3: Logical AND (&&)

Use `&&` when you only want to render something if a condition is true (no else case).

```jsx
// Only renders if condition is true
{notifications.length > 0 && (
  <div>
    You have {notifications.length} notifications!
  </div>
)}

// Common pattern for empty states
{notifications.length === 0 && (
  <p>No notifications</p>
)}
```

## Pattern 4: Switch/Case for Multiple Conditions

When you have multiple conditions, use a switch statement or object mapping.

```jsx
function LoadingStateDisplay({ state }) {
  switch (state) {
    case 'loading':
      return <Spinner />;
    case 'success':
      return <SuccessMessage />;
    case 'error':
      return <ErrorMessage />;
    default:
      return <IdleState />;
  }
}
```

## Pattern 5: Return null to Render Nothing

Components can return `null` to render nothing. This is useful for conditional components.

```jsx
function Modal({ isOpen, onClose }) {
  // Return null if not open - renders nothing
  if (!isOpen) return null;

  return (
    <div className="modal">
      <div className="modal-content">
        <button onClick={onClose}>Close</button>
      </div>
    </div>
  );
}
```

## Best Practices

### Do's and Don'ts

- Use `&&` for simple show/hide
- Use ternary for two alternatives
- Use switch/case for multiple conditions
- Avoid deeply nested ternaries (hard to read)
- Don't use `&&` with numbers (0 will render!)
- Extract complex conditions into separate components

## Key Takeaways

- Use `if-else` statements before return for complex logic
- Use ternary operator `? :` for inline two-way conditions
- Use logical AND `&&` for simple show/hide (one condition)
- Use `switch` statements or object mapping for multiple conditions
- Return `null` to render nothing
- Keep conditional rendering simple and readable
- Extract complex conditions into separate components

## Related

- Series: [[react-katas]]
- Prev: [[event-handling]]
- Next: [[use-effect-fundamentals]]
