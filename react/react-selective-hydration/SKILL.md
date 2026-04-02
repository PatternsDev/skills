---
name: react-selective-hydration
description: Combine streaming server-side rendering with selective hydration for faster interactivity in React 18+.
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

# Selective Hydration

In previous articles, we covered how SSR with hydration can improve user experience. React is able to (quickly) generate a tree on the server using the `renderToString` method that the `react-dom/server` library provides, which gets sent to the client after the entire tree has been generated. The rendered HTML is non interactive, until the JavaScript bundle has been fetched and loaded, after which React walks down the tree to hydrate and attaches the handlers.

However, this approach can lead to some performance issues due to some limitations with the current implementation.

## When to Use

- Use this when you want to make parts of your SSR page interactive before all JavaScript has loaded
- This is helpful when slow components (e.g., data-fetching components) are blocking the entire page's hydration

## Instructions

- Use `Suspense` boundaries to delineate independently hydratable chunks of UI
- Use `renderToPipeableStream` (Node) or `renderToReadableStream` (edge) for streaming SSR
- Place heavy data-fetching components inside `Suspense` so they don't delay sibling hydration
- Ensure critical interactive components are not inside long-lived loading fallbacks
- Use `hydrateRoot` (React 18+) to benefit from selective hydration

## Details

Before the server-rendered HTML tree is able to get sent to the client, all components need to be ready. This means that components that may rely on an external API call or any process that could cause some delays, might end up blocking smaller components from being rendered quickly.

Besides a slower tree generation, another issue is the fact that React only hydrates the tree once. This means that before React is able to hydrate any of the components, it needs to have fetched the JavaScript for all of the components before it's able to hydrate any of them. This means that smaller components (with smaller bundles) have to wait for the larger components's code to be fetched and loaded, until React is able to hydrate anything on your website. During this time, the website remained non-interactive.

React 18 solves these problems by allowing us to combine streaming server-side rendering with a new approach to hydration: Selective Hydration!

Instead of using `renderToString`, modern React SSR uses `renderToPipeableStream()` on Node runtimes or `renderToReadableStream()` on Web Stream runtimes.

These APIs, in combination with `hydrateRoot()` and `Suspense`, make it possible to start streaming HTML without having to wait for the larger components to be ready. This means that we can lazy-load components when using SSR without blocking the hydration of the rest of the page.

The `Comments` component, which earlier slowed down the tree generation and TTI, is now wrapped in `Suspense`. This tells React to not let this component slow down the rest of the tree generation. Instead, React inserts the fallback components as the initially rendered HTML, and continues to generate the rest of the tree before it's sent to the client.

In the meantime, we're still fetching the external data that we need for the `Comments` component.

Selective hydration makes it possible to already hydrate the components that were sent to the client, even before the `Comments` component has been sent!

Once the data for the `Comments` component is ready, React starts streaming the HTML for this component, as well as a small `<script>` to replace the fallback loader.

React starts the hydration after the new HTML has been injected.

React 18 fixes some issues that people often encountered when using SSR with React.

Streaming rendering allows you to start streaming components as soon as they're ready, without risking a slower FCP and TTI due to components that might take longer to generate on the server.

Components can be hydrated as soon as they're streamed to the client, since we no longer have to wait for all JavaScript to load to start hydrating and can start interacting with the app before all components have been hydrated.

> **Note (React 18+): Best Practices for Selective Hydration**
>
> Use Suspense boundaries in your SSR code to delineate independent chunks of UI—each Suspense boundary can hydrate independently. Structure your code so that any heavy data-fetching component is inside a Suspense, so it doesn't delay the initial HTML or hydration of siblings. Next.js automatically wraps each route segment in a Suspense boundary for you (via `loading.js`).
>
> If writing your own SSR, use `renderToPipeableStream()` (or `renderToReadableStream`) instead of the older `renderToNodeStream`, and provide an `onShellReady` callback to flush early HTML. React will attach event listeners progressively as chunks come in.
>
> **Ensure critical interactive components are not inside a loading fallback** when they appear on screen, or if they are, that the fallback is very short-lived. If a user sees a button, it should be hydratable immediately. Verify you're not using legacy APIs—use `hydrateRoot` correctly for React 18+ to benefit from these performance boosts.

## Source

- [patterns.dev/react/react-selective-hydration](https://patterns.dev/react/react-selective-hydration)

### References

- [New Suspense SSR Architecture in React 18](https://github.com/reactwg/react-18/discussions/37)
