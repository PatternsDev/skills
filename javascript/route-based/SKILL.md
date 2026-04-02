---
name: route-based
description: Dynamically load components based on the current route to reduce initial bundle size.
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

## Source

- [patterns.dev/vanilla/route-based](https://patterns.dev/vanilla/route-based)
