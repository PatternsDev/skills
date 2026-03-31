---
name: static-import
description: Import code that has been exported by another module using static ES2015 import syntax.
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

# Static Import

The `import` keyword allows us to import code that has been exported by another module. By default, all modules we're _statically importing_ get added to the initial bundle. A module that is imported by using the default ES2015 import syntax, `import module from 'module'`, is statically imported.

Let's look at an example! A simple chat app contains a `Chat` component, in which we're statically importing and rendering three components: `UserProfile`, a `ChatList`, and a `ChatInput` to type and send messages! Within the `ChatInput` module, we're statically importing an `EmojiPicker` component to be able to show the user the emoji picker when the user toggles the emoji.

## When to Use

- Use this for importing modules that are needed immediately on page load
- This is the default import mechanism — understand it to know when to switch to dynamic imports

## Instructions

- Use static imports for modules critical to the initial render
- Be aware that all statically imported modules are bundled into the initial bundle
- Consider switching to dynamic imports for modules not needed on initial render

## Details

The modules get executed as soon as the engine reaches the line on which we import them.

Since the components were statically imported, Webpack bundled the modules into the initial bundle:

```bash
Asset           Size      Chunks            Chunk Names
main.bundle.js  1.5 MiB    main  [emitted]  main
```

Our chat application's source code gets bundled into one bundle: `main.bundle.js`. A large bundle size can affect the loading time of our application significantly depending on the user's device and network connection. Before the `App` component is able to render its contents to the user's screen, it first has to load and parse all modules.

Luckily, there are many ways to speed up the loading time! We don't always have to import all modules at once: maybe there are some modules that should only get rendered based on user interaction, like the `EmojiPicker` in this case, or rendered further down the page. Instead of importing all component statically, we can _dynamically_ import the modules after the `App` component has rendered its contents and the user is able to interact with our application.

## Source

- [patterns.dev/vanilla/static-import](https://patterns.dev/vanilla/static-import)
