# KEY-DIFFS: patterns.dev Skills vs Vercel Skills

How the patterns.dev agent skills differ from Vercel's agent skills in philosophy, scope, framework focus, and content.

## Philosophy

| Dimension | Vercel Skills | patterns.dev Skills |
|-----------|--------------|---------------------|
| **Primary framework** | Next.js first | Framework-agnostic, Vite + React first |
| **Audience** | Next.js/Vercel platform users | Broader React web engineering community |
| **Structure** | Monolithic skills (57 rules in one skill) | Individual focused skills, one pattern per skill |
| **Scope** | React + Next.js performance | React + JS patterns, rendering, design patterns, Vue, performance |
| **Tone** | Prescriptive rules with CRITICAL/HIGH/LOW ratings | Educational patterns with "when to use" guidance |
| **Examples** | Incorrect/Correct pairs | Avoid/Prefer pairs with context on *why* |

## Framework Coverage

### Vercel: Next.js-Centric
- Server-side rules assume Next.js App Router (`next/after`, `next/dynamic`, route handlers)
- Bundle rules reference `optimizePackageImports` (Next.js config)
- Caching patterns use `unstable_cache` and Next.js fetch memoization
- Server Actions with `'use server'` directive
- Turbopack-specific optimizations

### patterns.dev: React + Vite First, Framework-Flexible
- Data fetching patterns use TanStack Query and SWR (work everywhere)
- Bundle optimization uses Vite config, Rollup `manualChunks`, `vite-plugin-barrel`
- Code splitting uses standard `React.lazy()` + `Suspense` (not `next/dynamic`)
- SSR patterns work with any SSR setup (Vite SSR, Remix, Next.js)
- Dedicated **vite-bundle-optimization** skill for Vite-specific configuration
- All React patterns work regardless of meta-framework choice

## Content Coverage Comparison

### Areas Where Both Compete (Different Approaches)

| Topic | Vercel Approach | patterns.dev Approach |
|-------|----------------|----------------------|
| **Re-render optimization** | 12 individual rule files in `react-best-practices` | Unified `react-render-optimization` skill with 15 patterns |
| **Composition patterns** | Separate `composition-patterns` skill (8 rules) | `react-composition-2026` skill (10 patterns) + existing `compound-pattern`, `render-props-pattern`, `hooks-pattern` |
| **Data fetching** | Split across `async-*`, `server-*`, `client-*` rules | Unified `react-data-fetching` skill covering client + server |
| **Bundle optimization** | `bundle-*` rules (Next.js focus) | `vite-bundle-optimization` skill + existing `bundle-splitting`, `tree-shaking`, `dynamic-import` skills |
| **JS performance** | `js-*` rules (12 micro-patterns) | `js-performance-patterns` skill (12 patterns) |

### Areas Unique to patterns.dev

| Skill | Description |
|-------|-------------|
| **27 JavaScript design/performance patterns** | Singleton, Observer, Proxy, Factory, Module, Command, Flyweight, Mediator, Mixin, Provider, Prototype patterns. Plus loading-sequence, static-import, dynamic-import, import-on-visibility, import-on-interaction, route-based splitting, PRPL, preload, prefetch, compression, virtual-lists |
| **8 Rendering strategy skills** | CSR, SSR, SSG, ISR, Streaming SSR, Progressive Hydration, Selective Hydration, React Server Components — each as a dedicated educational skill |
| **11 Vue.js skills** | Components, composables, script-setup, state-management, provide-inject, dynamic-components, async-components, render-functions, renderless-components, container-presentational, data-provider |
| **AI UI Patterns** | Design patterns for building AI-powered interfaces (chatbots, streaming, prompt management) with both Next.js and Vite examples |
| **React 2026 Stack Guide** | Comprehensive ecosystem guide covering build tools, frameworks, routing, state management, React 19 APIs, and Vite best practices |
| **Islands Architecture** | Progressive enhancement pattern for content-heavy sites |
| **View Transitions** | Modern View Transitions API pattern |
| **HOC Pattern** | Higher-Order Components — still relevant for cross-cutting concerns |
| **Presentational/Container** | Separation of concerns pattern |

### Areas Unique to Vercel

| Skill | Description |
|-------|-------------|
| **Web Design Guidelines** | UI review against 100+ accessibility/UX rules (fetches from external repo) |
| **Next.js server-specific rules** | `next/after` for non-blocking operations, route handler auth, `unstable_cache` |
| **React Native Guidelines** | Mobile-specific patterns (not in patterns.dev agent skills) |
| **Build system** | TypeScript-based build pipeline that compiles rules into AGENTS.md |

## Key Differentiators

### 1. Vite-First Bundle Optimization
Vercel's bundle rules reference Next.js-specific APIs (`optimizePackageImports`, `next/dynamic`). Our **vite-bundle-optimization** skill covers:
- `vite-plugin-barrel` for barrel file transformation
- Rollup `manualChunks` for vendor splitting
- `optimizeDeps` configuration for dev server performance
- `vite-plugin-compression` for Gzip/Brotli
- `vite-bundle-visualizer` for analysis
- `import.meta.env` for dead code elimination

### 2. TanStack Query Over SWR as Primary
Vercel favors SWR (their own library). We recommend **TanStack Query** as the primary choice for Vite apps because:
- More features (optimistic updates, infinite queries, query invalidation)
- Better DevTools
- Larger ecosystem
- First-class Suspense support (`useSuspenseQuery`)

We still cover SWR as a valid alternative.

### 3. Educational Depth Over Rule Density
Vercel packs 57 rules into one skill with minimal explanation (a few lines per rule). Our approach uses dedicated skills with:
- "When to Use" context
- Detailed explanations of *why* each pattern matters
- Multiple code examples showing progression
- Cross-references to related skills
- Framework-agnostic framing

### 4. Design Patterns Foundation
patterns.dev includes foundational software design patterns (Singleton, Observer, Factory, etc.) that Vercel doesn't cover. These patterns inform architectural decisions across any framework.

### 5. Rendering Strategy Education
We provide 8 dedicated skills explaining different rendering approaches (CSR, SSR, SSG, ISR, Streaming, Hydration strategies, RSC). Vercel assumes you're already on Next.js and focuses on optimizing within that context.

### 6. Vue.js Coverage
11 Vue skills covering core patterns. Vercel has no Vue coverage.

### 7. Broader Composition Coverage
Our composition skill includes patterns Vercel doesn't cover:
- Polymorphic `as` props for flexible elements
- Slot pattern for layout components
- Headless components (hooks-based behavior without markup)
- Context interface pattern for swappable state implementations

Vercel's composition skill focuses more narrowly on boolean prop elimination and compound components.

### 8. React 19 Coverage
Our `react-2026` skill and `react-composition-2026` skill cover React 19 APIs including:
- `ref` as regular prop (no `forwardRef`)
- `use()` for context and promises
- `useActionState` for forms
- `useOptimistic` for instant feedback
- React Compiler for auto-memoization

Vercel covers `forwardRef` removal but not the broader React 19 API surface.

## Skill Count Comparison

| Category | Vercel | patterns.dev |
|----------|--------|-------------|
| React performance | 1 (57 rules) | 2 skills (render optimization + data fetching) |
| React composition | 1 (8 rules) | 4 skills (composition-2026 + compound + render-props + hooks) |
| React patterns | — | 7 skills (HOC, presentational/container, AI UI, react-2026, etc.) |
| React rendering | — | 8 skills (CSR, SSR, SSG, ISR, streaming, hydration, RSC) |
| JS patterns | Embedded in react skill (12 rules) | 29 skills (design patterns + performance + bundle + Vite) |
| Vue | — | 11 skills |
| **Total** | **3 skills** | **~58 skills** |

## Summary

patterns.dev skills serve a broader audience building React applications across different stacks — Vite, Next.js, Remix, or custom setups. The educational approach (individual focused skills with context) differs from Vercel's prescriptive approach (many rules in few skills). The biggest competitive advantages are: Vite-first optimization, rendering strategy education, design pattern foundations, Vue coverage, and framework-agnostic React guidance that doesn't assume any specific meta-framework.
