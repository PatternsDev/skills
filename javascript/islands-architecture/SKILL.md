---
name: islands-architecture
description: Render small, focused chunks of interactivity within server-rendered web pages to reduce JavaScript overhead.
context: fork
allowed-tools: Read, Grep, Glob
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

# Islands Architecture

> tl;dr: The islands architecture encourages small, focused chunks of interactivity within server-rendered web pages. The output of islands is progressively enhanced HTML, with more specificity around how the enhancement occurs. Rather than a single application being in control of full-page rendering, there are multiple entry points. The script for these "islands" of interactivity can be delivered and hydrated independently, allowing the rest of the page to be just static HTML.

## When to Use

- Use this for primarily static websites that need sprinkles of interactivity (blogs, product pages, news sites)
- This is helpful when you want to reduce the volume of JavaScript shipped to the client
- Use this when SEO and fast initial page loads are priorities alongside selective interactivity

## Instructions

- Identify static and dynamic regions of each page separately
- Use frameworks like Astro, Marko, or Eleventy that support islands architecture
- Hydrate interactive components independently using `client:visible` or similar directives
- Keep the majority of the page as static HTML with zero JavaScript cost

## Details

Loading and processing excess JavaScript can hurt performance. However, some degree of interactivity and JavaScript is often required, even in primarily static websites. We have discussed variations of Static Rendering that enable you to build applications that try to find the balance between:

1. Interactivity comparable to Client-Side Rendered (CSR) applications
2. SEO benefits that are comparable to SSR applications.

The core principle for SSR is that HTML is rendered on the server and shipped with necessary JavaScript to rehydrate it on the client. Rehydration is the process of regenerating the state of UI components on the client-side after the server renders it. Since rehydration comes at a [cost](https://addyosmani.com/blog/rehydration/), each variation of SSR tries to optimize the rehydration process. This is mainly achieved by partial hydration of critical components or streaming of components as they get rendered. However, the net JavaScript shipped eventually in the above techniques remains the same.

The term [Islands architecture](https://jasonformat.com/islands-architecture/) was popularized by Katie Sylor-Miller and Jason Miller to describe a paradigm that aims to reduce the volume of JavaScript shipped through "islands" of interactivity that can be independent delivered on top of otherwise static HTML. Islands are a component-based architecture that suggests a compartmentalized view of the page with static and dynamic islands. The static regions of the page are pure non-interactive HTML and do not need hydration. The dynamic regions are a combination of HTML and scripts capable of rehydrating themselves after rendering.

### Islands of dynamic components

Most pages are a combination of static and dynamic content. Usually, a page consists of static content with sprinkles of interactive regions that can be isolated. For example:

1. Blog posts, news articles, and organization home pages contain text and images with interactive components like social media embeds and chat.
2. Product pages on e-commerce sites contain static product descriptions and links to other pages on the app. Interactive components such as image carousels and search are available in different regions of the page.
3. A typical bank account details page contains a list of static transactions with filters providing some interactivity.

Static content is stateless, does not fire events, and does not need rehydration after rendering. After rendering, dynamic content (buttons, filters, search bar) has to be rewired to its events. The DOM has to be regenerated on the client-side (virtual DOM). This regeneration, rehydration, and event handling functions contribute to the JavaScript sent to the client.

The Islands architecture facilitates server-side rendering of pages with all of their static content. However, in this case, the rendered HTML will include placeholders for dynamic content. The dynamic content placeholders contain self-contained component widgets. Each widget is similar to an app and combines server-rendered output and JavaScript used to hydrate the app on the client.

In progressive hydration, the hydration architecture of the page is top-down. The page controls the scheduling and hydration of individual components. Each component has its hydration script in the Islands architecture that executes asynchronously, independent of any other script on the page. A performance issue in one component should not affect the other.

### Implementing Islands

The Island architecture borrows concepts from different sources and aims to combine them optimally. Template-based static site generators such as [Jekyll](https://jekyllrb.com/) and [Hugo](https://gohugo.io/) support the rendering of static components to pages. Most modern JavaScript frameworks also support [isomorphic rendering](https://en.wikipedia.org/wiki/Isomorphic_JavaScript), which allows you to use the same code to render elements on the server and client.

Jason's post suggests the use of [`requestIdleCallback()`](https://developer.mozilla.org/en-US/docs/Web/API/Window/requestIdleCallback) to implement a scheduling approach for hydrating components. Static isomorphic rendering and scheduling of component level partial hydration can be built into a framework to support Islands architecture. Thus, the framework should:

1. Support static rendering of pages on the server with zero JavaScript.
2. Support embed of independent dynamic components via placeholders in static content. Each dynamic component contains its scripts and can hydrate itself using requestIdleCallback() as soon as the main thread is free.
3. Allow isomorphic rendering of components on the server with hydration on the client to recognize the same component at both ends.

#### Frameworks

Different frameworks today are capable of supporting the Islands architecture. Notable among them are:

1. **Marko**: [Marko](https://markojs.com/) is an open-source framework [developed](https://tech.ebayinc.com/engineering/the-future-of-marko/) and [maintained](https://tech.ebayinc.com/engineering/ebay-launches-marko-5/) by eBay to improve server rendering performance. It supports Islands architecture by combining streaming rendering with automatic partial hydration. HTML and other static assets are streamed to the client as soon as they are ready. Automatic partial hydration allows interactive components to hydrate themselves. Hydration code is only [shipped for interactive components](https://medium.com/@mlrawlings/maybe-you-dont-need-that-spa-f2c659bc7fec), which can change the state on the browser. It is isomorphic, and the Marko compiler generates optimized code depending on where it will run (client or server).
2. **Astro**: [Astro](https://astro.build/) is a static site builder that can generate lightweight static HTML pages from UI components built in other frameworks such as React, Preact, Svelte, Vue, and others. Components that need client-side JavaScript are loaded individually with their dependencies. Thus it provides built-in partial hydration. Astro can also lazy-load components depending on when they become visible.
3. **Eleventy + Preact:** [Markus Oberlehner](https://markus.oberlehner.net/blog/building-partially-hydrated-progressively-enhanced-static-websites-with-isomorphic-preact-and-eleventy/#lazy-hydration) demonstrates the use of Eleventy, a static site generator with isomorphic Preact components that can be partially hydrated. It also supports lazy hydration. The component itself declaratively controls the hydration of the component. Interactive components use a `WithHydration` wrapper so that they are hydrated on the client.

Note that Marko and Eleventy pre-date the definition of Islands provided by Jason but contain some of the features required to support it. **Astro**, however, was built based on the definition and inherently supports the Islands architecture.

#### Sample implementation

The following is a sample blog page implemented using Astro. The page SamplePost imports one interactive component, SocialButtons. This component is included in the HTML at the required position via markup.

```astro
// Component Imports
import { SocialButtons } from '../../components/SocialButtons.js';

<html lang="en">
 <head>
   <link rel="stylesheet" href="/blog.css" />
 </head>

 <body>
   <div class="layout">
     <article class="content">
       <section class="intro">
         <h1 class="title">Post title (static)</h1>
         <br/>
         <p>Post sub-title (static)</p>
       </section>
       <section class="intro">
           <p>This is the post content with images that is rendered by the server.</p>
           <p>The next section contains the interactive social buttons component which includes its script.</p>
       </section>
       <section class="social">
           <div>
           <SocialButtons client:visible></SocialButtons>
           </div>
       </section>
     </article>
   </div>
 </body>
</html>
```

```js
// SocialButtons.js
import { useState } from "preact/hooks";

export function SocialButtons() {
  const [count, setCount] = useState(0);
  const add = () => setCount((i) => i + 1);
  const subtract = () => setCount((i) => i - 1);
  return (
    <>
      <div>{count} people liked this post</div>
      <div align="right">
        <button onclick={add}>Like</button>
        <button onclick={subtract}>Unlike</button>
      </div>
    </>
  );
}
```

The `SocialButtons` component is a Preact component with its HTML, and corresponding event handlers included.

The component is embedded in the page at run time and hydrated on the client-side so that the click events function as required.

Astro allows for a clean separation between HTML, CSS, and scripts and encourages component-based design. It is easy to install and start building websites with this framework.

### Pros and Cons

The Islands architecture combines ideas from different rendering techniques such as server-side rendering, static site generation, and partial hydration. Some of the potential benefits of implementing islands are as follows.

1. **Performance**: Reduces the amount of JavaScript code shipped to the client. The code sent only consists of the script required for interactive components, which is much less than the script needed to recreate the virtual DOM for the entire page and rehydrate all the elements on the page. The smaller size of JavaScript automatically corresponds to faster page loads and Time to Interactive (TTI).

> Comparisons for Astro with documentation websites created for Next.js and Nuxt.js have shown an 83% reduction in JavaScript code. Other users have also reported performance improvements with Astro.

2. **SEO:** Since all of the static content is rendered on the server; pages are SEO friendly.
3. **Prioritizes important content:** Key content (especially for blogs, news articles, and product pages) is available almost immediately to the user. Secondary functionality for interactivity is usually required after consuming the key content becomes available gradually.
4. **Accessibility:** The use of standard static HTML links to access other pages helps to improve the accessibility of the website.
5. **Component-based:** The architecture offers all advantages of component-based architecture, such as reusability and maintainability.

Despite the advantages, the concept is still in a nascent stage. The limited support results in some disadvantages.

1. The only options available to developers to implement Islands are to use one of the few frameworks available or develop the architecture yourself. Migrating existing sites to Astro or Marko would require additional efforts.
2. Besides Jason's initial post, there is little discussion available on the idea.
3. [New frameworks](https://github.com/bensmithett/tropical-utils/tree/main/packages/tropical-islands) claim to support the Islands architecture making it difficult to filter the ones which will work for you.
4. The architecture is not suitable for highly interactive pages like social media apps which would probably require thousands of islands.

The Islands architecture concept is relatively new but likely to gain speed due to its performance advantages. It underscores the use of SSR for rendering static content while supporting interactivity through dynamic components with minimal impact on the page's performance. We hope to see many more players in this space in the future and have a wider choice of implementation options available.

### Further reading

- [Islands Architecture](https://jasonformat.com/islands-architecture/)
- [Is 0KB of JavaScript in your future](https://dev.to/this-is-learning/is-0kb-of-javascript-in-your-future-48og)
- [Modernizing Etsy's codebase with React](https://changelog.com/jsparty/105)

## Source

- [patterns.dev/vanilla/islands-architecture](https://patterns.dev/vanilla/islands-architecture)
