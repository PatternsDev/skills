---
name: streaming-ssr
description: Stream server-rendered HTML to the client in chunks for faster Time to First Byte and First Contentful Paint.
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

# Streaming Server-Side Rendering

We can reduce the Time To Interactive while still server rendering our application by _streaming_ the contents of our application. Instead of generating one large HTML string containing the necessary markup for the current navigation, we can send the shell first and stream slower parts later. The moment the client receives the first chunks of HTML, it can start parsing and painting the page.

Modern React streaming uses `renderToPipeableStream()` on Node runtimes or `renderToReadableStream()` on Web Stream runtimes, then hydrates the response with `hydrateRoot()` on the client.

## When to Use

- Use this when you want to improve TTFB and FCP by sending HTML incrementally as it's generated
- This is helpful for large pages where waiting for the full HTML would delay the initial paint

## Instructions

- Use `renderToPipeableStream` (React 18+) instead of the deprecated `renderToNodeStream`
- Combine streaming with `Suspense` boundaries to stream partial content while slow parts load
- Use the `onShellReady` callback to begin streaming once the critical shell is ready
- Handle streaming errors with the `onError` callback

## Details

The initial HTML gets sent to the response object alongside the chunks of data from the App component:

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Cat Facts</title>
    <link rel="stylesheet" href="/style.css" />
    <script type="module" defer src="/build/client.js"></script>
  </head>
  <body>
    <h1>Stream Rendered Cat Facts!</h1>
    <div id="approot"></div>
  </body>
</html>
```

Modern React streaming on Node uses `renderToPipeableStream`:

```js
import { renderToPipeableStream } from "react-dom/server";

app.use("*", (request, response) => {
  let didError = false;

  const { pipe } = renderToPipeableStream(<App />, {
    bootstrapScripts: ["/build/client.js"],
    onShellReady() {
      response.statusCode = didError ? 500 : 200;
      response.setHeader("Content-Type", "text/html");
      pipe(response);
    },
    onError(error) {
      didError = true;
      console.error(error);
    },
  });
});
```

If we were to server render the `App` component using `renderToString`, we would have to wait until the entire tree had rendered before sending the response. With streaming, the server can flush the shell early and continue sending slower content as it becomes ready.

### Concepts

Like progressive hydration, streaming is another rendering mechanism that can be used to improve SSR performance. As the name suggests, streaming implies chunks of HTML are streamed from the node server to the client as they are generated. As the client starts receiving "bytes" of HTML earlier even for large pages, the TTFB is reduced and relatively constant. All major browsers start parsing and rendering streamed content or the partial response earlier. As the rendering is progressive, it results in a fast FP and FCP.

Streaming responds well to network backpressure. If the network is clogged and not able to transfer any more bytes, the renderer gets a signal and stops streaming till the network is cleared up. Thus, the server uses less memory and is more responsive to I/O conditions. This enables your Node.js server to render multiple requests at the same time and prevents heavier requests from blocking lighter requests for a long time. As a result, the site stays responsive even in challenging conditions.

### React for Streaming

React 18 introduced the modern streaming APIs:

1. **`renderToPipeableStream(element, options)`** for Node.js HTTP responses.
2. **`renderToReadableStream(element, options)`** for Web Streams runtimes such as edge environments.

These APIs support Suspense boundaries, `onShellReady`, `onAllReady`, and progressive hydration through `hydrateRoot()` on the client.

The stream output can emit bytes as soon as the shell is ready. The response progressively sends chunks of data to the client while slower chunks continue rendering on the server.

### Streaming SSR - Pros and Cons

Streaming aims to improve the speed of SSR with React and provides the following benefits:

1. **Performance Improvement:** As the first byte reaches the client soon after rendering starts on the server, the TTFB is better than that for SSR. It is also more consistent irrespective of the page size. Since the client can start parsing HTML as soon as it receives it, the FP and FCP are also lower.

2. **Handling of Backpressure**: Streaming responds well to network backpressure or congestion and can result in responsive websites even under challenging conditions.

3. **Supports SEO**: The streamed response can be read by search engine crawlers, thus allowing for SEO on the website.

It is important to note that streaming implementation is not a simple find-replace from `renderToString` to `renderToPipeableStream()`. There are cases where the code that works with SSR may not work as-is with streaming:

1. Frameworks that use the server-render-pass to generate markup that needs to be added to the document before the SSR-ed chunk. Examples are frameworks that dynamically determine which CSS to add to the page in a preceding `<style>` tag.

2. Code, where `renderToStaticMarkup` is used to generate the page template and `renderToString` calls are embedded to generate dynamic content. Since the string corresponding to the component is expected in these cases, it cannot be replaced by a stream. For example:

```js
res.write("<!DOCTYPE html>");

res.write(renderToStaticMarkup(
 <html>
   <head>
     <title>My Page</title>
   </head>
   <body>
     <div id="content">
       { renderToString(<MyPage/>) }
     </div>
   </body>
 </html>);
```

Both Streaming and Progressive Hydration can help to bridge the gap between a pure SSR and a CSR experience.

## Source

- [patterns.dev/react/streaming-ssr](https://patterns.dev/react/streaming-ssr)
