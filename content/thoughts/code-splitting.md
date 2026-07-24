---
title: "Code Splitting"
date: 2026-07-24
tags:
  - react-katas
  - react
  - learning
  - performance
publish: false
---

# Code Splitting

Code splitting breaks your bundle into smaller chunks that load on demand. This reduces initial load time and improves performance.

## Why Code Splitting?

Without code splitting, users download your entire app upfront, even code they may never use.

### Benefits:

- Faster initial page load
- Load code only when needed
- Better caching (unchanged chunks stay cached)
- Improved user experience

## React.lazy() - Component Lazy Loading

Use `React.lazy()` to dynamically import components.

```jsx
import { lazy, Suspense } from 'react';

// Lazy load the component
const HeavyComponent = lazy(() => import('./HeavyComponent'));

function App() {
    return (
        <Suspense fallback={<div>Loading...</div>}>
	        <HeavyComponent />
        </Suspense>
    );
}
```

## Route-Based Code Splitting

Split code by routes - the most common pattern.

```jsx
import { lazy, Suspense } from 'react';
import { BrowserRouter, Routes, Route } from 'react-router-dom';

// Lazy load route components
const Home = lazy(() => import('./pages/Home'));
const About = lazy(() => import('./pages/About'));
const Dashboard = lazy(() => import('./pages/Dashboard'));

function App() {
	return (
		<BrowserRouter>
			<Suspense fallback={<PageLoader />}>
				<Routes>
				<Route path="/" element={<Home />} />
				<Route path="/about" element={<About />} />
				<Route path="/dashboard" element={<Dashboard />} />
				</Routes>
			</Suspense>
		</BrowserRouter>
	);
}
```

## Suspense - Loading States

`Suspense` shows a fallback while lazy components load.

```jsx
// Nested Suspense boundaries
<Suspense fallback={<PageLoader />}>
	<Header />
	<Suspense fallback={<SidebarLoader />}>
		<Sidebar />
	</Suspense>
	<Suspense fallback={<ContentLoader />}>
		<Content />
	</Suspense>
</Suspense>
```

## Best Practices

### When to Split:

- **Routes** - Different pages
- **Modals/Dialogs** - Shown conditionally
- **Tabs** - Content not immediately visible
- **Heavy libraries** - Charts, editors, etc.
- **Admin features** - Used by few users

## Key Takeaways

- Code splitting reduces initial bundle size
- Use `React.lazy()` for dynamic imports
- Wrap lazy components in `Suspense`
- Route-based splitting is the most common pattern
- Split code for modals, tabs, and heavy features
- Provide good loading states with Suspense fallbacks
- Vite automatically code-splits lazy imports

## Related

- Series: [[react-katas]]
- Prev: [[react-memo]]
- Next: [[provider-pattern]]
- Also: [[memoization]], [[profiling-and-debugging]]
