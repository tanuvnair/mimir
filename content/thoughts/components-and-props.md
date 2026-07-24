---
title: "Components and Props"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
publish: false
---

# Components and Props

## What are Components?

```jsx
// A simple component
function Welcome() {
  return <h1>Hello, World!</h1>;
}

// Using the component
<Welcome />
```

## Props: Passing Data to Components

Props (short for "properties") let you pass data from parent to child components. Think of them as function arguments.

```jsx
// Component that accepts props
function Greeting({ name, age }: { name: string; age: number }) {
  return (
    <div>
      <h2>Hello, {name}!</h2>
      <p>You are {age} years old.</p>
    </div>
  );
}

// Using the component with props
<Greeting name="Alice" age={25} />
<Greeting name="Bob" age={30} />
```

## Props Destructuring

You can destructure props directly in the function parameters for cleaner code. This is a common pattern in modern React.

### Without Destructuring

```jsx
function Card(props) {
  return (
    <div>
      <h3>{props.title}</h3>
      <p>{props.description}</p>
    </div>
  );
}
```

### With Destructuring (Better!):

```jsx
function Card({ title, description }) {
  return (
    <div>
      <h3>{title}</h3>
      <p>{description}</p>
    </div>
  );
}
```

## The Special "children" Prop

The `children` prop is a special prop that contains whatever you put between the opening and closing tags of a component. It's fundamental to component composition.

```jsx
function Card({ title, children }: { title: string; children: ReactNode }) {
  return (
    <div className="card">
      <h3>{title}</h3>
      <div className="card-content">
        {children}
      </div>
    </div>
  );
}

// Usage - anything between tags becomes "children"
<Card title="My Card">
  <p>This is the card content!</p>
  <button>Click me</button>
</Card>
```

## Component Composition

One of React's superpowers is composing small components into larger ones. This makes your code reusable and easier to maintain.

```jsx
// Composed from smaller components
function UserProfile({ name, role, bio, skills }) {
  return (
    <div>
      <Avatar name={name} />
      <UserInfo name={name} role={role} bio={bio} />
      <SkillsList skills={skills} />
    </div>
  );
}
```

## Key Takeaways

- Components are reusable pieces of UI defined as functions that return JSX
- Props let you pass data from parent to child components
- Destructure props in function parameters for cleaner code
- The children prop enables powerful composition patterns
- Build complex UIs by composing small, focused components
- Props are read-only - never modify them inside a component

## Related

- Series: [[react-katas]]
- Prev: [[element-vs-component]]
- Next: [[state-basics]]
