---
name: dynamic-import
description: Import parts of your code on demand using dynamic import() to reduce initial bundle size.
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

# Dynamic Import

In our chat application, we have four key components: `UserInfo`, `ChatList`, `ChatInput` and `EmojiPicker`. However, only _three_ of these components are used instantly on the initial page load: `UserInfo`, `ChatList` and `ChatInput`. The `EmojiPicker` isn't directly visible, and may not even be rendered at all if the user won't even click on the `Emoji` in order to toggle the `EmojiPicker`. This would mean that we unnecessarily added the `EmojiPicker` module to our initial bundle, which potentially increased the loading time!

In order to solve this, we can _dynamically import_ the `EmojiPicker` component. Instead of statically importing it, we'll only import it when we want to show the `EmojiPicker`. An easy way to dynamically import components in React is by using **React Suspense**. The `React.Suspense` component receives the component that should be dynamically loaded, which makes it possible for the `App` component to render its contents faster by suspending the import of the `EmojiPicker` module!

## When to Use

- Use this when certain modules are only needed based on user interaction or conditions
- This is helpful when you want to reduce the initial bundle size for faster page loads
- Use this when components like modals, pickers, or heavy libraries aren't needed on initial render

## Instructions

- Use `React.lazy` with `Suspense` for dynamic component imports in React
- Provide meaningful fallback UI while dynamically imported modules are loading
- Consider using `loadable-components` for SSR applications where React Suspense isn't supported
- Only dynamically import modules that aren't critical to the initial render

## Details

Instead of unnecessarily adding `EmojiPicker` to the initial bundle, we can split it up into its own bundle and reduce the size of the initial bundle!

A smaller initial bundle size means a faster initial load: the user doesn't have to stare at a blank loading screen for as long. The `fallback` component lets the user know that our application hasn't frozen: they simply need to wait a little while for the module to be processed and executed.

```
Asset                             Size         Chunks            Chunk Names
emoji-picker.bundle.js           1.48 KiB      1    [emitted]    emoji-picker
main.bundle.js                   1.33 MiB      main [emitted]    main
vendors~emoji-picker.bundle.js   171 KiB       2    [emitted]    vendors~emoji-picker
```

Whereas previously the initial bundle was `1.5MiB`, we've been able to reduce it to `1.33 MiB` by suspending the import of the `EmojiPicker`!

By dynamically importing the `EmojiPicker` component, we managed to reduce the initial bundle size from `1.5MiB` to `1.33 MiB`! Although the user may still have to wait a while until the `EmojiPicker` has been fully loaded, we have improved the user experience by making sure the application is rendered and interactive while the user waits for the component to load.

### Loadable Components

Server-side rendering doesn't support React Suspense (yet). A good alternative to React Suspense is the [`loadable-components`](https://loadable-components.com/docs/getting-started/) library, which can be used in SSR applications.

Similar to React Suspense, we can pass the lazily imported module to the `loadable`, which will only import the module once the `EmojiPicker` module is being requested! While the module is being loaded, we can render a `fallback` component.

Although loadable components are a great alternative to React Suspense for SSR applications, they're also useful in CSR applications in order to suspend the import of modules.

## Source

- [patterns.dev/vanilla/dynamic-import](https://patterns.dev/vanilla/dynamic-import)
