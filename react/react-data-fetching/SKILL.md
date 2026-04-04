---
name: react-data-fetching
description: Modern React data fetching patterns. Use when implementing caching, deduplication, optimistic updates, or parallel loading with TanStack Query, SWR, or Suspense.
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

# React Data Fetching Patterns

Production-ready patterns for fetching, caching, and synchronizing server data in React applications. These patterns are framework-agnostic — they work whether you're using Vite + React Router, Next.js, Remix, or a custom setup.

## When to Use

Reference these patterns when:
- Adding data fetching to components
- Replacing `useEffect` + `fetch` with a proper data layer
- Implementing caching, deduplication, or optimistic updates
- Debugging waterfall loading patterns
- Choosing between data fetching libraries

## Instructions

- Apply these patterns during code generation, review, and refactoring. When you see fetch-in-effect without caching or deduplication, suggest the appropriate pattern.

## Details

### Overview

The most common performance problem in React apps is **request waterfalls** — sequential fetches that could run in parallel. The second most common problem is **redundant fetches** — multiple components fetching the same data independently. The patterns below address both, starting with the highest-impact fixes.

---

### 1. Parallelize Independent Fetches with `Promise.all`

**Impact: CRITICAL** — Eliminates sequential waterfalls for 2-10x improvement.

When multiple fetches have no dependencies on each other, run them concurrently.

**Avoid — sequential (3 round trips):**

```typescript
async function loadDashboard() {
  const user = await fetchUser()
  const posts = await fetchPosts()
  const notifications = await fetchNotifications()
  return { user, posts, notifications }
}
```

**Prefer — parallel (1 round trip):**

```typescript
async function loadDashboard() {
  const [user, posts, notifications] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchNotifications(),
  ])
  return { user, posts, notifications }
}
```

When fetches have partial dependencies (B depends on A, but C doesn't), start independent work immediately:

```typescript
async function loadPage() {
  const userPromise = fetchUser()
  const configPromise = fetchConfig()

  const user = await userPromise
  const [config, posts] = await Promise.all([
    configPromise,
    fetchPosts(user.id), // depends on user
  ])
  return { user, config, posts }
}
```

---

### 2. Defer Await Until the Value Is Needed

**Impact: HIGH** — Starts work earlier without blocking on results you don't need yet.

A common mistake is to `await` each promise immediately, even when subsequent code doesn't need the result right away. Start the promise early, then `await` it at the point where you actually read the value.

**Avoid — blocks unnecessarily:**

```typescript
async function loadProfile(userId: string) {
  const user = await fetchUser(userId)       // waits here
  const prefs = await fetchPreferences()     // starts only after user resolves
  const avatar = buildAvatarUrl(user.avatar)
  return { user, prefs, avatar }
}
```

**Prefer — start early, await late:**

```typescript
async function loadProfile(userId: string) {
  const userPromise = fetchUser(userId)      // starts immediately
  const prefsPromise = fetchPreferences()    // starts immediately

  const user = await userPromise             // await when needed
  const avatar = buildAvatarUrl(user.avatar)
  const prefs = await prefsPromise           // may already be resolved

  return { user, prefs, avatar }
}
```

This is complementary to `Promise.all` — use defer-await when you need intermediate results between fetches, and `Promise.all` when you can wait for everything at once.

---

### 3. Use TanStack Query for Client-Side Data

**Impact: CRITICAL** — Automatic caching, deduplication, revalidation, and error handling.

Raw `useEffect` + `fetch` lacks caching, deduplication, retry, and background refresh. Use a data fetching library.

**Avoid — no caching, no dedup, no error handling:**

```tsx
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null)
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    setLoading(true)
    fetch(`/api/users/${userId}`)
      .then(r => r.json())
      .then(setUser)
      .finally(() => setLoading(false))
  }, [userId])

  if (loading) return <Skeleton />
  return <div>{user?.name}</div>
}
```

**Prefer — TanStack Query (recommended for Vite + React apps):**

```tsx
import { useQuery } from '@tanstack/react-query'

function UserProfile({ userId }: { userId: string }) {
  const { data: user, isLoading } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetch(`/api/users/${userId}`).then(r => r.json()),
  })

  if (isLoading) return <Skeleton />
  return <div>{user?.name}</div>
}
```

TanStack Query is the strongest choice for Vite apps — it's framework-agnostic, has built-in `useSuspenseQuery`, devtools, infinite queries, optimistic mutations, and offline support. SWR is a lighter alternative that covers the basics (dedup, caching, revalidation) but has fewer features for complex mutation workflows.

Both give you: request deduplication, stale-while-revalidate caching, automatic retries, and background refresh.

**Setup for Vite apps:**

```tsx
// main.tsx
import { QueryClient, QueryClientProvider } from '@tanstack/react-query'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60_000, // 1 minute
      retry: 2,
    },
  },
})

createRoot(document.getElementById('root')!).render(
  <QueryClientProvider client={queryClient}>
    <App />
  </QueryClientProvider>
)
```

---

### 4. Use Suspense for Declarative Loading States

**Impact: HIGH** — Cleaner code, automatic loading coordination, streaming support.

Suspense lets you declare loading boundaries in the component tree instead of managing `isLoading` state in every component.

**Avoid — manual loading orchestration:**

```tsx
function Dashboard() {
  const { data: user, isLoading: userLoading } = useQuery(userQuery)
  const { data: stats, isLoading: statsLoading } = useQuery(statsQuery)

  if (userLoading || statsLoading) return <FullPageSpinner />
  return (
    <div>
      <UserHeader user={user} />
      <StatsPanel stats={stats} />
    </div>
  )
}
```

**Prefer — Suspense boundaries:**

```tsx
function Dashboard() {
  return (
    <Suspense fallback={<FullPageSpinner />}>
      <DashboardContent />
    </Suspense>
  )
}

function DashboardContent() {
  const { data: user } = useSuspenseQuery(userQuery)
  const { data: stats } = useSuspenseQuery(statsQuery)
  return (
    <div>
      <UserHeader user={user} />
      <StatsPanel stats={stats} />
    </div>
  )
}
```

For independent sections, use separate Suspense boundaries so they load independently:

```tsx
function Dashboard() {
  return (
    <div>
      <Suspense fallback={<HeaderSkeleton />}>
        <UserHeader />
      </Suspense>
      <Suspense fallback={<StatsSkeleton />}>
        <StatsPanel />
      </Suspense>
    </div>
  )
}
```

TanStack Query provides `useSuspenseQuery` and SWR provides `{ suspense: true }` option.

---

### 5. Prefetch Data Before Navigation

**Impact: HIGH** — Eliminates loading states on page transitions.

Start fetching data before the user commits to a navigation — on hover, focus, or route preload.

**With TanStack Query:**

```tsx
import { useQueryClient } from '@tanstack/react-query'

function ProjectLink({ projectId }: { projectId: string }) {
  const queryClient = useQueryClient()

  const prefetch = () => {
    queryClient.prefetchQuery({
      queryKey: ['project', projectId],
      queryFn: () => fetchProject(projectId),
      staleTime: 30_000,
    })
  }

  return (
    <Link
      to={`/projects/${projectId}`}
      onMouseEnter={prefetch}
      onFocus={prefetch}
    >
      View Project
    </Link>
  )
}
```

**With React Router loaders (Vite apps):**

```tsx
// routes.tsx
const routes = [
  {
    path: '/projects/:id',
    loader: ({ params }) => queryClient.ensureQueryData({
      queryKey: ['project', params.id],
      queryFn: () => fetchProject(params.id!),
    }),
    Component: ProjectPage,
  },
]
```

---

### 6. Use `React.cache()` for Server-Side Deduplication

**Impact: MEDIUM** — Deduplicates expensive operations within a single server render.

In server components (RSC), `React.cache()` ensures the same async call made by multiple components only executes once per request.

```typescript
import { cache } from 'react'

export const getSession = cache(async () => {
  const session = await auth()
  if (!session?.user?.id) return null
  return session
})

export const getUser = cache(async (userId: string) => {
  return await db.user.findUnique({ where: { id: userId } })
})
```

Multiple components calling `getSession()` in the same render share one execution.

**Important:** Use primitive arguments (strings, numbers) for cache keys. Inline objects create new references and cause cache misses:

```typescript
// Cache miss every time — new object reference
getUser({ id: '123' })
getUser({ id: '123' }) // miss

// Cache hit — same string value
getUser('123')
getUser('123') // hit
```

---

### 7. Implement Optimistic Updates for Instant Feedback

**Impact: HIGH** — UI responds immediately without waiting for the server.

For mutations where the outcome is predictable (toggling a like, updating a name), update the UI instantly and reconcile with the server response.

**With TanStack Query:**

```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query'

function LikeButton({ postId }: { postId: string }) {
  const queryClient = useQueryClient()

  const { mutate: toggleLike } = useMutation({
    mutationFn: () => fetch(`/api/posts/${postId}/like`, { method: 'POST' }),
    onMutate: async () => {
      await queryClient.cancelQueries({ queryKey: ['post', postId] })
      const previous = queryClient.getQueryData<Post>(['post', postId])
      queryClient.setQueryData<Post>(['post', postId], old => ({
        ...old!,
        liked: !old!.liked,
        likeCount: old!.liked ? old!.likeCount - 1 : old!.likeCount + 1,
      }))
      return { previous }
    },
    onError: (_err, _vars, context) => {
      queryClient.setQueryData(['post', postId], context?.previous)
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['post', postId] })
    },
  })

  return <button onClick={() => toggleLike()}>Like</button>
}
```

---

### 8. Avoid Fetch Waterfalls in Component Trees

**Impact: CRITICAL** — Parent-then-child fetching is the #1 performance problem.

When a parent fetches data and a child fetches its own data based on the parent's result, you create a waterfall. Restructure to fetch in parallel.

**Avoid — child can't start until parent finishes:**

```tsx
function UserPage({ userId }: { userId: string }) {
  const { data: user } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  })

  if (!user) return <Skeleton />
  return <UserPosts userId={user.id} /> // starts fetching only after user loads
}

function UserPosts({ userId }: { userId: string }) {
  const { data: posts } = useQuery({
    queryKey: ['posts', userId],
    queryFn: () => fetchPosts(userId),
  })
  // ...
}
```

**Prefer — fetch both at the same level:**

```tsx
function UserPage({ userId }: { userId: string }) {
  const { data: user } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  })
  const { data: posts } = useQuery({
    queryKey: ['posts', userId],
    queryFn: () => fetchPosts(userId),
  })

  if (!user) return <Skeleton />
  return (
    <div>
      <UserHeader user={user} />
      <PostList posts={posts ?? []} />
    </div>
  )
}
```

Or use a route-level loader to fetch all data before the component renders.

---

### 9. Deduplicate Global Event Listeners

**Impact: MEDIUM** — Prevents N listeners for N component instances.

When multiple component instances need the same global event (resize, scroll, online), share a single listener.

```typescript
// hooks/useOnlineStatus.ts
import { useSyncExternalStore } from 'react'

function subscribe(callback: () => void) {
  window.addEventListener('online', callback)
  window.addEventListener('offline', callback)
  return () => {
    window.removeEventListener('online', callback)
    window.removeEventListener('offline', callback)
  }
}

function getSnapshot() {
  return navigator.onLine
}

export function useOnlineStatus() {
  return useSyncExternalStore(subscribe, getSnapshot, () => true)
}
```

`useSyncExternalStore` automatically deduplicates subscriptions and ensures consistent state across concurrent renders.

---

### 10. Use Passive Event Listeners for Scroll and Touch

**Impact: LOW-MEDIUM** — Prevents scroll jank from blocking listeners.

Non-passive scroll/touch listeners block the browser's compositor thread. Always mark them passive when you don't call `preventDefault()`.

**Avoid — blocks scrolling:**

```tsx
useEffect(() => {
  const handler = () => trackScroll(window.scrollY)
  window.addEventListener('scroll', handler)
  return () => window.removeEventListener('scroll', handler)
}, [])
```

**Prefer — non-blocking:**

```tsx
useEffect(() => {
  const handler = () => trackScroll(window.scrollY)
  window.addEventListener('scroll', handler, { passive: true })
  return () => window.removeEventListener('scroll', handler)
}, [])
```

---

### 11. Schema-Version Your Client Storage

**Impact: LOW-MEDIUM** — Prevents crashes from stale localStorage data.

When reading from localStorage or sessionStorage, stale data from a previous app version can crash your app. Add a schema version and validate.

**Avoid — crashes on schema change:**

```tsx
const [prefs, setPrefs] = useState(() => {
  return JSON.parse(localStorage.getItem('prefs') || '{}')
})
```

**Prefer — versioned with fallback:**

```tsx
const PREFS_VERSION = 2

const [prefs, setPrefs] = useState<Prefs>(() => {
  try {
    const raw = localStorage.getItem('prefs')
    if (!raw) return DEFAULT_PREFS
    const parsed = JSON.parse(raw)
    if (parsed._v !== PREFS_VERSION) return DEFAULT_PREFS
    return parsed
  } catch {
    return DEFAULT_PREFS
  }
})

// On save, include version
useEffect(() => {
  localStorage.setItem('prefs', JSON.stringify({ ...prefs, _v: PREFS_VERSION }))
}, [prefs])
```

---

## Source

Patterns from [patterns.dev](https://www.patterns.dev/) — framework-agnostic React data fetching guidance for the broader web engineering community.
