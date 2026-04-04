---
name: react-2026
description: A comprehensive guide to building React apps with a modern 2026 stack, covering frameworks, build tools, routing, state management, and AI integration.
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

# React Stack Patterns

React has come a long way since its early days of simple component libraries. By late 2025, the React ecosystem is **rich but complex**, offering multiple ways to choose your stack for building apps. Modern React developers face choices at every layer of the stack - from **build tools** and **frameworks** to **routers** and other libraries. The official React documentation (now at **react.dev**) encourages using higher-level frameworks for new projects. In fact, **Create React App (CRA)** - once the go-to bootstrap tool - was deprecated in early 2025, signaling a shift in how we start React apps.

Instead of stitching together your own tooling from scratch, the recommendation is to either **use a React framework** (like Next.js or Remix) or, if you have special requirements, start from a modern **build tool** like Vite or Parcel. In this opinionated guide, we'll explore the 2025 React landscape: the **toolchain and stack** we suggest for mid-to-senior developers, covering build tools (Vite, Turbopack, Webpack), routing solutions (React Router vs. TanStack Router), popular frameworks (Next.js, Remix, etc.), key libraries for state and data management, and even how **AI is influencing React development**.

## When to Use

- Use this as a reference when choosing your React stack (framework, build tools, routing, state management)
- This is helpful when starting a new React project and evaluating modern ecosystem options

## Instructions

- For full-stack apps needing SSR/SEO: use Next.js or Remix as the framework
- For SPAs, internal tools, and dashboards: use Vite + React Router or TanStack Router
- Use Vite as the build tool for any custom (non-framework) React setup
- Consider TanStack Router for type-safe routing in custom stacks; use framework-provided routing otherwise
- Use TanStack Query (React Query) for server state and Zustand or Redux for complex global state
- Use React 19 APIs (ref as prop, use(), Actions) where supported
- Embrace AI-assisted development tools but always validate generated code

## Details

**Target Audience:** This guide is written for intermediate to senior React developers who know the basics and want to make informed choices about their stack in 2025.

### Official Guidance and Evolving Best Practices

React's core team has updated its guidance in line with community trends. The new React docs emphasize *"don't reinvent the wheel"* when starting a project. **Create React App** served us well to quickly spin up React SPAs, but it struggled to scale to production needs (e.g. lacking built-in routing or SSR). With CRA now deprecated, developers are nudged either toward **full-featured frameworks** or assembling their own stack using lighter tools. The rule of thumb given by React core: **if your app needs routing, you will benefit from a framework**. Modern frameworks tightly integrate routing with data fetching, code-splitting, and more, so you don't end up writing a mini-framework yourself.

Why this shift? Over the last few years, React has introduced powerful features like **hooks**, **concurrent rendering**, and **Server Components**. But leveraging many of these in an optimal way often requires sophisticated tooling. Frameworks like Next.js have become the vehicle for React's newest capabilities (e.g. streaming SSR, server-side data loading, etc.). **React Server Components (RSC)** landed as a production-ready feature in Next.js 13's App Router, enabling a *hybrid rendering model* (server-driven UI with zero JS cost for server-rendered components). Early reports showed **bundle size reductions over 20%** when using RSC, since logic that runs on the server doesn't ship to the client. React 19 is also stabilizing **Server Actions** (invoked with a `'use server'` directive) which let you define form submissions or other mutations to run on the server - further blurring the line between front and back end.

That said, not every project needs a heavyweight framework. The React team acknowledges there *are cases for going lean*: small widgets, adding React to an existing site, or learning by building from scratch. If you choose the custom route, you can still follow React's updated "Build from Scratch" guides using modern tools like Vite or **RSBuild**.

### Build Tools in 2025: Vite, Turbopack, and Beyond

The **build toolchain** is what transforms your React code (JSX, CSS, etc.) into something that runs in the browser. In 2025, the developer experience has greatly improved:

* **Vite** has become the de facto choice for non-framework React projects. Vite provides a super-fast dev server (powered by ESBuild) and uses Rollup for production bundling. Its popularity has surged, and by 2025 its React integration is the second most widely used build setup after Next.js. The appeal: **instant server start** and near-instant module hot reloads. If you're migrating off CRA, the React team explicitly suggests **Vite** as a top choice.

* **Turbopack** - the new Rust-based bundler introduced by Vercel - is an upcoming star positioned as the spiritual successor to Webpack, focused on **incremental compilation** for ultrafast reloads. It's integrated within Next.js, so if you use Next, you'll likely benefit from it. Outside of Next, Turbopack isn't yet a generic tool.

* **Webpack** - the long-time workhorse - is still with us, but mostly in legacy mode. For new projects, you'd rarely choose Webpack directly in 2025 unless you have very specific needs. **Webpack for legacy, Vite (or framework default) for greenfield**.

* **Rspack / RSBuild** - part of a new breed of **Rust-powered bundlers**. **Rspack** is a high-performance bundler largely compatible with Webpack's ecosystem. **RSBuild** is a zero-config build tool built on Rspack, offering an easy setup for frameworks including React.

In summary, **our pick for most cases is Vite** - it's quick to start, fast to run, and well-supported.

### Frameworks and Starting Points

Choosing a **starting point** for a React project in 2025 often means choosing a **framework**.

**Next.js (Full-Stack React Framework):** Next.js, maintained by Vercel, has become *the* go-to solution for production React apps. It provides out-of-the-box support for **file-based routing**, **SSR**, **SSG**, **image optimization**, API routes, and **React Server Components**. Next's strength is in its **strong defaults and conventions**. Our **top recommendation for building a new React app** when you want a complete solution.

**Remix (and React Router v7):** Remix emphasizes web fundamentals and progressive enhancement. Created by the React Router team, Remix introduced an integrated approach to **routing + data loading + mutations**. React Router v7 adopted many of Remix's patterns. A solid choice especially for apps where you want fine-grained control over server-vs-client work.

**"No Framework" Custom Setup (Vite + library stack):** If you decide a full framework is not needed, start with **Vite** and add pieces: e.g. **React Router or TanStack Router for routing, React Query for data fetching**. You might choose this for something like an internal tool where SSR/SEO are irrelevant.

**Other Notables:** **Astro** has gained attention for content sites (zero JS by default, mix frameworks). **RedwoodJS** offers an opinionated "React + GraphQL + Prisma + SSR" stack. **Expo** with the **Expo Router** brings React Native and web together.

**Our opinionated advice:** Match the tool to the job:
- **Next.js** for public-facing apps that need SSR, SEO, or server components
- **Remix** for apps that emphasize progressive enhancement and web fundamentals
- **Vite + React Router/TanStack Router** for SPAs, dashboards, internal tools, and any app where SSR isn't a requirement

**Custom Vite SSR (without a meta-framework):** If you need SSR but don't want a full framework, Vite has built-in SSR support. Tools like **Vike** (formerly vite-plugin-ssr) provide a thin layer on top of Vite for file-based routing with SSR, giving you framework-like DX while keeping full control.

```typescript
// server.ts — minimal custom Vite SSR setup
import express from 'express'
import { createServer as createViteServer } from 'vite'

const app = express()
const vite = await createViteServer({ server: { middlewareMode: true } })
app.use(vite.middlewares)

app.use('*', async (req, res) => {
  const template = await vite.transformIndexHtml(req.originalUrl, indexHtml)
  const { render } = await vite.ssrLoadModule('/src/entry-server.tsx')
  const appHtml = await render(req.originalUrl)
  res.send(template.replace('<!--app-->', appHtml))
})
```

This is a good fit when you need SSR for SEO or performance but your app's routing and data loading are simple enough that a framework adds more complexity than value.

Many successful React apps in 2025 run on Vite without a meta-framework. The key is choosing the right tool for your constraints, not defaulting to the most feature-rich option.

### Routing Solutions: React Router vs. TanStack Router

**Framework-provided Routing:** If you use Next.js or Remix, routing is largely solved by the framework. These frameworks tie routing with data fetching closely, which avoids common pitfalls like loading waterfalls.

**React Router:** Battle-tested and widely used. React Router v6 introduced a simplified API with hooks and nested routes. By v6.4+, it added **async data and suspense support**. Still a top choice if you're not using a framework.

**TanStack Router:** A newer entrant with first-class **TypeScript** support, built-in **data loaders with caching**, and a rich API for **search params**. It's trying to give you the power of a Remix/Next router but decoupled from a specific framework.

The **biggest differences**: **Type Safety** and **Developer Experience**. TanStack Router is built "TS-first". React Router is more minimal. Both can do nested routes and data loading.

**Our recommendation:** For custom stacks, give **TanStack Router** a serious look for its modern feature set. React Router is still perfectly valid, especially if your team is already familiar with it.

```jsx
import { createRootRoute, createRoute, createRouter, RouterProvider } from '@tanstack/react-router';

const rootRoute = createRootRoute();
const indexRoute = createRoute({
  getParentRoute: () => rootRoute,
  path: '/',
  component: () => <div>Hello, world!</div>,
});

const routeTree = rootRoute.addChildren([indexRoute]);
const router = createRouter({ routeTree });

export default function App() {
  return <RouterProvider router={router} />;
}
```

### State Management and Data Fetching Libraries

**Local and Global State with Hooks:** For local state, `useState` and `useReducer` suffice. React's Context API works for lightweight global state but be careful — context updates re-render all consumers.

**Redux (and modern Redux Toolkit):** Very much alive in 2025, though concentrated in large applications. **Redux Toolkit (RTK)** significantly reduces boilerplate. **When should you reach for Redux?** If your app has very complex state transitions or you need features like undo/redo, caching, devtools.

**Zustand, Jotai, and lightweight state libraries:** For simpler global store needs. Zustand provides a minimalistic, hook-based global state store with **no boilerplate**.

**TanStack Query:** The leader for managing **server state**. Provides hooks like `useQuery` and `useMutation` to declaratively fetch and cache data.

```tsx
import { useQuery } from '@tanstack/react-query';

function TodoList() {
  const { data: todos, error, isLoading } = useQuery({
    queryKey: ['todos'],
    queryFn: fetchTodos,
  });
  if (isLoading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  return <ul>{todos.map(t => <li key={t.id}>{t.title}</li>)}</ul>;
}
```

**React Query and friends remain relevant** even in the age of RSC — they might be used more for *mutations and real-time updates* while initial loads shift to server-side.

**Form State:** **React Hook Form** has established itself as a great library. Coupled with **Zod** for schemas, you can validate inputs declaratively.

**Key Libraries:** MUI, Chakra UI, Radix UI, Headless UI for components. **@tanstack/react-virtual** for list virtualization. **Vitest + React Testing Library** for testing in Vite projects, **Jest + React Testing Library** for other setups, **Cypress** or **Playwright** for E2E.

### Testing with Vitest

For Vite projects, **Vitest** is the natural testing companion — it shares Vite's config, transforms, and plugin pipeline, so there's no separate test bundler to configure or keep in sync.

```typescript
// vite.config.ts — Vitest uses this directly
/// <reference types="vitest/config" />
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: './src/test/setup.ts',
    css: true,
  },
})
```

```typescript
// src/test/setup.ts
import '@testing-library/jest-dom/vitest'
```

```tsx
// src/components/Button.test.tsx
import { render, screen } from '@testing-library/react'
import userEvent from '@testing-library/user-event'
import { Button } from './Button'

test('calls onClick when clicked', async () => {
  const onClick = vi.fn()
  render(<Button onClick={onClick}>Save</Button>)
  await userEvent.click(screen.getByRole('button', { name: 'Save' }))
  expect(onClick).toHaveBeenCalledOnce()
})
```

Vitest gives you: instant watch mode (shared with Vite's transform cache), native ESM support, Jest-compatible API (`describe`, `it`, `expect`, `vi.fn()`), in-source testing, and built-in code coverage with `v8` or `istanbul`.

### React 19 and Modern APIs

React 19 brings several significant API changes that simplify common patterns:

**ref as a regular prop:** No more `forwardRef` — pass `ref` directly as a prop:

```tsx
// React 19 — ref is just a prop
function Input({ ref, ...props }: InputProps & { ref?: React.Ref<HTMLInputElement> }) {
  return <input ref={ref} {...props} />
}
```

**`use()` API:** Can read promises or context, and unlike Hooks it can be called conditionally:

```tsx
import { use } from 'react'

function UserPanel({ show }: { show: boolean }) {
  if (!show) return null
  const user = use(UserContext)
  return <div>{user.name}</div>
}
```

**Actions and useActionState:** Simplify form handling with server and client actions:

```tsx
function ContactForm() {
  const [state, formAction, isPending] = useActionState(submitContact, null)
  return (
    <form action={formAction}>
      <input name="email" type="email" required />
      <button disabled={isPending}>Submit</button>
      {state?.error && <p>{state.error}</p>}
    </form>
  )
}
```

**`useOptimistic`:** Instant UI feedback during async operations:

```tsx
function TodoList({ todos }: { todos: Todo[] }) {
  const [optimisticTodos, addOptimistic] = useOptimistic(todos)

  async function addTodo(text: string) {
    addOptimistic([...optimisticTodos, { id: 'temp', text, pending: true }])
    await saveTodo(text)
  }

  return <ul>{optimisticTodos.map(t => <li key={t.id}>{t.text}</li>)}</ul>
}
```

**React Compiler:** An opt-in compiler that auto-memoizes components and expressions, eliminating the need for manual `useMemo`, `useCallback`, and `React.memo` in most cases. For Vite projects, add it as a Babel plugin:

```typescript
// vite.config.ts
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [
    react({
      babel: {
        plugins: ['babel-plugin-react-compiler'],
      },
    }),
  ],
})
```

Requires React 19. Can be adopted incrementally per-file with a `'use memo'` directive.

### Vite-Specific Best Practices

When using Vite + React (without a meta-framework), follow these essentials:

- **Barrel file imports** are the #1 bundle size issue — import directly from source files, not barrel `index.ts` files. Use `vite-plugin-barrel` if you need ergonomic syntax.
- **Manual chunk splitting** — split vendor code by stability (react, router, query, UI library) for better caching.
- **Route-level code splitting** — use `React.lazy()` for each route, wrapped in `<Suspense>`.
- **Dependency pre-bundling** — add slow-to-resolve deps to `optimizeDeps.include` for faster dev server.
- **Bundle analysis** — run `npx vite-bundle-visualizer` after every major dependency change.

See the **vite-bundle-optimization** skill for complete Vite configuration patterns.

### React + AI: The New Frontier of "Vibe Coding"

**AI-Assisted Development:** Tools like Copilot, ChatGPT, or Cursor can generate components, suggest hooks, and configure build tools. The term **"vibe coding"** describes a workflow where developers collaborate with AI assistants. Senior developers get more out of it because they know what to ask and how to verify the output.

Tips for "vibe coding" effectively:
- Use AI for boilerplate and configuration
- Generate component templates
- Leverage AI for tests and types
- Stay in control — always review AI-generated code

**AI in React Apps (AI-powered UI):** React developers are increasingly asked to build UIs that incorporate AI features. Key considerations include prompt management, streaming responses, error handling, and AI-specific UI elements. The Vercel AI SDK provides hooks like `useChat` that abstract streaming and caching.

**Our take:** Embrace AI as a tool in your React development workflow. Use your expertise to guide the AI: you define the architecture, let the AI fill in the boilerplate, then you refine the results.

### Conclusion

React in 2026 offers an abundance of choices:

* **Build tool:** Start with Vite for most cases (or Turbopack if inside Next.js).
* **Framework or not:** Lean towards using a React framework (Next.js being the frontrunner) for any sizeable app.
* **Routing:** If on Next/Remix, use what's built-in. If building your own stack, consider TanStack Router for type safety or React Router for reliability.
* **State and data libraries:** Use React Query for server data, Zustand or Redux for complex global state.
* **Embrace new React features:** Hooks are standard. Use React 19 APIs (`use()`, Actions, `useOptimistic`). Understand **Server Components**. Try the React Compiler for auto-memoization.
* **Performance:** Use `useTransition` for non-urgent updates, derived state over stored state, `useSyncExternalStore` for external subscriptions. See the **react-render-optimization** and **react-data-fetching** skills.
* **AI in your workflow:** Enhance productivity with AI coding assistants, but always validate output.

React is more powerful than ever. By choosing the right stack, you'll be well-equipped to build robust, scalable applications. Happy coding, and may your components re-render only when necessary!

## Source

- [patterns.dev/react/react-2026](https://patterns.dev/react/react-2026)
