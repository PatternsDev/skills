---
name: incremental-static-rendering
description: Update static content after you have built your site using Incremental Static Regeneration (ISR).
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

# Incremental Static Generation

Static Generation (SSG) addresses most of the concerns of SSR and CSR but is suitable for rendering mostly static content. It poses limitations when the content to be rendered is dynamic or changing frequently.

Think of a growing blog with multiple posts. You wouldn't possibly want to rebuild and redeploy the site just because you want to correct a typo in one of the posts. Similarly, one new blog post should also not require a rebuild for all the existing pages. Thus, SSG on its own is not enough for rendering large websites or applications.

## When to Use

- Use this when you have mostly static pages that need periodic data updates without full rebuilds
- This is helpful for large sites (blogs, e-commerce) where rebuilding every page on each change is impractical

## Instructions

- Use `revalidate` in `getStaticProps` to set a time interval for background page regeneration
- Use `fallback: true` in `getStaticPaths` to lazily generate pages on first request
- Consider on-demand revalidation (`revalidatePath`, `revalidateTag`) for immediate updates after content changes
- In Next.js 13+ App Router, use `generateStaticParams` and async components for ISR

## Details

The Incremental Static Generation (iSSG) pattern was introduced as an upgrade to SSG, to help solve the dynamic data problem and help static sites scale for large amounts of frequently changing data. iSSG allows you to update existing pages and add new ones by pre-rendering a subset of pages in the background even while fresh requests for pages are coming in.

### Sample Code

iSSG works on two fronts to incrementally introduce updates to an existing static site after it has been built.

1. Allows addition of new pages
2. Allows updates to existing pages also known as Incremental Static "Re"generation

#### Adding New pages

The lazy loading concept is used to include new pages on the website after the build. This means that the new page is generated immediately on the first request. While the generation takes place, a fallback page or a loading indicator can be shown to the user on the front-end.

```js
export async function getStaticPaths() {
  const products = await getProductsFromDatabase();

  const paths = products.map((product) => ({
     params: { id: product.id }
  }));

  // fallback: true means that the missing pages
  // will not 404, and instead can render a fallback.
  return { paths, fallback: true };
}

// params will contain the id for each generated page.
export async function getStaticProps({ params }) {
  return {
    props: {
      product: await getProductFromDatabase(params.id)
    }
  }
}

export default function Product({ product }) {
  const router = useRouter();

  if (router.isFallback) {
    return <div>Loading...</div>;
  }

  // Render product
}
```

Here, we have used `fallback: true`. Now if the page corresponding to a specific product is unavailable, we show a fallback version of the page, eg., a loading indicator as shown in the Product function above. Meanwhile, Next.js will generate the page in the background. Once it is generated, it will be cached and shown instead of the fallback page. The cached version of the page will now be shown to any subsequent visitors immediately upon request.

#### Update Existing pages

To re-render an existing page, a suitable timeout is defined for the page. This will ensure that the page is revalidated whenever the defined timeout period has elapsed. The user will continue to see the previous version of the page, till the page has finished revalidation. Thus, iSSG uses the [stale-while-revalidate](https://web.dev/stale-while-revalidate/) strategy where the user receives the cached or stale version while the revalidation takes place. The revalidation takes place completely in the background without the need for a full rebuild.

```js
// This function runs at build time on the build server
export async function getStaticProps() {
  return {
    props: {
      products: await getProductsFromDatabase(),
      revalidate: 60, // This will force the page to revalidate after 60 seconds
    }
  }
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
  )
}
```

The code to revalidate the page after 60 seconds is included in the `getStaticProps()` function. When a request comes in the available static page is served first. Every one minute the static page gets refreshed in the background with new data. Once generated, the new version of the static file becomes available and will be served for any new requests in the subsequent minute. This feature is available in Next.js 9.5 and above.

### Advantages

iSSG provides all the advantages of SSG and then some more:

1. **Dynamic data**: Its ability to support dynamic data without a need to rebuild the site.
2. **Speed**: iSSG is at least as fast as SSG because data retrieval and rendering still takes place in the background. There is little processing required on the client or the server.
3. **Availability**: A fairly recent version of any page will always be available online for users to access. Even if the regeneration fails in the background, the old version remains unaltered.
4. **Consistent**: As the regeneration takes place on the server one page at a time, the load on the database and the backend is low and performance is consistent. As a result, there are no spikes in latency.
5. **Ease of Distribution**: Just like SSG sites, iSSG sites can also be distributed through a network of CDN's used to serve pre-rendered web pages.

> **Note (React 18+ / Next.js 13+): On-Demand Revalidation**
>
> A new best practice is to use **On-Demand Revalidation** when possible. Next.js provides APIs (e.g., `res.revalidate` in API routes, or `revalidatePath` and `revalidateTag` in App Router) to trigger revalidation of specific pages immediately after content changes, rather than waiting for the next timed interval.
>
> For **fallback pages**, the pattern of using `fallback: true` in `getStaticPaths` remains valid. In Next.js 13 App Router, fallback behavior is handled by the `loading.js` conventions automatically.
>
> **Edge caching and ISR:** Platforms like Vercel and Cloudflare let you run ISR on the edge. ISR is now a **standard practice** for large sites—refine it with targeted revalidation and good loading states.

## Source

- [patterns.dev/react/incremental-static-rendering](https://patterns.dev/react/incremental-static-rendering)
