---
name: server-side-rendering
description: Generate HTML on the server in response to user requests for faster initial page loads and SEO.
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

# Server-side Rendering

Server-side rendering (SSR) is one of the oldest methods of rendering web content. SSR generates the full HTML for the page content to be rendered in response to a user request. The content may include data from a datastore or external API.

The connect and fetch operations are handled on the server. HTML required to format the content is also generated on the server. Thus, with SSR we can avoid making additional round trips for data fetching and templating. As such, rendering code is not required on the client and the JavaScript corresponding to this need not be sent to the client.

## When to Use

- Use this when SEO and fast First Contentful Paint are important for your application
- This is helpful for content-heavy pages that need to be quickly visible to users and search engines

## Instructions

- Use frameworks like Next.js that provide built-in SSR support
- Consider upgrading to `renderToPipeableStream` (React 18+) for streaming SSR with Suspense support
- Combine SSR with client-side hydration for interactive pages
- Be aware of TTFB implications — optimize server response times and consider caching
- Explore React Server Components as a complement to SSR for reducing client-side JavaScript

## Details

With SSR every request is treated independently and will be processed as a new request by the server. Even if the output of two consecutive requests is not very different, the server will process and generate it from scratch. Since the server is common to multiple users, the processing capability is shared by all active users at a given time.

### Classic SSR Implementation

Consider a simple example showing and updating the current time on a page using classic SSR and JavaScript.

```html
<!DOCTYPE html>
<html>
   <head>
       <title>Time</title>
   </head>
   <body>
       <div>
       <h1>Hello, world!</h1>
       <b>It is <div id=currentTime></div></b>
       </div>
   </body>
</html>
```

```js
function tick() {
    var d = new Date();
    var n = d.toLocaleTimeString();
    document.getElementById("currentTime").innerHTML = n;
}
setInterval(tick, 1000);
```

Note how this is different from the CSR code that provides the same output. Also note that, while the HTML is rendered by the server, the time displayed here is the local time on the client as populated by the JavaScript function `tick()`. If you want to display any other data that is server specific, e.g., server time, you will need to embed it in the HTML before it is rendered. This means it will not get refreshed automatically without a round trip to the server.

### Pros and Cons

Executing the rendering code on the server and reducing JavaScript offers the following advantages.

#### Lesser JavaScript leads to quicker FCP and TTI

In cases where there are multiple UI elements and application logic on the page, SSR has considerably less JavaScript when compared to CSR. The time required to load and process the script is thus lesser. FP, FCP and TTI are shorter and FCP = TTI. With SSR, users will not be left waiting for all the screen elements to appear and for it to become interactive.

#### Provides additional budget for client-side JavaScript

Development teams are required to work with a JS budget that limits the amount of JS on the page to achieve the desired performance. With SSR, since you are directly eliminating the JS required to render the page, it creates additional space for any third party JS that may be required by the application.

#### SEO enabled

Search engine crawlers are easily able to crawl the content of an SSR application thus ensuring higher search engine optimization on the page.

> **Note (React 18+): Adopt Streaming SSR with Suspense**
>
> Traditional SSR renders the full HTML on the server for each request. React 18 introduced **Streaming Server-Side Rendering**, which sends HTML to the client in chunks as it's generated, rather than waiting for the whole render to finish. This significantly improves TTFB and LCP—users see the page shell almost immediately.
>
> If your SSR implementation uses older APIs like `ReactDOMServer.renderToString` or `renderToNodeStream`, consider upgrading to `renderToPipeableStream` (Node) or `renderToReadableStream` (for edge environments) introduced in React 18. These APIs support Suspense boundaries, allowing you to send partial content and display a `<Suspense fallback>` for slow parts.
>
> **React Server Components:** Perhaps the biggest change is the rise of **React Server Components (RSC)**. RSCs allow you to render part of your UI on the server *ahead of time* without sending the associated JS to the client, dramatically shrinking client JS bundles.

SSR works great for static content due to the above advantages. However, it does have a few disadvantages because of which it is not perfect for all scenarios.

#### Slow TTFB

Since all processing takes place on the server, the response from the server may be delayed in case of one or more of the following scenarios:

- Multiple simultaneous users causing excess load on the server.
- Slow network
- Server code not optimized.

#### Full page reloads required for some interactions

Since all code is not available on the client, frequent round trips to the server are required for all key operations causing full page reloads. This could increase the time between interactions as users are required to wait longer between operations. A single-page application is thus not possible with SSR.

### SSR with Next.js

The Next.js framework also supports SSR. This pre-renders a page on the server on every request. It can be accomplished by exporting an async function called [`getServerSideProps()`](https://nextjs.org/docs/basic-features/data-fetching#getserversideprops-server-side-rendering) from a page as follows.

```js
export async function getServerSideProps(context) {
  return {
    props: {}, // will be passed to the page component as props
  };
}
```

The context object contains keys for HTTP request and response objects, routing parameters, querystring, locale, etc.

The following implementation shows the use of `getServerSideProps()` for rendering data on a page formatted using React:

```js
// data fetched from an external data source using `getServerSideProps`

const Users = ({ users, error }) => {
 return (
   <section>
     <header>
       <h1>List of users</h1>
     </header>
     {error && <div>There was an error.</div>}
     {!error && users && (
       <table>
         <thead>
           <tr>
             <th>Username</th>
             <th>Email</th>
             <th>Name</th>
           </tr>
         </thead>
         <tbody>
           {users.map((user, key) => (
             <tr key={key}>
               <td>{user.username}</td>
               <td>{user.email}</td>
               <td>{user.name}</td>
             </tr>
           ))}
         </tbody>
       </table>
     )}
   </section>
 );
};

export async function getServerSideProps() {
 // Fetch data from external API
 const res = await fetch("https://jsonplaceholder.typicode.com/users")
 const data = await res.json();

 // Pass data to the page via props
 return { props: { data } }
}

export default Users;
```

### React for the Server

React can be rendered isomorphically, which means that it can function both on the browser as well as other platforms like the server. Thus, UI elements may be rendered on the server using React.

React can also be used with universal code which will allow the same code to run in multiple environments. This is made possible by using Node.js on the server.

```js
ReactDOMServer.renderToString(element);
```

This function returns an HTML string corresponding to the React element. The HTML can then be rendered to the client for a faster page load.

The `renderToString()` function may be used with `hydrateRoot()`. This preserves the HTML rendered on the server and attaches event handlers on the client.

To implement this, we use a `.js` file on both client and server corresponding to every page. The `.js` file on the server will render the HTML content, and the `.js` file on the client will hydrate it.

The server code:

```js
app.get("/", (req, res) => {
  const app = ReactDOMServer.renderToString(<App />);
});
```

The client-side code to ensure the element `App` is hydrated:

```js
import { hydrateRoot } from "react-dom/client";

hydrateRoot(document.getElementById("root"), <App />);
```

A complete example of SSR with React can be found [here](https://www.digitalocean.com/community/tutorials/react-server-side-rendering).

## Source

- [patterns.dev/react/server-side-rendering](https://patterns.dev/react/server-side-rendering)
