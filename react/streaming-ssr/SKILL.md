---
name: streaming-ssr
description: Stream server-rendered HTML to the client in chunks for faster Time to First Byte and First Contentful Paint.
metadata:
  author: patterns.dev
  version: "1.0"
---

# Streaming Server-Side Rendering

We can reduce the Time To Interactive while still server rendering our application by _streaming server rendering_ the contents of our application. Instead of generating one large HTML file containing the necessary markup for the current navigation, we can split it up into smaller chunks! Node streams allow us to stream data into the response object, which means that we can continuously send data down to the client. The moment the client receives the chunks of data, it can start rendering the contents.

React's built-in `renderToNodeStream` makes it possible for us to send our application in smaller chunks. As the client can start painting the UI when it's still receiving data, we can create a very performant first-load experience. Calling the `hydrate` method on the received DOM nodes will attach the corresponding event handlers, which makes the UI interactive!

## When to Use

- Use this when you want to improve TTFB and FCP by sending HTML incrementally as it's generated
- This is helpful for large pages where waiting for the full HTML would delay the initial paint

## Instructions

- Use `renderToPipeableStream` (React 18+) instead of the deprecated `renderToNodeStream`
- Combine streaming with `Suspense` boundaries to stream partial content while slow parts load
- Use the `onShellReady` callback to begin streaming once the critical shell is ready
- Handle streaming errors with the `onError` callback
- Use the ask questions tool if you need to clarify requirements with the user

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

The `renderToNodeStream` method streams data to the client:

```js
import { renderToNodeStream } from 'react-dom/server';
import Frontend from '../client';

app.use('*', (request, response) => {
  // Send the start of your HTML to the browser
  response.write('<html><head><title>Page</title></head><body><div id="root">');

  // Render your frontend to a stream and pipe it to the response
  const stream = renderToNodeStream(<Frontend />);
  stream.pipe(response, { end: 'false' });

  // When React finishes rendering send the rest of your HTML to the browser
  stream.on('end', () => {
    response.end('</div></body></html>');
  });
});
```

If we were to server render the `App` component using the `renderToString` method, we would have had to wait until the application has received all data before it can start loading and processing this metadata. To speed this up, `renderToNodeStream` makes it possible for the app to start loading and processing this information as it's still receiving the chunks of data from the App component!

### Concepts

Like progressive hydration, streaming is another rendering mechanism that can be used to improve SSR performance. As the name suggests, streaming implies chunks of HTML are streamed from the node server to the client as they are generated. As the client starts receiving "bytes" of HTML earlier even for large pages, the TTFB is reduced and relatively constant. All major browsers start parsing and rendering streamed content or the partial response earlier. As the rendering is progressive, it results in a fast FP and FCP.

Streaming responds well to network backpressure. If the network is clogged and not able to transfer any more bytes, the renderer gets a signal and stops streaming till the network is cleared up. Thus, the server uses less memory and is more responsive to I/O conditions. This enables your Node.js server to render multiple requests at the same time and prevents heavier requests from blocking lighter requests for a long time. As a result, the site stays responsive even in challenging conditions.

### React for Streaming

React introduced support for streaming in React 16. The following APIs were included in the ReactDOMServer to support streaming.

1. **`ReactDOMServer.renderToNodeStream(element)`**: The output HTML from this function is the same as `ReactDOMServer.renderToString(element)` but is in a Node.js readablestream format instead of a string. The function will only work on the server to render HTML as a stream. The client receiving this stream can subsequently call `ReactDOM.hydrate()` to hydrate the page and make it interactive.

2. **`ReactDOMServer.renderToStaticNodeStream(element)`**: This corresponds to `ReactDOMServer.renderToStaticMarkup(element)`. The HTML output is the same but in a stream format. It can be used for rendering static, non-interactive pages on the server and then streaming them to the client.

> **Note (React 18+): Use `renderToPipeableStream` Instead of `renderToNodeStream`**
>
> In React 18, `renderToNodeStream` has been replaced by **`renderToPipeableStream`** (for Node HTTP streams) and **`renderToReadableStream`** (for Web Streams API in edge runtimes). The new APIs support Suspense boundaries, allowing you to send partial content and even display a `<Suspense fallback>` for slow parts.
>
> The new API supports `onShellReady` and `onAllReady` callbacks. Use `onShellReady` (which fires when the shell is ready to stream) to flush any critical scripts and begin streaming to the client. React will handle flushing Suspense boundaries automatically.
>
> **Error Handling:** Ensure you handle streaming errors using the `onError` callback of `renderToPipeableStream`.
>
> Using the modern API ensures compatibility with React's concurrency features—the older `renderToNodeStream` does not support Suspense properly.

The readable stream output by both functions can emit bytes once you start reading from it. This can be achieved by piping the readable stream to a writable stream such as the response object. The response object progressively sends chunks of data to the client while waiting for new chunks to be rendered.

### Streaming SSR - Pros and Cons

Streaming aims to improve the speed of SSR with React and provides the following benefits:

1. **Performance Improvement:** As the first byte reaches the client soon after rendering starts on the server, the TTFB is better than that for SSR. It is also more consistent irrespective of the page size. Since the client can start parsing HTML as soon as it receives it, the FP and FCP are also lower.

2. **Handling of Backpressure**: Streaming responds well to network backpressure or congestion and can result in responsive websites even under challenging conditions.

3. **Supports SEO**: The streamed response can be read by search engine crawlers, thus allowing for SEO on the website.

It is important to note that streaming implementation is not a simple find-replace from `renderToString` to `renderToNodeStream()`. There are cases where the code that works with SSR may not work as-is with streaming:

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
