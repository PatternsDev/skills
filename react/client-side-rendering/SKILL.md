---
name: client-side-rendering
description: Render your application's UI entirely on the client using JavaScript.
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

# Client-side Rendering

In Client-Side Rendering (CSR) only the barebones HTML container for a page is rendered by the server. The logic, data fetching, templating and routing required to display content on the page is handled by JavaScript code that executes in the browser/client. CSR became popular as a method of building single-page applications. It helped to blur the difference between websites and installed applications.

## When to Use

- Use this for internal tools, dashboards, or SPAs where SEO is not a priority
- This is helpful when you need a fully interactive single-page application experience

## Instructions

- Keep initial JavaScript bundles small (< 100-170KB minified/gzipped) for fast First Contentful Paint
- Use code-splitting and lazy loading to defer non-critical JavaScript
- Consider SSR/SSG for public-facing pages where SEO and initial load performance matter
- Use service workers and application shell caching for offline and repeat visit performance

## Details

### Basic structure

Consider this simple example for showing and updating the current time on a page using React.

```tsx
import { createRoot } from 'react-dom/client'
import { useEffect, useState } from 'react'

function Clock() {
  const [time, setTime] = useState(new Date())

  useEffect(() => {
    const id = setInterval(() => setTime(new Date()), 1000)
    return () => clearInterval(id)
  }, [])

  return (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {time.toLocaleTimeString()}.</h2>
    </div>
  )
}

createRoot(document.getElementById('root')!).render(<Clock />)
```

```html
<div id="root"></div>
```

The HTML consists of just a single root `<div>` tag. Content display and updates on the other hand are handled completely in JavaScript. There is no round trip to the server and rendered HTML is updated in-place. Here time could be replaced by any other real-time information like exchange rates or stock prices obtained from an API and displayed without refreshing the page or a round trip to the server.

### JavaScript bundles and Performance

As the complexity of the page increases to show images, display data from a data store and include event handling, the complexity and size of the JavaScript code required to render the page will also increase. CSR resulted in large JavaScript bundles which increased the FCP and TTI of the page.

As the size of bundle.js increases, the FCP and TTI are pushed forward. This implies that the user will see a blank screen for the entire duration between FP and FCP.

### Pros and Cons

With React most of the application logic is executed on the client and it interacts with the server through API calls to fetch or save data. Almost all of the UI is thus generated on the client. The entire web application is loaded on the first request. As the user navigates by clicking on links, no new request is generated to the server for rendering the pages. The code runs on the client to change the view/data.

CSR allows us to have a Single-Page Application that supports navigation without page refresh and provides a great user experience. As the data processed to change the view is limited, routing between pages is generally faster making the CSR application seem more responsive. CSR also allows developers to achieve a clear separation between client and server code.

> **Note (React 18+): Reevaluate Pure CSR for Initial Loads**
>
> While CSR yields a rich interactive experience after load, it has well-known drawbacks for first-page load performance and SEO. Today's best practice is to **avoid pure-CSR for content-rich or public-facing pages**. Instead, use hybrid approaches (SSR/SSG plus hydration) for the initial render. Frameworks like Next.js now default to pre-rendering pages on the server (or at build time) and then hydrating on the client.
>
> Server-rendering HTML can drastically improve FCP and make content indexable for search engines. React 18's improvements (automatic batching, Suspense, streaming) make SSR + hydration very performant. React 18 also introduced **Progressive Hydration** and **Selective Hydration** which mitigate the traditional TTI gap—React can hydrate parts of the UI as their scripts arrive or as the user interacts.
>
> **Conclusion:** Pure CSR (loading a big bundle and rendering everything on client) is generally **discouraged for large apps**. Use SSR/SSG for initial content and hydrate on the client. If you *do* use CSR (e.g., an internal dashboard where SEO doesn't matter), apply aggressive code-splitting and use React 18's `<Suspense>` with lazy-loaded components to defer loading non-critical parts of the UI.

Despite the great interactive experience that it provides, there are a few pitfalls to CSR:

1. **SEO considerations:** Most web crawlers can interpret server rendered websites in a straight-forward manner. Things get slightly complicated in the case of client-side rendering as large payloads and a waterfall of network requests (e.g for API responses) may result in meaningful content not being rendered fast enough for a crawler to index it. Crawlers may understand JavaScript but there are limitations. As such, some workarounds are required to make a client-rendered website SEO friendly.

2. **Performance**: With client-side rendering, the response time during interactions is greatly improved as there is no round trip to the server. However, for browsers to render content on client-side the first time, they have to wait for the JavaScript to load first and start processing. Thus users will experience some lag before the initial page loads. This may affect the user experience as the size of JS bundles get bigger and/or the client does not have sufficient processing power.

3. **Code Maintainability:** Some elements of code may get repeated across client and server (APIs) in different languages. In other cases, clean separation of business logic may not be possible. Examples of this could include validations and formatting logic for currency and date fields.

4. **Data Fetching**: With client-side rendering, data fetching is usually event-driven. The page could initially be loaded without any data. Data may be subsequently fetched on the occurrence of events like page-load or button-clicks using API calls. Depending on the size of data this could add to the load/interaction time of the application.

The importance of these considerations may be different across applications. Developers are often interested in finding SEO friendly solutions that can serve pages faster without compromising on the interaction time.

### Improving CSR performance

Since performance for CSR is inversely proportional to the size of the JavaScript bundle, the best thing we can do is structure our JavaScript code for optimal performance. Following is a list of pointers that could help:

- **Budgeting JavaScript**: Ensure that you have a reasonably tight JavaScript budget for your initial page loads. An initial bundle of < 100-170KB minified and gzipped is a good starting point. Code can then be loaded on-demand as features are needed.

- **Preloading**: This technique can be used to preload critical resources that would be required by the page, earlier in the page lifecycle. Critical resources may include JavaScript which can be preloaded by including the following directive in the `<head>` section of the HTML.

```html
<link rel="preload" as="script" href="critical.js" />
```

This informs the browser to start loading the `critical.js` file before the page rendering mechanism starts. The script will thus be available earlier and will not block the page rendering mechanism thereby improving the performance.

- **Lazy loading**: With lazy loading, you can identify resources that are non-critical and load these only when needed. Initial page load times can be improved using this approach as the size of resources loaded initially is reduced. For example, a chat widget component would generally not be needed immediately on page load and can be lazy loaded.

- **Code Splitting**: To avoid a large bundle of JavaScript code, you could start splitting your bundles. Code-Splitting is supported by bundlers like [Webpack](https://webpack.js.org/guides/code-splitting/) where it can be used to create multiple bundles that can be dynamically loaded at runtime. Code splitting also enables you to lazy load JavaScript resources.

- **Application shell caching with service workers:** This technique involves caching the application shell which is the minimal HTML, CSS, and JavaScript powering a user interface. Service workers can be used to cache the application shell offline. This can be useful in providing a native single-page app experience where the remaining content is loaded progressively as needed.

With these techniques, CSR can help to provide a faster Single-Page Application experience with a decent FCP and TTI.

## Source

- [patterns.dev/react/client-side-rendering](https://patterns.dev/react/client-side-rendering)
