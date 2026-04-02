---
name: loading-sequence
description: Optimize the loading sequence of critical resources to improve Core Web Vitals like FCP, LCP, and FID.
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

# Optimize your loading sequence

In every successful web page load, some critical components and resources become available at just the right time to give you a smooth loading experience. This ensures users perceive the performance of the application to be excellent. This excellent user experience should generally also translate to passing the [Core Web Vitals](https://web.dev/vitals/).

## When to Use

- Use this when optimizing page load performance for Core Web Vitals
- This is helpful when you need to coordinate the loading of 1P JS, 3P JS, CSS, fonts, and images
- Use this when third-party scripts are impacting your loading performance

## Instructions

- Inline critical CSS and font CSS; use preconnect for external fonts
- Sequence resources so FCP fires before LCP, and LCP fires before FID
- Start fetching first-party JS before ATF images on the network
- Use `async` or `defer` attributes for non-critical scripts
- Lazy-load below-the-fold images and non-essential third-party resources

## Details

Key [metrics](https://web.dev/metrics/) such as [First Content Paint](https://web.dev/fcp/), [Largest Contentful Paint](https://web.dev/lcp/), [First Input Delay](https://web.dev/fid/), etc used to measure performance are directly dependent on the loading sequence of critical resources. For example, the page cannot have its LCP if a critical resource like the [hero image](https://en.wikipedia.org/wiki/Hero_image) is not loaded. This article talks about the relationship between the loading sequence of resources and web vitals. Our objective is to provide clear guidance on how to optimize the loading sequence for better web vitals.

Before we establish an ideal loading sequence, let us first try to understand why it is so difficult to get the loading sequence right.

### Why is optimal loading difficult to achieve?

We have had the unique opportunity to work on performance analysis for many of our partner's websites. We identified multiple similar issues that plagued the efficient loading of pages across different partner sites.

There is often a critical gap between developers' expectations and how the browser prioritizes resources on the page. This often results in sub-optimal performance scores. We analyzed further to discover what caused this gap and the following points summarise the essence of our analysis.

**Sub-optimal sequencing**

[Web Vitals](https://web.dev/vitals) optimization requires not only a good understanding of what each metrics stands for but also the order in which they occur and how they relate to different critical resources. FCP occurs before LCP which occurs before FID. As such, resources required for achieving FCP should be prioritized over those required by LCP followed by those required by FID.

Resources are often not sequenced and pipelined in the correct order. This may be because developers are not aware of the dependency of metrics on resource loads. As a result, relevant resources are sometimes not available at the right time for the corresponding metric to trigger.

**Examples**:

a) By the time FCP fires, the hero image should be available for firing LCP.
b) By the time LCP fires, the JavaScript (JS) should be downloaded, parsed and ready (or already executing) to unblock interaction (FID).

**Network/CPU Utilization**

Resources are also not pipelined appropriately to ensure full CPU and Network utilization. This results in "Dead Time" on the CPU when the process is network bound and vice versa.

A great example of this is scripts that may be downloaded concurrently or sequentially. As the bandwidth gets divided during concurrent download, the total time for downloading all scripts is the same for both sequential and concurrent downloads. If you download scripts concurrently, the CPU is underutilized during the download. However, if you download the scripts sequentially, the CPU can start processing the first one as soon as it is downloaded. This results in better CPU and Network utilization.

**Third-Party (3P) Products**

3P libraries are often required to add common features and functionality to the website. Third parties include ads, analytics, social widgets, live chat, and other embeds that power a website. A third party library comes with its own JavaScript, images, fonts etc.

3P products don't usually have an incentive to optimize for and support the consumer site's loading performance. They could have a heavy JavaScript execution cost that delays interactivity, or gets in the way of other critical resources being downloaded.

Developers who include 3P products may tend to focus more on the value they add in terms of features rather than performance implications. As a result, 3P resources are sometimes added haphazardly, without full consideration in terms of how it fits into the overall loading sequence. This makes them hard to control and schedule.

**Platform Quirks**

Browsers may differ in how they prioritize requests and implement hints. Optimization is easier if you have a deep knowledge of the platform and its quirks. Behavior particular to a specific browser makes it difficult to achieve the desired loading sequence consistently.

An example of this is the [preload bug](https://bugs.chromium.org/p/chromium/issues/detail?id=788757) on the chromium platform. The [Preload](https://medium.com/reloading/preload-prefetch-and-priorities-in-chrome-776165961bbf) (`<link rel=preload>`) instruction can be used to tell the browser to download key resources as soon as possible. It should only be used when you are sure that the resource will be used on the current page. The bug in Chromium causes it to behave such that requests issued via `<link rel=preload>` always start before other requests seen by the preload scanner even if those have higher priority. Issues such as these put a wrench in optimization plans.

**HTTP2 Prioritization**

The protocol itself does not provide many options or knobs for adjusting the order and priority of resources. Even if better prioritization primitives were to be made available, there are underlying [problems with HTTP2 prioritization](https://github.com/andydavies/http2-prioritization-issues) that make optimal sequencing difficult. Mainly, we cannot predict in what order servers or CDN's will prioritize requests for individual resources. Some CDN's re-prioritize requests while others implement partial, flawed, or no prioritization.

**Resource level optimization**

Effective sequencing needs that the resources that are being sequenced to be served optimally so that they will load quickly. Critical CSS should be inlined, Images should be sized correctly and JS should be code-split and delivered incrementally.

The framework itself is lacking constructs that allow code-splitting and serve JS and data incrementally. Users must rely on one of the following to split large chunks of 1P JS:

1. Modern React (Suspense / Concurrent mode / Data Fetching) - This is still available for experimentation only
2. Lazy loading using dynamic imports - This is not intuitive and developers need to manually identify the boundaries along which to split the code.

When code-splitting, developers need to achieve just the right granularity of chunks because of a granularity vs performance trade-off.

Higher granularity is desirable because it:

1. Minimizes JS needed for individual route and on subsequent user interactions
2. Allows for caching of common dependencies. This ensures that a change in the library doesn't require re-fetching of the entire bundle.

At the same time too much granularity when code-splitting can be bad because too many small chunks lower compression rates for individual chunks and affect browser performance.

Resource optimization also requires the elimination of dead or unused code. Unnecessary or obsolete JS may be often shipped to modern browsers which negatively affects performance. JS transpiled to ES5 and bundled with polyfills is unnecessary for modern browsers. Libraries and npm packages are often not published in ES module format. This makes it hard for bundlers to tree shake and optimize.

As you might have noticed, these issues are not limited to a particular set of resources or platforms. To work around these problems, one requires an understanding of the entire tech stack and how different resources can be coalesced to achieve optimal metrics.

### More on Resources - Relations, Constraints, and Priorities

In the previous section, we gave a few examples of how certain resources are required for a specific event like FCP or LCP to fire. Let us try to understand all such dependencies first before we discuss a way to work with them. Following is a resource-wise list of recommendations, constraints, and gotchas that need to be considered before we define an ideal sequence.

**Critical CSS**

[Critical CSS](https://web.dev/extract-critical-css/) refers to the minimum CSS required for FCP. It is better to inline such CSS within HTML rather than import it from another CSS file. Only the CSS required for the route should be downloaded at any given time and all critical CSS should be split accordingly.

If inlining is not possible, critical CSS should be preloaded and served from the same origin as the document. Avoid serving critical CSS from multiple domains or direct use of 3rd party critical CSS like Google Fonts. Your own server could serve as a proxy for 3rd party critical CSS instead.

Delay in fetching CSS or incorrect order of fetching CSS could impact FCP and LCP. To avoid this, non-inlined CSS should be prioritized and ordered above 1P JS and ATF images on the network.

Too much inlined CSS can cause HTML bloating and long style parsing times on the main thread. This can hurt the FCP. As such identifying what is critical and code-splitting are essential.

Inlined CSS cannot be cached. One workaround for this is to have a duplicate request for the CSS that can be cached. Note however, that this can result in multiple full-page layouts which could impact FID.

**Fonts**

Like critical CSS, the CSS for [critical fonts](https://web.dev/font-best-practices/) should also be inlined. If inlining is not possible the script should be loaded with a [preconnect](https://web.dev/uses-rel-preconnect/) specified. Delay in fetching fonts, e.g., google fonts or fonts from a different domain can affect FCP. Preconnect tells the browser to set up connections to these resources earlier.

Inlining fonts can bloat the HTML significantly and delay initiating other critical resource fetches. [Font fallback](https://css-tricks.com/css-basics-fallback-font-stacks-robust-web-typography/) may be used to unblock FCP and make the text available. However, using font fallback can affect [CLS](https://web.dev/cls/) due to jumping fonts. It can also affect FID due to a potentially large style and layout task on the main thread when the real font arrives.

**Above the Fold (ATF) Images**

This refers to images that are initially visible to the user on page load because they are within the viewport. A special case for ATF images is the hero image for the page. All ATF images should be sized. Unsized images hurt the CLS metric because of the layout shift that occurs when they are fully rendered. Placeholders for ATF images should be rendered by the server.

Delayed hero image or blank placeholders would result in a late LCP. Moreover, LCP will re-trigger, if the placeholder size does not match with the intrinsic size of the actual hero image and the image is not overlaid on replacement. Ideally, there should be no impact on FCP due to ATF images but in practice, an image can fire FCP.

**Below the Fold (BTF) Images**

These are images that are not immediately visible to the user on page load. As such they are ideal candidates for lazy loading. This ensures that they do not contend with 1P JS or important 3P needed on the page. If BTF images were to be loaded before 1P JS or important 3P resources, FID would get delayed.

**1P JavaScript**

1P JS impacts the interaction readiness of the application. It can get delayed on the network behind images & 3P JS and on the main thread behind 3P JS. As such it should start loading before ATF images on the network and execute before 3P JS on the main thread. 1P JS does not block FCP and LCP in pages that are rendered on the server-side.

**3P JavaScript**

3P sync script in HTML head could block CSS & font parsing and therefore FCP. Sync script in the head also blocks HTML body parsing. 3P script execution on the main thread can delay 1P script execution and push out hydration and FID. As such, better control is required for loading 3P scripts.

These recommendations and constraints would generally apply irrespective of the tech stack and browser. Note, how something that is a recommendation can also become a constraint. For example, inlining fonts and CSS is great, but too much of it can cause bloating. The trick is to find a balance between 'Too little Too late' and 'Too much Too soon'.

Following are the key takeaways from Chrome's resource priority table:

- CSS and Fonts are loaded with the highest priority. This should help us prioritize critical CSS and fonts.
- Scripts get different priorities based on where they are in the document and whether they are async, defer, or blocking. Blocking scripts requested before the first image (or an image early in the document) are given higher priority over blocking scripts requested after the first image is fetched. Async/defer/injected scripts, regardless of where they are in the document, have the lowest priority. Thus we can prioritize different scripts by using the appropriate attributes for [async and defer](https://javascript.info/script-async-defer).
- Images that are visible and in the viewport have a higher priority (Net: Medium) than those that are not in the viewport (Net: Lowest). This helps us prioritize ATF images over BTF images.

Let us now see how all of the above details can be put together to define an optimal loading sequence.

### What is the Ideal Loading Sequence

With that background, we can now propose a loading sequence that should optimize the loading of both 1P and 3P resources. The proposed sequence uses [Next.js](https://nextjs.org/) Server Side Rendering (SSR) as a reference for optimization.

#### Current State

Based on our experience, the following is the typical loading sequence we have observed for a Next.js SSR application before optimization.

| Resource | State |
|----------|-------|
| CSS | CSS is preloaded before JS but is not inlined |
| JavaScript | 1P JS is preloaded. 3P JS is not managed and can still be render-blocking anywhere in the document. |
| Fonts | Fonts are neither inlined nor do they use preconnect. Fonts are loaded via external stylesheets which delays the loading. Fonts may or may not be display blocking. |
| Images | Hero images are not prioritized. Both ATF and BTF images are not optimized. |

#### Proposed Sequence without 3P

Following is a loading sequence that takes into account all of the constraints discussed previously. Let us first tackle a sequence without 3P. We will then see how 3P resources can be interleaved in this sequence. Note that, we have considered Google Fonts as 1P here.

| # (Main Thread) | Event | # (Network) | Request |
|---|---|---|---|
| 1 | Parse the HTML | 1 | Small inline 1P scripts |
| 2 | Execute small inline 1P scripts | 2 | Inlined critical CSS (Preload if external) |
| | | 3 | Inlined critical Fonts (Preconnect if external) |
| 3 | Parse FCP resources (critical CSS, font) | 4 | LCP Image (Preconnect if external) |
| **FCP** | | 5 | Fonts (triggered from inline font-css, Preconnect) |
| 4 | Render LCP resources (Hero image, text) | 6 | Non-critical (async) CSS |
| | | 7 | First-party JS for interactivity |
| | | 8 | Above the fold images (preconnect) |
| **LCP** | | 9 | Below the fold images |
| 5 | Render important ATF images | | |
| **Visually Complete** | | | |
| 6 | Parse Non-critical (async) CSS | | |
| 7 | Execute 1P JS and hydrate | 10 | Lazy-loaded JS chunks |
| **FID** | | | |

While some parts of this sequence may be intuitive, the following points will help to justify it further.

1. We recommend avoiding preload as much as possible because it forces manual preload on every preceding resource and also causes manual curation of ordering. Preload should be especially avoided on fonts, as it is tricky to detect critical fonts.
2. Font-CSS should be ideally inlined. Fonts from another origin should be fetched using preconnect.
3. Preconnect is recommended for all resources from another origin. This will ensure that a connection is established in advance for downloading these resources.
4. Non-critical CSS should be fetched before user interaction begins (FID). This would avoid styling problems due to subsequent rendering of such CSS.
5. Start fetching first-party JS before ATF images on the network. It will take some time to download and parse the JS.
6. Parsing of the HTML on the main thread and download of ATF images can continue in parallel while 1P JS is parsed.

#### Proposed Sequence with 3P

Finally, we can propose a sequence for all key resources that are commonly loaded in a modern web application. Following is what the sequence for events on the main browser thread and network fetch requests will look like with 3P resources in the picture.

| # | Main Thread Event | Network Request |
|---|---|---|
| 1 | Parse the HTML | FCP blocking 3P resources |
| 2 | | |
| 3 | | Small inline 1P scripts |
| 4 | Execute small inline 1P scripts | Inlined critical CSS (Preload if external) |
| 5 | Parse FCP blocking 3P resources | Inlined critical Fonts (Preconnect if external) |
| 6 | Parse FCP resources (critical CSS, font) | 3P personalized ATF image required for LCP |
| 7 | **FCP** | LCP Image (Preconnect if external) |
| 8 | Render 3P personalized ATF image required for LCP | Fonts (triggered from inline font-css, Preconnect) |
| 9 | | Non-critical (async) CSS |
| 10 | Render LCP resources (Hero image, text) | 3P that must execute before first user interaction |
| 11 | | First-party JS for interactivity |
| 12 | **LCP** | |
| 13 | Render important ATF images | Default 3P JS |
| 14 | Parse Non-critical (async) CSS | |
| 15 | Execute 3P required for first user interaction | Below the fold images |
| 16 | Execute 1P JS and hydrate | Lazy-loaded JS chunks |
| **FID** | | Less important 3P JS |

The main concern here is how do you ensure that 3P scripts are downloaded optimally and in the required sequence.

Since the script request goes to another domain, preconnect is recommended for the following 3P requests:

- #1 - FCP blocking 3P resources
- #5 - 3P personalized ATF image required for LCP
- #9 - 3P that must execute before first user interaction
- #12 - Default 3P JS

To achieve the desired sequence, we recommend using the ScriptLoader component for Next. This component is designed to "optimize the critical rendering path and ensure external scripts don't become a bottleneck to optimal page load." The feature most relevant to our discussion is Loading Priorities. This allows us to schedule the scripts at different milestones to support different use cases. Following are the loading priority values available:

**After-Interactive**: Loads the specific 3P script after the next hydration. This can be used to load Tag-managers, Ads, or Analytics scripts that we want to execute as early as possible but after 1P scripts.

**Before-Interactive**: Loads the specific 3P script before hydration. It can be used in cases where we want the 3P script to execute before the 1P script. Eg., polyfill.io, bot detection, security and authentication, user consent management (GDPR), etc.

**Lazy-Onload**: Prioritize all other resources over the specified 3P script and lazy load the script. It can be used for CRM components like Google Feedback or Social Network specific scripts like those used for share buttons, comments, etc.

Thus, preconnect, script attributes and ScriptLoader for Next.js together can help us get the desired sequence for all our scripts.

### Conclusion

The responsibility of optimizing apps falls on the shoulders of the creators of the platforms used as well as the developers who use it. Common issues need to be addressed. We aim to make sequencing easier from the inside out. A tried and tested set of recommendations for different use cases and initiatives like the Script Loader help to achieve this for the React-Next.js stack. The next step would be to ensure that new apps conform to the recommendations above.

## Source

- [patterns.dev/vanilla/loading-sequence](https://patterns.dev/vanilla/loading-sequence)
