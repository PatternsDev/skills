---
name: react-server-components
description: Render components on the server without sending their JavaScript to the client, dramatically reducing bundle sizes.
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

# React Server Components

React's [Server Components](https://react.dev/reference/rsc/server-components) enable **modern UX with a server-driven mental model**. This is quite different from Server-side Rendering (SSR) of components and results in significantly smaller client-side JavaScript bundles.

## When to Use

- Use this when you want to reduce client-side JavaScript by running data-fetching and rendering on the server
- This is helpful for improving performance with zero-JS-cost server-rendered components in Next.js 13+ App Router

## Instructions

- Use Server Components (default in frameworks like Next.js App Router) for data fetching and non-interactive UI
- Add `'use client'` directive only to components that need interactivity (event handlers, state, effects)
- Server Components can use heavy libraries (markdown parsers, date formatters) at zero client bundle cost
- Server Components complement SSR — they are not a replacement for it
- Use Server Functions or Server Actions (`'use server'`) for form submissions and mutations when your framework supports them

## Details

> **Update (React 19 / modern frameworks):** React Server Components are now a production feature for framework users. Unlike classic SSR, RSCs let you render part of your UI on the server without shipping that component's JavaScript to the client. The Container/Presentational pattern is a strong fit here: the "container" can be a Server Component that fetches data and passes it to an interactive Client Component.
>
> In Next.js App Router, you no longer use `getServerSideProps`—instead, any React component in the `app/` directory can be async to fetch data on the server. React Server Components are *not* a replacement for SSR—they complement it. You typically use RSC for the majority of the page (rendered and streamed as part of SSR), and add `'use client'` directives for components that need interactivity.
>
> The `'use server'` directive is for Server Functions or Server Actions, not for marking a component as a Server Component. Server Components have no directive; they are the default in frameworks that support them unless you opt into `'use client'`.

React Server Components are now production-ready in frameworks like Next.js App Router. The following resources are the most useful starting points:

- The [React Server Components reference](https://react.dev/reference/rsc/server-components) is the best current overview.
- The [RFC](https://github.com/reactjs/rfcs/blob/bf51f8755ddb38d92e23ad415fc4e3c02b95b331/text/0188-server-components.md) is still useful for deeper implementation context.
- [Next.js App Router documentation](https://nextjs.org/docs/app) for the modern approach to Server Components
- [Shopify Hydrogen and Server Components](https://shopify.dev/custom-storefronts/hydrogen/framework/react-server-components)

### Server-side rendering limitations

Today's Server-side rendering of client-side JavaScript can be suboptimal. JavaScript for your components is rendered on the server into an HTML string. This HTML is delivered to the browser, which can appear to result in a fast First Contentful Paint or Largest Contentful Paint.

However, JavaScript still needs to be fetched for interactivity which is often achieved via a hydration step. Server-side rendering is generally used for the initial page load, so post-hydration you're unlikely to see it used again.

**Note:** While it's true that one could build a server-only React app leveraging SSR and avoiding hydrating on the client at all, heavy interactivity in the model often involves stepping outside of React. The hybrid model that Server Components enable will allow deciding this on a per-component basis.

With React Server Components, our components can be refetched regularly. An application with components which rerender when there is new data can be run on the server, limiting how much code needs to be sent to the client.

> [RFC]: Developers constantly have to make choices about using third-party packages. Using a package to render some markdown or format a date is convenient for us as developers, but it increases code size and hurts performance for our users

Before Server Components, using a markdown renderer and sanitizer would add to the bundle:

```js
// *Before* Server Components
import marked from "marked"; // 35.9K (11.2K gzipped)
import sanitizeHtml from "sanitize-html"; // 206K (63.3K gzipped)

function NoteWithMarkdown({text}) {
  const html = sanitizeHtml(marked(text));
  return (/* render */);
}
```

### Server Components

React's new Server Components complement Server-side rendering, enabling rendering into an intermediate abstraction format without needing to add to the JavaScript bundle. This both allows merging the server tree with the client tree without losing state and enables scaling up to more components.

Server Components are not a replacement for SSR. When paired together, they support quickly rendering in an intermediate format, then having Server-side rendering infrastructure rendering this into HTML enabling early paints to still be fast. We SSR the Client components which the Server components emit, similar to how SSR is used with other data-fetching mechanisms.

This time however, the JavaScript bundle will be significantly smaller. Early explorations have shown that bundle size wins could be significant (-18-29%), but the React team will have a clearer idea of wins in the wild once further infrastructure work is complete.

> [RFC]: If we migrate the above example to a Server Component we can use the exact same code for our feature but avoid sending it to the client - a code savings of over 240K (uncompressed):

```js
import marked from "marked"; // zero bundle size
import sanitizeHtml from "sanitize-html"; // zero bundle size

function NoteWithMarkdown({text}) {
  // same as before
}
```

### Automatic Code-Splitting

It's been considered a best-practice to only serve code users need as they need it by using code-splitting. This allows you to break your app down into smaller bundles requiring less code to be sent to the client. Prior to Server Components, one would manually use `React.lazy()` to define "split-points" or rely on a heuristic set by a meta-framework, such as routes/pages to create new chunks.

Before Server Components:

```js
// *Before* Server Components
import React from "react";

// one of these will start loading *when rendered on the client*:
const OldPhotoRenderer = React.lazy(() => import("./OldPhotoRenderer.js"));
const NewPhotoRenderer = React.lazy(() => import("./NewPhotoRenderer.js"));

function Photo(props) {
  // Switch on feature flags, logged in/out, type of content, etc:
  if (FeatureFlags.useNewPhotoRenderer) {
    return <NewPhotoRenderer {...props} />;
  } else {
    return <PhotoRenderer {...props} />;
  }
}
```

**Some of the challenges with code-splitting are:**

- Outside of a meta-framework (like Next.js), you often have to tackle this optimization manually, replacing `import` statements with dynamic imports.
- It might delay when the application begins loading the component impacting the user-experience.

Server Components introduce automatic code-splitting treating all normal imports in Client components as possible code-split points. They also allow developers to select which component to use much earlier (on the server), allowing the client to fetch it earlier in the rendering process.

With Server Components:

```js
import React from "react";

// one of these will start loading *once rendered and streamed to the client*:
import OldPhotoRenderer from "./OldPhotoRenderer.client.js";
import NewPhotoRenderer from "./NewPhotoRenderer.client.js";

function Photo(props) {
  // Switch on feature flags, logged in/out, type of content, etc:
  if (FeatureFlags.useNewPhotoRenderer) {
    return <NewPhotoRenderer {...props} />;
  } else {
    return <PhotoRenderer {...props} />;
  }
}
```


### Will Server Components replace Next.js SSR?

No. They are quite different. Initial adoption of Server Components will actually be experimented with via meta-frameworks such as Next.js as research and experimentation continue.

To summarize the differences between Next.js SSR and Server Components from Dan Abramov:

- **Code for Server Components is never delivered to the client.** In many implementations of SSR using React, component code gets sent to the client via JavaScript bundles anyway. This can delay interactivity.
- **Server Components enable access to the back-end from anywhere in the tree.** In older Next.js data APIs, logic often had to live in page-level functions like `getServerSideProps()`. With Server Components, data fetching can live closer to the component that needs it.
- **Server Components may be refetched while maintaining Client-side state inside of the tree.** This is because the main transport mechanism is much richer than just HTML, allowing the refetching of a server-rendered part (e.g such as a search result list) without blowing away state inside (e.g search input text, focus, text selection)

Some of the early integration work for Server Components will be done via a webpack plugin which:

- Locates all Client components
- Creates a mapping between IDs => chunk URLs
- A Node.js loader replaces imports to Client components with references to this map.
- Some of this work will require deeper integrations (e.g with pieces such as Routing) which is why getting this to work with a framework like Next.js will be valuable.

As Dan notes, one of the goals of this work is to enable meta-frameworks to get much better.

### Learn more

To go deeper, read the [React Server Components reference](https://react.dev/reference/rsc/server-components), the [Server Functions reference](https://react.dev/reference/rsc/server-functions), the [RFC](https://github.com/reactjs/rfcs/blob/bf51f8755ddb38d92e23ad415fc4e3c02b95b331/text/0188-server-components.md), and the [server components demo](https://github.com/reactjs/server-components-demo).

## Source

- [patterns.dev/react/react-server-components](https://patterns.dev/react/react-server-components)
