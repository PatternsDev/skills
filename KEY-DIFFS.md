# Positioning: patterns.dev Skills vs Vercel Skills

This document explains how the patterns.dev skills are positioned relative to Vercel's skills. It is intended as a scope and philosophy guide, not as a point-in-time feature benchmark.

Read this file if you want to understand:

- why this repo is structured as many focused skills instead of a few large ones
- which frameworks and problem spaces it emphasizes
- how it differs in teaching style and technical scope

## Philosophy

| Dimension             | Vercel Skills                                     | patterns.dev Skills                                               |
| --------------------- | ------------------------------------------------- | ----------------------------------------------------------------- |
| **Primary framework** | Next.js first                                     | Framework-agnostic, with a strong Vite + React emphasis           |
| **Audience**          | Next.js/Vercel platform users                     | Broader React web engineering community                           |
| **Structure**         | Monolithic skills (57 rules in one skill)         | Individual focused skills, one pattern per skill                  |
| **Scope**             | React + Next.js performance                       | React + JS patterns, rendering, design patterns, Vue, performance |
| **Tone**              | Prescriptive rules with CRITICAL/HIGH/LOW ratings | Educational patterns with "when to use" guidance                  |
| **Examples**          | Incorrect/Correct pairs                           | Avoid/Prefer pairs with context on _why_                          |

## Framework Coverage

### Vercel: Next.js-Centric

- Server-side rules assume Next.js App Router (`next/after`, `next/dynamic`, route handlers)
- Bundle rules reference `optimizePackageImports` (Next.js config)
- Caching patterns use `unstable_cache` and Next.js fetch memoization
- Server Actions with `'use server'` directive
- Turbopack-specific optimizations

### patterns.dev: React-Flexible, Often Vite-Oriented

- Data fetching patterns use TanStack Query and SWR in framework-agnostic ways
- Bundle optimization uses Vite config, Rollup `manualChunks`, `vite-plugin-barrel`
- Code splitting uses standard `React.lazy()` + `Suspense` rather than Next.js-specific APIs
- SSR patterns are presented in ways that transfer across Vite SSR, Remix, and Next.js
- There is a dedicated [`vite-bundle-optimization`](javascript/vite-bundle-optimization/SKILL.md) skill for Vite-specific configuration
- Most React patterns are written to remain useful regardless of meta-framework choice

## Content Coverage Comparison

### Areas Where Both Compete (Different Approaches)

| Topic                      | Vercel Approach                                      | patterns.dev Approach                                                                                                                                                                                                                                                |
| -------------------------- | ---------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Re-render optimization** | 12 individual rule files in `react-best-practices`   | Unified [`react-render-optimization`](react/react-render-optimization/SKILL.md) skill with multiple patterns                                                                                                                                                         |
| **Composition patterns**   | Separate `composition-patterns` skill (8 rules)      | [`react-composition-2026`](react/react-composition-2026/SKILL.md) plus focused skills like [`compound-pattern`](react/compound-pattern/SKILL.md), [`render-props-pattern`](react/render-props-pattern/SKILL.md), and [`hooks-pattern`](react/hooks-pattern/SKILL.md) |
| **Data fetching**          | Split across `async-*`, `server-*`, `client-*` rules | Unified [`react-data-fetching`](react/react-data-fetching/SKILL.md) skill covering client and server patterns                                                                                                                                                        |
| **Bundle optimization**    | `bundle-*` rules (Next.js focus)                     | [`vite-bundle-optimization`](javascript/vite-bundle-optimization/SKILL.md) plus [`bundle-splitting`](javascript/bundle-splitting/SKILL.md), [`tree-shaking`](javascript/tree-shaking/SKILL.md), and [`dynamic-import`](javascript/dynamic-import/SKILL.md)           |
| **JS performance**         | `js-*` rules (12 micro-patterns)                     | [`js-performance-patterns`](javascript/js-performance-patterns/SKILL.md)                                                                                                                                                                                             |

### Areas Unique to patterns.dev

| Skill                                          | Description                                                                                                                                                                                                                                                                                                                                                                          |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **JavaScript design and performance patterns** | Foundational patterns like Singleton, Observer, Proxy, Factory, Module, Command, Flyweight, Mediator, Mixin, Provider, Prototype, plus performance-focused skills such as [`loading-sequence`](javascript/loading-sequence/SKILL.md), [`preload`](javascript/preload/SKILL.md), [`prefetch`](javascript/prefetch/SKILL.md), and [`virtual-lists`](javascript/virtual-lists/SKILL.md) |
| **Rendering strategy skills**                  | Dedicated React skills for CSR, SSR, SSG, ISR, Streaming SSR, Progressive Hydration, Selective Hydration, and React Server Components                                                                                                                                                                                                                                                |
| **Vue.js skills**                              | Coverage for components, composables, script setup, state management, provide/inject, dynamic components, async components, render functions, renderless components, container/presentational, and data-provider patterns                                                                                                                                                            |
| **AI UI patterns**                             | [`ai-ui-patterns`](react/ai-ui-patterns/SKILL.md) for interfaces such as chat, streaming, and AI-specific interaction flows                                                                                                                                                                                                                                                          |
| **Modern React stack guidance**                | [`react-2026`](react/react-2026/SKILL.md) for ecosystem-level decisions across tooling, routing, state, and rendering                                                                                                                                                                                                                                                                |
| **Additional web platform topics**             | Skills like [`islands-architecture`](javascript/islands-architecture/SKILL.md) and [`view-transitions`](javascript/view-transitions/SKILL.md) broaden scope beyond component patterns alone                                                                                                                                                                                          |

### Areas Unique to Vercel

| Skill                             | Description                                                                    |
| --------------------------------- | ------------------------------------------------------------------------------ |
| **Web Design Guidelines**         | UI review against 100+ accessibility/UX rules (fetches from external repo)     |
| **Next.js server-specific rules** | `next/after` for non-blocking operations, route handler auth, `unstable_cache` |
| **React Native Guidelines**       | Mobile-specific patterns (not in patterns.dev agent skills)                    |
| **Build system**                  | TypeScript-based build pipeline that compiles rules into AGENTS.md             |

## Distinctives

### 1. Vite-First Bundle Optimization

Vercel's bundle rules reference Next.js-specific APIs (`optimizePackageImports`, `next/dynamic`). This repo includes a dedicated [`vite-bundle-optimization`](javascript/vite-bundle-optimization/SKILL.md) skill covering:

- `vite-plugin-barrel` for barrel file transformation
- Rollup `manualChunks` for vendor splitting
- `optimizeDeps` configuration for dev server performance
- `vite-plugin-compression` for Gzip/Brotli
- `vite-bundle-visualizer` for analysis
- `import.meta.env` for dead code elimination

### 2. TanStack Query Over SWR as Primary

Vercel favors SWR (their own library). This repo often uses **TanStack Query** as the primary recommendation for custom React stacks because:

- More features (optimistic updates, infinite queries, query invalidation)
- Better DevTools
- Larger ecosystem
- First-class Suspense support (`useSuspenseQuery`)

We still cover SWR as a valid alternative.

### 3. Educational Depth Over Rule Density

Vercel packs many rules into a smaller number of skills with short explanations. This repo uses dedicated skills with:

- "When to Use" context
- Detailed explanations of _why_ each pattern matters
- Multiple code examples showing progression
- Cross-references to related skills
- Framework-agnostic framing

### 4. Design Patterns Foundation

patterns.dev includes foundational software design patterns (Singleton, Observer, Factory, etc.) that Vercel doesn't cover. These patterns inform architectural decisions across any framework.

### 5. Rendering Strategy Education

This repo provides dedicated skills explaining different rendering approaches (CSR, SSR, SSG, ISR, streaming, hydration strategies, RSC). Vercel assumes you're already on Next.js and focuses on optimizing within that context.

### 6. Vue.js Coverage

11 Vue skills covering core patterns. Vercel has no Vue coverage.

### 7. Broader Composition Coverage

The composition material in this repo includes patterns Vercel doesn't emphasize:

- Polymorphic `as` props for flexible elements
- Slot pattern for layout components
- Headless components (hooks-based behavior without markup)
- Context interface pattern for swappable state implementations

Vercel's composition skill focuses more narrowly on boolean prop elimination and compound components.

### 8. React 19 Coverage

The [`react-2026`](react/react-2026/SKILL.md) and [`react-composition-2026`](react/react-composition-2026/SKILL.md) skills cover modern React APIs including:

- `ref` as regular prop (no `forwardRef`)
- `use()` for context and promises
- `useActionState` for forms
- `useOptimistic` for instant feedback
- React Compiler for auto-memoization

Vercel covers `forwardRef` removal but not the broader React 19 API surface.

## Skill Count Comparison

| Category          | Vercel                             | patterns.dev                                                      |
| ----------------- | ---------------------------------- | ----------------------------------------------------------------- |
| React performance | 1 (57 rules)                       | 2 skills (render optimization + data fetching)                    |
| React composition | 1 (8 rules)                        | 4 skills (composition-2026 + compound + render-props + hooks)     |
| React patterns    | —                                  | 7 skills (HOC, presentational/container, AI UI, react-2026, etc.) |
| React rendering   | —                                  | 8 skills (CSR, SSR, SSG, ISR, streaming, hydration, RSC)          |
| JS patterns       | Embedded in react skill (12 rules) | 29 skills (design patterns + performance + bundle + Vite)         |
| Vue               | —                                  | 11 skills                                                         |
| **Total**         | **3 skills**                       | **~58 skills**                                                    |

## Summary

patterns.dev skills serve a broader audience building React applications across different stacks — Vite, Next.js, Remix, or custom setups. The educational approach (individual focused skills with context) differs from Vercel's prescriptive approach (many rules in few skills). The biggest competitive advantages are: Vite-first optimization, rendering strategy education, design pattern foundations, Vue coverage, and framework-agnostic React guidance that doesn't assume any specific meta-framework.
