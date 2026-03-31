---
name: js-performance-patterns
description: JavaScript runtime performance patterns for hot paths, loops, DOM operations, caching, and data structures. Framework-agnostic.
license: MIT
metadata:
  author: patterns.dev
  version: "1.1"
paths:
  - "**/*.js"
  - "**/*.ts"
related_skills:
  - "module-pattern"
  - "singleton-pattern"
---

# JavaScript Performance Patterns

Runtime performance micro-patterns for JavaScript hot paths. These patterns matter most in tight loops, frequent callbacks (scroll, resize, animation frames), and data-heavy operations. They apply to any JavaScript environment — React, Vue, vanilla, Node.js.

## When to Use

Reference these patterns when:
- Profiling reveals a hot function or tight loop
- Processing large datasets (1,000+ items)
- Handling high-frequency events (scroll, mousemove, resize)
- Optimizing build-time or server-side scripts
- Reviewing code for performance in critical paths

## Instructions

- Apply these patterns only in **measured hot paths** — code that runs frequently or processes large datasets. Don't apply them to cold code paths where readability is more important than nanosecond gains.

## Details

### Overview

Micro-optimizations are **not** a substitute for algorithmic improvements. Always address the algorithm first (O(n^2) to O(n), removing waterfalls, reducing DOM mutations). Once the algorithm is right, these patterns squeeze additional performance from hot paths.

---

### 1. Use `Set` and `Map` for Lookups

**Impact: HIGH for large collections** — O(1) vs O(n) per lookup.

Array methods like `.includes()`, `.find()`, and `.indexOf()` scan linearly. For repeated lookups against the same collection, convert to `Set` or `Map` first.

**Avoid — O(n) per check:**

```typescript
const allowedIds = ['a', 'b', 'c', /* ...hundreds more */]

function isAllowed(id: string) {
  return allowedIds.includes(id) // scans entire array
}

items.filter(item => allowedIds.includes(item.id)) // O(n * m)
```

**Prefer — O(1) per check:**

```typescript
const allowedIds = new Set(['a', 'b', 'c', /* ...hundreds more */])

function isAllowed(id: string) {
  return allowedIds.has(id)
}

items.filter(item => allowedIds.has(item.id)) // O(n)
```

For key-value lookups, use `Map` instead of scanning an array of objects:

```typescript
// Avoid
const users = [{ id: 1, name: 'Alice' }, { id: 2, name: 'Bob' }]
const user = users.find(u => u.id === targetId) // O(n)

// Prefer
const userMap = new Map(users.map(u => [u.id, u]))
const user = userMap.get(targetId) // O(1)
```

---

### 2. Batch DOM Reads and Writes

**Impact: HIGH** — Prevents layout thrashing.

Interleaving DOM reads (e.g., `offsetHeight`, `getBoundingClientRect`) with DOM writes (e.g., `style.height = ...`) forces the browser to recalculate layout multiple times. Batch all reads first, then all writes.

**Avoid — layout thrashing (read/write/read/write):**

```typescript
elements.forEach(el => {
  const height = el.offsetHeight    // read → forces layout
  el.style.height = `${height * 2}px` // write
})
// Each iteration forces a layout recalculation
```

**Prefer — batched reads then writes:**

```typescript
// Read phase
const heights = elements.map(el => el.offsetHeight)

// Write phase
elements.forEach((el, i) => {
  el.style.height = `${heights[i] * 2}px`
})
```

For complex cases, use `requestAnimationFrame` to defer writes to the next frame, or use a library like [fastdom](https://github.com/wilsonpage/fastdom).

**CSS class approach — single reflow:**

```typescript
// Avoid multiple style mutations
el.style.width = '100px'
el.style.height = '200px'
el.style.margin = '10px'

// Prefer — one reflow
el.classList.add('expanded')
// or
el.style.cssText = 'width:100px;height:200px;margin:10px;'
```

---

### 3. Cache Property Access in Tight Loops

**Impact: MEDIUM** — Reduces repeated property resolution.

Accessing deeply nested properties or array `.length` in every iteration adds overhead in tight loops.

**Avoid:**

```typescript
for (let i = 0; i < data.items.length; i++) {
  process(data.items[i].value.nested.prop)
}
```

**Prefer:**

```typescript
const { items } = data
for (let i = 0, len = items.length; i < len; i++) {
  const val = items[i].value.nested.prop
  process(val)
}
```

This matters for arrays with 10,000+ items or when called at 60fps. For small arrays or infrequent calls, the readable version is fine.

---

### 4. Memoize Expensive Function Results

**Impact: MEDIUM-HIGH** — Avoids recomputing the same result.

When a pure function is called repeatedly with the same arguments, cache the result.

**Simple single-value cache:**

```typescript
function memoize<T extends (...args: any[]) => any>(fn: T): T {
  let lastArgs: any[] | undefined
  let lastResult: any

  return ((...args: any[]) => {
    if (lastArgs && args.every((arg, i) => Object.is(arg, lastArgs![i]))) {
      return lastResult
    }
    lastArgs = args
    lastResult = fn(...args)
    return lastResult
  }) as T
}

const expensiveCalc = memoize((data: number[]) => {
  return data.reduce((sum, n) => sum + heavyTransform(n), 0)
})
```

**Multi-key cache with Map:**

```typescript
const cache = new Map<string, Result>()

function getResult(key: string): Result {
  if (cache.has(key)) return cache.get(key)!
  const result = computeExpensiveResult(key)
  cache.set(key, result)
  return result
}
```

For caches that can grow unbounded, use an LRU strategy or `WeakMap` for object keys.

---

### 5. Combine Iterations Over the Same Data

**Impact: MEDIUM** — Single pass instead of multiple.

Chaining `.filter().map().reduce()` creates intermediate arrays and iterates the data multiple times. For large arrays in hot paths, combine into a single loop.

**Avoid — 3 iterations, 2 intermediate arrays:**

```typescript
const result = users
  .filter(u => u.active)
  .map(u => u.name)
  .reduce((acc, name) => acc + name + ', ', '')
```

**Prefer — single pass:**

```typescript
let result = ''
for (const u of users) {
  if (u.active) {
    result += u.name + ', '
  }
}
```

For small arrays (< 100 items), the chained version is fine and more readable. Optimize only when profiling shows it matters.

---

### 6. Short-Circuit with Length Checks First

**Impact: LOW-MEDIUM** — Avoids expensive operations on empty inputs.

Before running expensive comparisons or transformations, check if the input is empty.

```typescript
function findMatchingItems(items: Item[], query: string): Item[] {
  if (items.length === 0 || query.length === 0) return []

  const normalized = query.toLowerCase()
  return items.filter(item =>
    item.name.toLowerCase().includes(normalized)
  )
}
```

---

### 7. Return Early to Skip Unnecessary Work

**Impact: LOW-MEDIUM** — Reduces average-case execution.

Structure functions to exit as soon as possible for common non-matching cases.

**Avoid — always does full work:**

```typescript
function processEvent(event: AppEvent) {
  let result = null
  if (event.type === 'click') {
    if (event.target && event.target.matches('.actionable')) {
      result = handleAction(event)
    }
  }
  return result
}
```

**Prefer — exits early:**

```typescript
function processEvent(event: AppEvent) {
  if (event.type !== 'click') return null
  if (!event.target?.matches('.actionable')) return null
  return handleAction(event)
}
```

---

### 8. Hoist RegExp and Constant Creation Outside Loops

**Impact: LOW-MEDIUM** — Avoids repeated compilation.

Creating RegExp objects or constant values inside loops or frequently-called functions wastes CPU.

**Avoid — compiles regex 10,000 times:**

```typescript
function validate(items: string[]) {
  return items.filter(item => {
    const pattern = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/
    return pattern.test(item)
  })
}
```

**Prefer — compile once:**

```typescript
const EMAIL_PATTERN = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/

function validate(items: string[]) {
  return items.filter(item => EMAIL_PATTERN.test(item))
}
```

---

### 9. Use `toSorted()`, `toReversed()`, `toSpliced()` for Immutability

**Impact: LOW** — Correct immutability without manual copying.

The new non-mutating array methods avoid the `[...arr].sort()` pattern and communicate intent more clearly.

**Avoid — manual copy then mutate:**

```typescript
const sorted = [...items].sort((a, b) => a.price - b.price)
const reversed = [...items].reverse()
const without = [...items]; without.splice(index, 1)
```

**Prefer — non-mutating methods:**

```typescript
const sorted = items.toSorted((a, b) => a.price - b.price)
const reversed = items.toReversed()
const without = items.toSpliced(index, 1)
```

These are available in all modern browsers and Node.js 20+.

---

### 10. Use `requestAnimationFrame` for Visual Updates

**Impact: MEDIUM** — Syncs with the browser's render cycle.

DOM updates triggered outside the rendering cycle (from timers, event handlers, etc.) can cause jank. Batch visual updates inside `requestAnimationFrame`.

**Avoid — updates outside render cycle:**

```typescript
window.addEventListener('scroll', () => {
  progressBar.style.width = `${getScrollPercent()}%`
  counter.textContent = `${getScrollPercent()}%`
}, { passive: true })
```

**Prefer — synced to render:**

```typescript
let ticking = false

window.addEventListener('scroll', () => {
  if (!ticking) {
    requestAnimationFrame(() => {
      const pct = getScrollPercent()
      progressBar.style.width = `${pct}%`
      counter.textContent = `${pct}%`
      ticking = false
    })
    ticking = true
  }
}, { passive: true })
```

---

### 11. Use `structuredClone` for Deep Copies

**Impact: LOW** — Correct deep cloning without libraries.

`structuredClone()` handles circular references, typed arrays, Dates, RegExps, Maps, and Sets — unlike `JSON.parse(JSON.stringify())`.

```typescript
// Avoid — loses Dates, Maps, Sets, undefined values
const copy = JSON.parse(JSON.stringify(original))

// Prefer — handles all standard types
const copy = structuredClone(original)
```

Note: `structuredClone` cannot clone functions or DOM nodes. For those cases, implement a custom clone.

---

### 12. Prefer `Map` Over Plain Objects for Dynamic Keys

**Impact: LOW-MEDIUM** — Better performance for frequent additions/deletions.

V8 optimizes plain objects for static shapes. When keys are added and removed dynamically (caches, counters, registries), `Map` provides consistently better performance.

```typescript
// Avoid for dynamic keys
const counts: Record<string, number> = {}
items.forEach(item => {
  counts[item.category] = (counts[item.category] || 0) + 1
})

// Prefer for dynamic keys
const counts = new Map<string, number>()
items.forEach(item => {
  counts.set(item.category, (counts.get(item.category) ?? 0) + 1)
})
```

---

## Source

Patterns from [patterns.dev](https://www.patterns.dev/) — JavaScript performance guidance for the broader web engineering community.
