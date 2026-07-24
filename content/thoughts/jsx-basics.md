---
title: "JSX Basics"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
publish: false
---

# JSX Basics

## What is JSX?

- JSX is a syntax extension
- Lets you write HTML-like markup inside JavaScript

### JSX

```jsx
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```

## What React actually creates using JavaScript

```jsx
const element = React.createElement(
  'h1',
  { className: 'greeting' },
  'Hello, world!'
);
```

## Embedding Expressions in JSX

We can embed JavaScript expression in JSX by wrapping it in curly braces `{}`

```jsx
const name = 'React Developer';
const currentYear = new Date().getFullYear();
const isLearning = true;

// Using expressions in JSX
<ul>
  <li>Variable: <strong>{name}</strong></li>
  <li>Expression: <strong>{2 + 2}</strong></li>
  <li>Function call: <strong>{currentYear}</strong></li>
  <li>Conditional: <strong>{isLearning ? 'Learning!' : 'Not learning'}</strong></li>
</ul>
```

## Important JSX Rules

- Return a single root element
	- Wrap multiple elements in a parent tag or Fragment
- Close all tags
	- Even self-closing tags like `<img/>`
- Use camelCase
	- HTML attributes become camelCase (`className`, `onClick`)
- Reserved words
	- Use `className` instead of `class`, `htmlFor` instead of `for`

## React Fragments

When you need to return multiple elements without adding extra nodes to the DOM, use Fragments

```jsx
// Long syntax
<React.Fragment>
  <h1>Title</h1>
  <p>Paragraph</p>
</React.Fragment>

// Short syntax (more common)
<>
  <h1>Title</h1>
  <p>Paragraph</p>
</>
```

## Related

- Series: [[react-katas]]
- Next: [[element-vs-component]]
- Also: [[element-vs-component]]
