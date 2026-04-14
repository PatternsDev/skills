---
name: import-on-visibility
description: Teaches visibility-based lazy loading using Intersection Observer. Use when you have below-the-fold components or images that don't need to load until the user scrolls to them.
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

# Import On Visibility

Besides user interaction, we often have components that aren't visible on the initial page. A good example of this is lazy loading images that aren't directly visible in the viewport, but only get loaded once the user scrolls down.

As we're not requesting all images instantly, we can reduce the initial loading time. We can do the same with components! In order to know whether components are currently in our viewport, we can use the [`IntersectionObserver` API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API), or use libraries such as `react-lazyload` or `react-loadable-visibility` to quickly add import on visibility to our application.

## When to Use

- Use this for components that aren't visible on the initial page (e.g., below-the-fold content)
- This is helpful for lazy loading images, widgets, or heavy components as the user scrolls

## When NOT to Use

- For above-the-fold content that must render immediately — deferring it causes visible layout shifts and slow LCP
- When the component is lightweight enough that lazy loading adds more overhead than it saves
- When the content is critical for SEO and needs to be present in the initial HTML

## Instructions

- Use the `IntersectionObserver` API to detect when components enter the viewport
- Use libraries like `react-lazyload` or `react-loadable-visibility` for quick implementation
- Provide a loading fallback component while the module is being loaded

## Details

Whenever a component is rendered to the screen, `react-loadable-visibility` detects that the element should be visible on the screen. Only then, it will start importing the module while the user sees a loading component being rendered.

This fallback component lets the user know that our application hasn't frozen: they simply need to wait a short while for the module to be loaded, parsed, compiled, and executed!

## Source

- [patterns.dev/vanilla/import-on-visibility](https://patterns.dev/vanilla/import-on-visibility)
