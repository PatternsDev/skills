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
  version: "1.1"
related_skills:
  - "hooks-pattern"
  - "hoc-pattern"
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

## Instructions

- Apply these patterns during code generation, review, and refactoring. When you see an anti-pattern, suggest the corrected version with an explanation.

## Details

### Overview

React re-renders a component whenever its state changes, a parent re-renders, or context it consumes updates. Most re-renders are harmless, but when they trigger expensive computation, deep trees, or layout thrashing they become visible to users.

The patterns below are ordered by impact — address the biggest wins first before reaching for micro-optimizations.

---

### 1. Compute Derived Values During Render — Don't Store Them

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

**Prefer — derive during render (cheap derivations use plain `const`):**

```tsx
function ProductList({ products }: { products: Product[] }) {
  const [search, setSearch] = useState('')

  // Cheap derivation — plain const, no useMemo needed
  const hasSearch = search.length > 0
  const normalizedSearch = search.toLowerCase()

  // Expensive derivation — useMemo is justified when iterating large arrays
  const filtered = useMemo(
    () => products.filter(p =>
      p.name.toLowerCase().includes(normalizedSearch)
    ),
    [products, normalizedSearch]
  )

  return (
    <>
      <input value={search} onChange={e => setSearch(e.target.value)} />
      {hasSearch && <ClearButton />}
      {filtered.map(p => <ProductCard key={p.id} product={p} />)}
    </>
  )
}
```

**When to use `useMemo` vs a plain `const`:**
- **Plain `const`** — boolean flags, string formatting, simple arithmetic, object property access, `.length` checks. These are essentially free and `useMemo` overhead is not worth it.
- **`useMemo`** — filtering/sorting arrays, building data structures, `JSON.parse`, expensive transformations, anything that iterates collections or involves O(n) work.

The rule: if the expression returns a primitive or is a single property access, skip `useMemo`. If it iterates or transforms data, wrap it.

> **React Compiler note:** If React Compiler is enabled, it auto-memoizes expressions and you can skip manual `useMemo` calls.

---

### 2. Subscribe to Coarse-Grained State, Not Raw Values

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

### 3. Extract Expensive Subtrees into Memoized Components

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

### 4. Use Lazy State Initialization

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

### 5. Use Functional setState for Stable Callbacks

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

### 6. Put Interaction Logic in Event Handlers, Not Effects

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

### 7. Use `useRef` for Transient, High-Frequency Values

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

### 8. Use `startTransition` for Non-Urgent Updates

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

### 9. Defer State Reads to the Point of Use

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

### 10. Use Stable References for Default Props

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

### 11. CSS `content-visibility` for Long Lists

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

### 12. Hoist Static JSX Outside Components

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

### 13. Initialize Expensive Operations Once Per App

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

### 14. Store Event Handlers in Refs for Stable Subscriptions

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

### 15. Prevent Hydration Flicker for Client-Only Data

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

### 16. Never Define Components Inside Components

**Impact: HIGH** — Causes remounting, state loss, and wasted DOM work every render.

When you define a component inside another component's render, React creates a new component type on every render. This means the entire subtree unmounts and remounts — losing all state, DOM nodes, and effect cleanup/setup.

**Avoid — `Row` is a new type every render:**

```tsx
function Table({ data }: { data: Item[] }) {
  // This creates a NEW component type on every render
  function Row({ item }: { item: Item }) {
    const [selected, setSelected] = useState(false)
    return <tr onClick={() => setSelected(!selected)}>{item.name}</tr>
  }

  return <table>{data.map(item => <Row key={item.id} item={item} />)}</table>
}
```

**Prefer — `Row` defined at module scope:**

```tsx
function Row({ item }: { item: Item }) {
  const [selected, setSelected] = useState(false)
  return <tr onClick={() => setSelected(!selected)}>{item.name}</tr>
}

function Table({ data }: { data: Item[] }) {
  return <table>{data.map(item => <Row key={item.id} item={item} />)}</table>
}
```

This also applies to components defined inside `useMemo`, `useCallback`, or any other hook. Always define components at module scope or as static properties.

---

### 17. Use `useDeferredValue` for Expensive Derived Renders

**Impact: HIGH** — Keeps the UI responsive while expensive subtrees re-render in the background.

`useDeferredValue` tells React to defer re-rendering components that depend on a fast-changing value. Unlike `useTransition` (which wraps the state update), `useDeferredValue` wraps the consumption — useful when you don't control the state setter.

**Avoid — every keystroke blocks the UI:**

```tsx
function SearchPage({ query }: { query: string }) {
  // Expensive: filters and renders 10,000 items on every keystroke
  const results = filterItems(query)
  return <ResultsList items={results} />
}
```

**Prefer — input stays responsive, results update in background:**

```tsx
import { useDeferredValue, useMemo } from 'react'

function SearchPage({ query }: { query: string }) {
  const deferredQuery = useDeferredValue(query)
  const isStale = query !== deferredQuery

  const results = useMemo(() => filterItems(deferredQuery), [deferredQuery])

  return (
    <div style={{ opacity: isStale ? 0.7 : 1 }}>
      <ResultsList items={results} />
    </div>
  )
}
```

**When to use `useDeferredValue` vs `useTransition`:**
- `useTransition` — you control the state setter and can wrap it in `startTransition`
- `useDeferredValue` — the value comes from props, a parent, or a library you don't control

---

### 18. Use Explicit Checks in Conditional Rendering

**Impact: MEDIUM** — Prevents rendering `0`, `NaN`, or empty strings to the DOM.

The `&&` operator in JSX short-circuits on falsy values — but `0`, `NaN`, and `""` are falsy yet still render as visible text nodes.

**Avoid — renders `0` to the DOM when count is zero:**

```tsx
function NotificationBadge({ count }: { count: number }) {
  return <div>{count && <Badge>{count}</Badge>}</div>
  // When count is 0, renders: <div>0</div>
}
```

**Prefer — explicit boolean check:**

```tsx
function NotificationBadge({ count }: { count: number }) {
  return <div>{count > 0 && <Badge>{count}</Badge>}</div>
}

// Or use a ternary for clarity
function NotificationBadge({ count }: { count: number }) {
  return <div>{count > 0 ? <Badge>{count}</Badge> : null}</div>
}
```

This applies to any value that might be `0`, `NaN`, or `""` — array lengths, string values, numeric props. Always use an explicit boolean expression (`> 0`, `!== ''`, `!= null`) rather than relying on truthiness.

---

### 19. Narrow Effect Dependencies to Primitives

**Impact: MEDIUM** — Prevents effects from re-running when unrelated object properties change.

When an effect only needs one property from an object, extract it before the dependency array. Passing the whole object causes re-runs whenever any property changes.

**Avoid — effect re-runs when `user.name` or `user.avatar` changes:**

```tsx
function UserStatus({ user }: { user: User }) {
  useEffect(() => {
    updatePresence(user.id)
  }, [user]) // re-runs on ANY user property change
}
```

**Prefer — only re-runs when the ID changes:**

```tsx
function UserStatus({ user }: { user: User }) {
  const { id } = user
  useEffect(() => {
    updatePresence(id)
  }, [id])
}
```

This also applies to hook return values. If `useQuery` returns `{ data, status, fetchStatus }` and your effect only cares about `status`, destructure first.

---

### 20. Split Combined Hook Computations

**Impact: MEDIUM** — Prevents re-renders for consumers that only need part of a hook's output.

When a custom hook computes multiple unrelated values, a change in one forces re-renders in all consumers — even those that only read the unchanged value.

**Avoid — changing `total` re-renders components that only need `average`:**

```tsx
function useStats(items: number[]) {
  return useMemo(() => ({
    total: items.reduce((a, b) => a + b, 0),
    average: items.reduce((a, b) => a + b, 0) / items.length,
    max: Math.max(...items),
  }), [items])
}
```

**Prefer — split into focused hooks:**

```tsx
function useTotal(items: number[]) {
  return useMemo(() => items.reduce((a, b) => a + b, 0), [items])
}

function useAverage(items: number[]) {
  return useMemo(() => items.reduce((a, b) => a + b, 0) / items.length, [items])
}

function useMax(items: number[]) {
  return useMemo(() => Math.max(...items), [items])
}
```

Components call only the hook they need. If a single component needs all three, combining them there is fine — the split prevents unnecessary coupling at the hook level.

---

### 21. Avoid Layout Thrashing with Batched DOM Reads/Writes

**Impact: HIGH** — Prevents forced synchronous layouts that block the main thread.

Reading a layout property (e.g., `offsetHeight`, `getBoundingClientRect()`) after writing to the DOM forces the browser to recalculate layout synchronously. In a loop, this creates layout thrashing.

**Avoid — forces layout recalculation on every iteration:**

```tsx
function resizeCards(cards: HTMLElement[]) {
  cards.forEach(card => {
    const height = card.offsetHeight          // READ (forces layout)
    card.style.minHeight = `${height + 20}px` // WRITE (invalidates layout)
  })
}
```

**Prefer — batch all reads, then all writes:**

```tsx
function resizeCards(cards: HTMLElement[]) {
  // Read phase
  const heights = cards.map(card => card.offsetHeight)

  // Write phase
  cards.forEach((card, i) => {
    card.style.minHeight = `${heights[i] + 20}px`
  })
}
```

In React, this most commonly occurs in `useLayoutEffect` or `useEffect` callbacks that measure and mutate DOM elements. When you need to read layout inside an animation frame, use `requestAnimationFrame` to batch:

```tsx
useLayoutEffect(() => {
  const measurements = items.map(el => el.getBoundingClientRect())

  requestAnimationFrame(() => {
    items.forEach((el, i) => {
      el.style.transform = `translateY(${measurements[i].top}px)`
    })
  })
}, [items])
```

---

### 22. Animate SVG Wrappers, Not SVG Elements Directly

**Impact: MEDIUM** — Avoids repainting the entire SVG on every animation frame.

Animating properties on an SVG element itself (e.g., `<svg>` or `<path>`) triggers a full SVG repaint. Wrap the SVG in a `<div>` and animate the wrapper instead.

**Avoid — repaints entire SVG tree:**

```tsx
<motion.svg animate={{ rotate: 360 }} style={{ width: 200, height: 200 }}>
  <ComplexChart />
</motion.svg>
```

**Prefer — only the wrapper repaints:**

```tsx
<motion.div animate={{ rotate: 360 }} style={{ width: 200, height: 200 }}>
  <svg viewBox="0 0 200 200">
    <ComplexChart />
  </svg>
</motion.div>
```

This also applies to CSS animations. Use `transform` on a wrapper element rather than animating SVG attributes like `cx`, `cy`, or `d` directly.

---

### 23. Suppress Expected Hydration Mismatches

**Impact: LOW-MEDIUM** — Silences known-safe warnings without hiding real bugs.

Some content is intentionally different between server and client — timestamps, random IDs, user-agent-specific rendering. Use `suppressHydrationWarning` on those specific elements.

```tsx
function Comment({ createdAt }: { createdAt: Date }) {
  return (
    <article>
      <p>{comment.body}</p>
      <time suppressHydrationWarning>
        {formatRelativeTime(createdAt)} {/* "2 minutes ago" differs server vs client */}
      </time>
    </article>
  )
}
```

Apply sparingly and only on leaf elements. Never suppress warnings on container elements — it masks real mismatches in children.

---

### 24. React DOM Resource Hints for Vite SPAs

**Impact: HIGH** — Lets the browser start loading critical resources earlier without framework support.

React 19 adds `preload()` and `preinit()` from `react-dom` — imperative resource hints that work in any React app. In Vite SPAs (which don't get framework-level prefetching), these are especially valuable.

```tsx
import { preload, preinit } from 'react-dom'

function App() {
  // Preload a font before it's needed
  preload('/fonts/inter-var.woff2', { as: 'font', type: 'font/woff2', crossOrigin: 'anonymous' })

  // Preinit a critical CSS file (loads + applies it)
  preinit('/critical.css', { as: 'style' })

  return <RouterProvider router={router} />
}
```

**On navigation — preload the next page's data and code:**

```tsx
function ProductLink({ id }: { id: string }) {
  const handleHover = () => {
    // Preload the image the next page will need
    preload(`/api/products/${id}/image.webp`, { as: 'image' })
    // Prefetch the route code
    import('./pages/ProductDetail')
  }

  return <Link to={`/products/${id}`} onMouseEnter={handleHover}>View</Link>
}
```

These are no-ops if the resource is already loaded, so calling them eagerly is safe. For Vite apps without a meta-framework, this is the primary mechanism for resource prioritization.

---

### 25. Use `useTransition` for Route Navigation

**Impact: MEDIUM** — Keeps the current page interactive while the next route loads.

In Vite SPAs with `React.lazy()` routes, clicking a navigation link can freeze the UI while the chunk loads and the component renders. Wrapping navigation in `startTransition` lets React show the old page until the new one is ready.

```tsx
import { useTransition } from 'react'
import { useNavigate } from 'react-router-dom'

function NavLink({ to, children }: { to: string; children: React.ReactNode }) {
  const navigate = useNavigate()
  const [isPending, startTransition] = useTransition()

  const handleClick = (e: React.MouseEvent) => {
    e.preventDefault()
    startTransition(() => {
      navigate(to)
    })
  }

  return (
    <a
      href={to}
      onClick={handleClick}
      style={{ opacity: isPending ? 0.7 : 1 }}
    >
      {children}
    </a>
  )
}
```

This prevents the blank-screen flash between lazy-loaded routes and gives you `isPending` to show a subtle loading indicator on the current page.

---

## Source

Patterns from [patterns.dev](https://www.patterns.dev/) — framework-agnostic React performance guidance for the broader web engineering community.
