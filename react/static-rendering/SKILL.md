---
name: static-rendering
description: Deliver pre-rendered HTML content that was generated at build time for fast, cacheable pages.
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

# Static Rendering

Based on our discussion on SSR, we know that a high request processing time on the server negatively affects the TTFB. Similarly, with CSR, a large JavaScript bundle can be detrimental to the FCP, LCP and TTI of the application due to the time taken to download and process the script.

Static rendering or static generation (SSG) attempts to resolve these issues by delivering pre-rendered HTML content to the client that was generated when the site was built.

## When to Use

- Use this for static content like About pages, blog posts, and product listings that don't change per-request
- This is helpful when you want the fastest possible TTFB via CDN-served static HTML

## Instructions

- Use `getStaticProps` (Pages Router) or async components (App Router) to fetch data at build time
- Use `getStaticPaths` / `generateStaticParams` to pre-render dynamic routes
- Consider Incremental Static Regeneration (ISR) for pages that need periodic updates
- Deploy to a CDN for edge-cached performance

## Details

A static HTML file is generated ahead of time corresponding to each route that the user can access. These static HTML files may be available on a server or a CDN and fetched as and when requested by the client.

Static files may also be cached thereby providing greater resiliency. Since the HTML response is generated in advance, the processing time on the server is negligible thereby resulting in a faster TTFB and better performance. In an ideal scenario, client-side JS should be minimal and static pages should become interactive soon after the response is received by the client. As a result, SSG helps to achieve a faster FCP/TTI.

### Basic Structure

As the name suggests, static rendering is ideal for static content, where the page need not be customized based on the logged-in user (e.g personalized recommendations). Thus static pages like the '_About us_', '_Contact us_', _Blog_ pages for websites or product pages for e-commerce apps, are ideal candidates for static rendering. Frameworks like Next.js, Gatsby, and VuePress support static generation.

**Next.js:**

```js
// pages/about.js

export default function About() {
  return (
    <div>
      <h1>About Us</h1>
      {/* ... */}
    </div>
  );
}
```

When the site is built (using `next build`), this page will be pre-rendered into an HTML file `about.html` accessible at the route `/about`.


### SSG with Data

Static content like that in 'About us' or 'Contact us' pages may be rendered as-is without getting data from a data-store. However, for content like individual blog pages or product pages, the data from a data-store has to be merged with a specific template and then rendered to HTML at build time.

#### Listing Page - All Items

In Next.js this can be achieved by exporting the function `getStaticProps()` in the page component. The function is called at build time on the build server to fetch the data.

```js
// This function runs at build time on the build server
export async function getStaticProps() {
  return {
    props: {
      products: await getProductsFromDatabase(),
    },
  };
}

// The page component receives products prop from getStaticProps at build time
export default function Products({ products }) {
  return (
    <>
      <h1>Products</h1>
      <ul>
        {products.map((product) => (
          <li key={product.id}>{product.name}</li>
        ))}
      </ul>
    </>
  );
}
```

The function will not be included in the client-side JS bundle and hence can even be used to fetch the data directly from a database.

#### Individual Details Page - Per Item

For individual details pages, we can use the function `getStaticPaths()` in combination with dynamic routes.

```js
// pages/products/[id].js

export async function getStaticPaths() {
  const products = await getProductsFromDatabase();

  const paths = products.map((product) => ({
    params: { id: product.id },
  }));

  // fallback: false means pages that don't have the correct id will 404.
  return { paths, fallback: false };
}

// params will contain the id for each generated page.
export async function getStaticProps({ params }) {
  return {
    props: {
      product: await getProductFromDatabase(params.id),
    },
  };
}

export default function Product({ product }) {
  // Render product
}
```

### SSG - Key Considerations

> **Note (React 18+ / Next.js 13+): Modern Static Generation**
>
> In Next.js 13's App Router, `getStaticProps` and `getStaticPaths` have their equivalents in **`generateStaticParams`** (for dynamic routes) and the ability to fetch data in an async component for static generation.
>
> **Use Incremental Static Regeneration (ISR) for Freshness:** If you have pages that are mostly static but need periodic updates, set a `revalidate` interval or use on-demand revalidation.
>
> **Partial Prerendering (PPR):** A recent development in Next.js 14+ is Partial Prerendering, which allows a page to be *partially* statically rendered at build time and partially filled in at request time.

1. **A large number of HTML files:** Individual HTML files need to be generated for every possible route that the user may access. Maintaining a large number of HTML files can be challenging.

2. **Hosting Dependency:** For an SSG site to be super-fast and respond quickly, the hosting platform used to store and serve the HTML files should also be good. Superlative performance is possible if a well-tuned SSG website is hosted right on multiple CDNs to take advantage of edge-caching.

3. **Dynamic Content:** An SSG site needs to be built and re-deployed every time the content changes. The content displayed may be stale if the site has not been built + deployed after any content change. This makes SSG unsuitable for highly dynamic content.

## Source

- [patterns.dev/react/static-rendering](https://patterns.dev/react/static-rendering)
