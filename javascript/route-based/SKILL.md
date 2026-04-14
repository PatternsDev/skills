---
name: route-based
description: Teaches route-based code splitting for single-page applications. Use when different routes load distinct feature sets and you want to avoid loading all route code upfront.
paths:
  - "**/*.js"
  - "**/*.ts"
license: MIT
metadata:
  author: patterns.dev
  version: "1.1"
related_skills:
  - "module-pattern"
  - "singleton-pattern"
---

# Route Based Splitting

We can request resources that are only needed for specific routes, by adding _route-based splitting_. By combining **React Suspense** or `loadable-components` with libraries such as `react-router`, we can dynamically load components based on the current route.

By lazily loading the components per route, we're only requesting the bundle that contains the code that's necessary for the current route. Since most people are used to the fact that there may be some loading time during a redirect, it's the perfect place to lazily load components!

## When to Use

- Use this when your application has multiple routes and not all code is needed on every page
- This is helpful for reducing initial load time by only loading code for the current route

## Instructions

- Combine React Suspense or `loadable-components` with routing libraries like `react-router`
- Lazily load page-level components per route for optimal code splitting
- Take advantage of natural loading pauses during route transitions for a seamless experience

## Details

By lazily loading the components per route, only the bundle containing code necessary for the current route is requested. Since users expect some loading time during navigation, route transitions are a natural place to introduce code splitting without degrading the experience.

Most modern frameworks (Next.js, Remix, React Router) support route-based splitting out of the box. In a custom setup, use `React.lazy()` with `Suspense` to wrap route-level components, or configure your bundler's entry points per route.

## Source

- [patterns.dev/vanilla/route-based](https://patterns.dev/vanilla/route-based)
