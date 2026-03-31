---
name: react-render-optimization
description: React rendering performance patterns. Use when reducing re-renders, optimizing memoization, state design, or reviewing React performance.
context: fork
allowed-tools: Read, Grep, Glob
paths:
  - "**/*.tsx"
  - "**/*.jsx"
license: MIT
metadata:
  author: patterns.dev
  version: "1.0"
---

# React Render Optimization

Practical patterns for eliminating unnecessary re-renders, reducing rendering cost, and keeping React UIs responsive. These patterns apply to any React application — whether you're using Vite, Next.js, Remix, or a custom setup.

## When to Use

Reference these patterns when:
- Components re-render more often than expected
- UI feels sluggish during typing, scrolling, or interactions
- Profiler shows wasted renders in the component tree
- Building performance-sensitive features (dashboards, editors, lists)
- Reviewing or refactoring existing React components

## Overview

React re-renders a component whenever its state changes, a parent re-renders, or context it consumes updates. Most re-renders are harmless, but when they trigger expensive computation, deep trees, or layout thrashing they become visible to users.

The patterns below are ordered by impact — address the biggest wins first before reaching for micro-optimizations.

---

## Instructions

Apply these patterns during code generation, review, and refactoring. When you see an anti-pattern, suggest the corrected version with an explanation.

---

## 1. Compute Derived Values During Render — Don't Store Them

**Impact: HIGH** — Eliminates an entire category of bugs and unnecessary state.

Storing values that can be computed from existing state or props creates synchronization problems and extra re-renders. Compute them inline instead.

**Avoid — redundant state that drifts:**

```tsx
function ProductList({ products }: { products: Product[] }) {
  const [search, setSearch] = useState('')
  const [filtered, setFiltered] = useState(products)

  useEffect(() => {
    setFiltered(products.filter(p =>
      p.name.toLowerCase().includes(search.toLowerCase())
    ))
  }, [products, search])

  return (
    <>
      <input value={search} onChange={e => setSearch(e.target.value)} />
      {filtered.map(p => <ProductCard key={p.id} product={p} />)}
    </>
  )
}
```

**Prefer — derive during render:**

```tsx
function ProductList({ products }: { products: Product[] }) {
  const [search, setSearch] = useState('')

  const filtered = useMemo(
    () => products.filter(p =>
      p.name.toLowerCase().includes(search.toLowerCase())
    ),
    [products, search]
  )

  return (
    <>
      <input value={search} onChange={e => setSearch(e.target.value)} />
      {filtered.map(p => <ProductCard key={p.id} product={p} />)}
    </>
  )
}
```

For cheap derivations (boolean flags, string formatting), skip `useMemo` entirely — a plain `const` is fine.

> **React Compiler note:** If React Compiler is enabled, it auto-memoizes expressions and you can skip manual `useMemo` calls.

---

## 2. Subscribe to Coarse-Grained State, Not Raw Values

**Impact: HIGH** — Prevents re-renders on irrelevant changes.

If your component only cares about a derived boolean (e.g., "is mobile?"), don't subscribe to the raw value that changes continuously.

**Avoid — re-renders on every pixel:**

```tsx
function Sidebar() {
  const width = useWindowWidth() // fires on every resize
  const isMobile = width < 768
  return <nav className={isMobile ? 'mobile' : 'desktop'}>...</nav>
}
```

**Prefer — re-renders only when the boolean flips:**

```tsx
function Sidebar() {
  const isMobile = useMediaQuery('(max-width: 767px)')
  return <nav className={isMobile ? 'mobile' : 'desktop'}>...</nav>
}
```

This applies broadly: subscribe to `isLoggedIn` rather than the entire user object, `hasItems` rather than the full cart array, etc.

---

## 3. Extract Expensive Subtrees into Memoized Components

**Impact: HIGH** — Enables early returns and skip-rendering.

When a parent has fast paths (loading, error, empty), expensive children still compute if they live in the same component. Extract them so React can skip their render entirely.

**Avoid — avatar computation runs even during loading:**

```tsx
function Profile({ user, loading }: Props) {
  const avatar = useMemo(() => processAvatar(user), [user])

  if (loading) return <Skeleton />
  return <div><img src={avatar} /></div>
}
```

**Prefer — computation skipped when loading:**

```tsx
const UserAvatar = memo(function UserAvatar({ user }: { user: User }) {
  const avatar = useMemo(() => processAvatar(user), [user])
  return <img src={avatar} />
})

function Profile({ user, loading }: Props) {
  if (loading) return <Skeleton />
  return <div><UserAvatar user={user} /></div>
}
```

> **React Compiler note:** The compiler auto-memoizes, making manual `memo()` wrapping less necessary. But extracting components for early returns is still valuable.

---

## 4. Use Lazy State Initialization

**Impact: MEDIUM** — Avoids wasted computation on every render.

When `useState` receives a function call as its initial value, that call executes on every render even though the result is only used once. Pass a function reference instead.

**Avoid — `buildIndex()` runs every render:**

```tsx
const [index, setIndex] = useState(buildSearchIndex(items))
```

**Prefer — runs only on mount:**

```tsx
const [index, setIndex] = useState(() => buildSearchIndex(items))
```

Use lazy init for: `JSON.parse`, `localStorage` reads, building data structures, heavy transformations. Skip it for simple primitives like `useState(0)` or `useState(false)`.

---

## 5. Use Functional setState for Stable Callbacks

**Impact: MEDIUM** — Removes state variables from dependency arrays.

When a callback only needs the previous state to compute the next state, use the functional form. This eliminates the state variable from the dependency array and produces a stable callback identity.

**Avoid — callback changes when `count` changes:**

```tsx
const [count, setCount] = useState(0)
const increment = useCallback(() => setCount(count + 1), [count])
```

**Prefer — callback is always stable:**

```tsx
const [count, setCount] = useState(0)
const increment = useCallback(() => setCount(c => c + 1), [])
```

---

## 6. Put Interaction Logic in Event Handlers, Not Effects

**Impact: MEDIUM** — Avoids re-running side effects on dependency changes.

If a side effect is triggered by a user action (click, submit, drag), run it in the event handler. Modeling it as state + effect causes re-runs when unrelated dependencies change.

**Avoid — effect re-runs when `theme` changes:**

```tsx
function Form() {
  const [submitted, setSubmitted] = useState(false)
  const theme = useContext(ThemeContext)

  useEffect(() => {
    if (submitted) {
      post('/api/register')
      showToast('Registered', theme)
    }
  }, [submitted, theme])

  return <button onClick={() => setSubmitted(true)}>Submit</button>
}
```

**Prefer — logic in the handler:**

```tsx
function Form() {
  const theme = useContext(ThemeContext)

  function handleSubmit() {
    post('/api/register')
    showToast('Registered', theme)
  }

  return <button onClick={handleSubmit}>Submit</button>
}
```

---

## 7. Use `useRef` for Transient, High-Frequency Values

**Impact: MEDIUM** — Prevents re-renders on rapid updates.

Values that change very frequently (mouse position, scroll offset, interval ticks) but don't need to drive re-renders should live in a ref. Update the DOM directly when needed.

**Avoid — re-renders on every mouse move:**

```tsx
function Cursor() {
  const [x, setX] = useState(0)

  useEffect(() => {
    const handler = (e: MouseEvent) => setX(e.clientX)
    window.addEventListener('mousemove', handler)
    return () => window.removeEventListener('mousemove', handler)
  }, [])

  return <div style={{ transform: `translateX(${x}px)` }} />
}
```

**Prefer — zero re-renders:**

```tsx
function Cursor() {
  const ref = useRef<HTMLDivElement>(null)

  useEffect(() => {
    const handler = (e: MouseEvent) => {
      if (ref.current) {
        ref.current.style.transform = `translateX(${e.clientX}px)`
      }
    }
    window.addEventListener('mousemove', handler)
    return () => window.removeEventListener('mousemove', handler)
  }, [])

  return <div ref={ref} />
}
```

---

## 8. Use `startTransition` for Non-Urgent Updates

**Impact: MEDIUM** — Keeps high-priority updates (typing, clicking) responsive.

Wrap non-urgent state updates in `startTransition` so React can interrupt them for urgent work. This is especially useful for search filtering, tab switching, and list re-sorting.

**Avoid — typing blocks while list filters:**

```tsx
function Search({ items }: { items: Item[] }) {
  const [query, setQuery] = useState('')
  const [filtered, setFiltered] = useState(items)

  function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    setQuery(e.target.value)
    setFiltered(items.filter(i => i.name.includes(e.target.value)))
  }

  return (
    <>
      <input value={query} onChange={handleChange} />
      <List items={filtered} />
    </>
  )
}
```

**Prefer — input stays responsive:**

```tsx
import { useState, useTransition } from 'react'

function Search({ items }: { items: Item[] }) {
  const [query, setQuery] = useState('')
  const [filtered, setFiltered] = useState(items)
  const [isPending, startTransition] = useTransition()

  function handleChange(e: React.ChangeEvent<HTMLInputElement>) {
    setQuery(e.target.value)
    startTransition(() => {
      setFiltered(items.filter(i => i.name.includes(e.target.value)))
    })
  }

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending && <Spinner />}
      <List items={filtered} />
    </>
  )
}
```

---

## 9. Defer State Reads to the Point of Use

**Impact: MEDIUM** — Avoids subscriptions to state you only read in callbacks.

Don't call hooks like `useSearchParams()` if you only read the value inside an event handler. Read it on demand instead.

**Avoid — component re-renders on every URL change:**

```tsx
function ShareButton({ id }: { id: string }) {
  const [searchParams] = useSearchParams()

  const handleShare = () => {
    const ref = searchParams.get('ref')
    share(id, { ref })
  }

  return <button onClick={handleShare}>Share</button>
}
```

**Prefer — reads on demand, no subscription:**

```tsx
function ShareButton({ id }: { id: string }) {
  const handleShare = () => {
    const params = new URLSearchParams(window.location.search)
    share(id, { ref: params.get('ref') })
  }

  return <button onClick={handleShare}>Share</button>
}
```

---

## 10. Use Stable References for Default Props

**Impact: MEDIUM** — Prevents `memo()` from being defeated by new object/array literals.

Passing `[]` or `{}` as default prop values creates new references every render, defeating memoization on child components.

**Avoid — new array each render:**

```tsx
function Dashboard({ tabs = [] }: { tabs?: Tab[] }) {
  return <TabBar tabs={tabs} /> {/* TabBar re-renders every time */}
}
```

**Prefer — stable reference:**

```tsx
const EMPTY_TABS: Tab[] = []

function Dashboard({ tabs = EMPTY_TABS }: { tabs?: Tab[] }) {
  return <TabBar tabs={tabs} />
}
```

---

## 11. CSS `content-visibility` for Long Lists

**Impact: HIGH** — 5-10x faster initial render for long scrollable content.

Apply `content-visibility: auto` to off-screen items so the browser skips their layout and paint until they scroll into view.

```css
.list-item {
  content-visibility: auto;
  contain-intrinsic-size: 0 80px; /* estimated height */
}
```

```tsx
function MessageList({ messages }: { messages: Message[] }) {
  return (
    <div style={{ overflowY: 'auto', height: '100vh' }}>
      {messages.map(msg => (
        <div key={msg.id} className="list-item">
          <MessageCard message={msg} />
        </div>
      ))}
    </div>
  )
}
```

For 1,000 items, the browser skips layout and paint for ~990 off-screen items. Combine with virtualization (e.g., `react-window`, `@tanstack/react-virtual`) for truly massive lists.

---

## 12. Hoist Static JSX Outside Components

**Impact: LOW** — Avoids re-creating identical elements.

JSX elements that never change can be lifted to module scope. React reuses the same object reference across renders.

**Avoid — recreated every render:**

```tsx
function Page() {
  return (
    <main>
      <footer>
        <p>Copyright 2026 Acme Inc.</p>
      </footer>
    </main>
  )
}
```

**Prefer — created once:**

```tsx
const footer = (
  <footer>
    <p>Copyright 2026 Acme Inc.</p>
  </footer>
)

function Page() {
  return <main>{footer}</main>
}
```

Most impactful for large SVG elements which are expensive to recreate.

> **React Compiler note:** The compiler auto-hoists static JSX, making this manual optimization unnecessary.

---

## 13. Initialize Expensive Operations Once Per App

**Impact: LOW-MEDIUM** — Avoids duplicate init in Strict Mode and remounts.

App-wide initialization (analytics, auth checks, service workers) should not live in `useEffect` — components remount in development and in concurrent features. Use a module-level guard.

**Avoid — runs twice in dev, again on remount:**

```tsx
function App() {
  useEffect(() => {
    initAnalytics()
    checkAuth()
  }, [])
  return <Router />
}
```

**Prefer — once per app load:**

```tsx
let initialized = false

function App() {
  useEffect(() => {
    if (initialized) return
    initialized = true
    initAnalytics()
    checkAuth()
  }, [])
  return <Router />
}
```

Or initialize at the module level in your entry file (`main.tsx`), outside any component.

---

## 14. Store Event Handlers in Refs for Stable Subscriptions

**Impact: LOW** — Prevents effect re-subscriptions.

When a custom hook subscribes to an event and accepts a callback, store the callback in a ref so the subscription doesn't tear down and recreate on every render.

**Avoid — re-subscribes when handler changes:**

```tsx
function useWindowEvent(event: string, handler: (e: Event) => void) {
  useEffect(() => {
    window.addEventListener(event, handler)
    return () => window.removeEventListener(event, handler)
  }, [event, handler])
}
```

**Prefer — stable subscription:**

```tsx
function useWindowEvent(event: string, handler: (e: Event) => void) {
  const saved = useRef(handler)
  useEffect(() => { saved.current = handler }, [handler])

  useEffect(() => {
    const listener = (e: Event) => saved.current(e)
    window.addEventListener(event, listener)
    return () => window.removeEventListener(event, listener)
  }, [event])
}
```

If using React 19+, `useEffectEvent` provides this pattern as a built-in:

```tsx
import { useEffectEvent } from 'react'

function useWindowEvent(event: string, handler: (e: Event) => void) {
  const onEvent = useEffectEvent(handler)
  useEffect(() => {
    window.addEventListener(event, onEvent)
    return () => window.removeEventListener(event, onEvent)
  }, [event])
}
```

---

## 15. Prevent Hydration Flicker for Client-Only Data

**Impact: MEDIUM** — Eliminates flash of wrong content during SSR hydration.

When rendering depends on client-only data (localStorage, cookies), an inline script can set the correct value before React hydrates — avoiding both SSR errors and a visible flash.

```tsx
function ThemeRoot({ children }: { children: React.ReactNode }) {
  return (
    <>
      <div id="app-root">{children}</div>
      <script
        dangerouslySetInnerHTML={{
          __html: `(function(){
            try {
              var t = localStorage.getItem('theme') || 'light';
              document.getElementById('app-root').dataset.theme = t;
            } catch(e) {}
          })();`,
        }}
      />
    </>
  )
}
```

This approach works in any SSR setup — Next.js, Remix, or a custom Vite SSR pipeline.

---

## Source

Patterns from [patterns.dev](https://www.patterns.dev/) — framework-agnostic React performance guidance for the broader web engineering community.
