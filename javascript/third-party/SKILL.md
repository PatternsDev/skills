---
name: third-party
description: Reduce the performance impact of third-party scripts on your site using optimized loading strategies.
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

# Optimize loading third-parties

> tl;dr: Third-party resources can slow down sites and can be a challenge to optimize. You can follow certain best practices to load or delay different types of third-parties efficiently. You can also use framework-level components such as the [Next.js Script component](https://nextjs.org/docs/basic-features/script), which provides a template for framing the "when" and "how" for loading third-party scripts. Alternatively, experimental ideas like [Partytown](https://github.com/BuilderIO/partytown) may be of interest.

## When to Use

- Use this when third-party scripts (analytics, ads, chat widgets, social embeds) are slowing down your site
- This is helpful for optimizing Core Web Vitals while retaining essential third-party functionality

## Instructions

- Use `async` or `defer` attributes for non-critical third-party scripts
- Establish early connections with `preconnect` and `dns-prefetch` resource hints
- Lazy-load below-the-fold embeds (YouTube, maps, social media) using IntersectionObserver or facades
- Consider self-hosting critical third-party scripts for better caching control
- Use the Next.js Script component with appropriate `strategy` values for framework-level optimization

## Details

It would be hard to find a modern site that operates in silos. Most sites coexist and depend on several other sources on the web for data, functions, content, and much more. Any resource located on another domain and consumed by your site is a third-party (3P) resource for your site. Typical third-party resources that are included on sites include:

- Embeds for maps, videos, social media, and chat services
- Advertisements
- Analytics components and tag managers
- A/B testing and personalization scripts
- Utility libraries that provide ready-to-use helper functions such as those used for data visualization or animations.
- [reCAPTCHA](https://www.google.com/recaptcha/about/) or CAPTCHA for bot detection.

You can use third-parties to integrate other features that add value to your content or to reduce some drudgery involved in building a site from scratch. As per the Web Almanac report for 2021, more than [94% of the pages](https://almanac.httparchive.org/en/2021/third-parties#prevalence) on the web use third-parties - [images and JavaScript](https://almanac.httparchive.org/en/2020/third-parties#content-types) forming the most significant contributors to third-party content.

While third-party resources can enrich your site with valuable features, they can also slow it down if:

- They cause additional round trips to the third-party domain for every required resource.
- They make heavy usage of JavaScript (impacting download and execution time) or are bulky in size because of unoptimized images/videos.
- Individual site owners cannot influence the implementation, and their behavior can be unpredictable.
- They can block the rendering of other critical resources on the page and affect [Core Web Vitals](https://web.dev/vitals/) (CWV).

Despite these issues, third-parties may be essential to your business. If you cannot do away with 3P's, the next best thing is to optimize them to reduce performance impact.

### Assessing the performance impact of 3P resources

You can use a combination of techniques to find how third-party code is affecting your site.

- The following Lighthouse audits help identify slow third-party scripts that affect CWV:
  - [Reduce the impact of third-party code](https://web.dev/third-party-summary/) for scripts that block the main thread.
  - [Reduce JavaScript execution time](https://web.dev/bootup-time/) for scripts that take long to execute
  - [Avoid enormous network payloads](https://web.dev/total-byte-weight/) for large scripts

- Use the WebPageTest (WPT) waterfall chart to identify [third-party blocking scripts](https://nooshu.com/blog/2019/10/02/how-to-read-a-wpt-waterfall-chart/#third-party-blocking-javascript) or WPT side-by-side comparison to [measure the impact of 3rd party tags](https://andydavies.me/blog/2018/02/19/using-webpagetest-to-measure-the-impact-of-3rd-party-tags/).
- Sites like [Bundlephobia](https://bundlephobia.com/) help to assess the cost of adding available npm packages to your bundles. You can also find size and dependencies included in any package using [npm package search](https://www.npmjs.com/package/).

### Optimization strategies

Since third-party code is not under your control, you cannot optimize the libraries directly. This leaves you with two options.

1. **Replace or remove**: If the value provided by the third-party script is not proportional to its performance cost, consider removing it. You can also evaluate other alternatives that are lightweight but provide similar functionality.

2. **Optimize the loading sequence**: The loading process involves loading several first-party and third-party resources in the browser. To design an optimal loading strategy, you would need to consider the browser assigned priority for different resources, their position on the page, and the value of each resource for the web page.

#### Load 3P scripts efficiently

Following are the time-tested best practices that can reduce the performance impact of third-party resources when used correctly.

#### Use async or defer to prevent scripts from blocking other content.

**Applicable to:** Non-critical scripts (tag managers, analytics)

JavaScript download and execution is synchronous by default and can block the HTML parser and DOM construction on the main thread. Use of [async](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#attr-async) or [defer](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#attr-defer) attributes in the `<script>` element tells the browser to download scripts asynchronously. You can use these to download any script that is not necessary for the critical rendering path (e.g., the main UI component).

- **`defer`**: the script is fetched in parallel as the parser executes, and script execution is delayed till the parsing is complete. Defer should be the default choice for delaying execution until after DOM construction.
- **`async`**: the script is fetched in parallel while parsing but executed as soon as it is available when it blocks the parser. For module scripts with dependencies, the script and all its dependencies are executed in the defer queue. Use `async` for scripts that need to run earlier in the loading process. For example, you may want to execute specific analytics scripts early without missing any early page-load data.

```html
<script src="https://example.com/deferthis.js" defer></script>
<script src="https://example.com/asyncthis.js" async></script>
```

> Credit: [developers.google.com](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/loading-third-party-javascript)

One caveat worth mentioning here is that async and defer lower the browser assigned priority of the resources causing it to load significantly later. A new feature for [priority hints](https://web.dev/priority-hints/) can help to work around this problem.

#### Establish early connections to required origins using resource hints

**Applicable to:** Critical scripts, fonts, CSS, images from third-party CDNs

Connecting to third-party origins can be slow due to the DNS lookups, redirects, and multiple round trips that may be required for each third-party server. Resource hints dns-prefetch and preconnect help cut down the time needed for this setup by initiating the connections early in the life cycle.

Including a [dns-prefetch](https://developer.mozilla.org/en-US/docs/Web/Performance/dns-prefetch) resource hint corresponding to a domain will perform the DNS lookup early, thus reducing the latency associated with dns lookups. You can pair this with [preconnect](https://developer.mozilla.org/en-US/docs/Web/HTML/Link_types/preconnect) for the most critical resources. The preconnect initiates a connection with the third-party domain by performing TCP round trips and handling TLS negotiations in addition to the DNS lookup.

```html
<head>
  <link rel="preconnect" href="http://example.com" />
  <link rel="dns-prefetch" href="http://example.com" />
</head>
```

Benefits of using resource hints are evident in this case study where Andy Davies discusses how using [preconnect helped reduce the loading time](https://andydavies.me/blog/2019/03/22/improving-perceived-performance-with-a-link-rel-equals-preconnect-http-header/) for the main product image by initiating an early connection to the third-party image CDN.

> "The real-world metrics showed a 400ms improvement at the median and greater than 1s improvement at the 95th percentile."

Similarly, you may use resource hints to optimize loading time for critical third-parties like bot detection (reCaptcha) and consent management.

#### Lazy load below-the-fold 3P resources

**Applicable to:** Embeds such as YouTube, Maps, Advertisements and Social media

Third-party embeds like those used for social media feeds, advertisements, YouTube videos, and maps can slow down web pages. However, all such embeds may not be visible to users on page load and may be lazy-loaded as the user scrolls down to them. You can use different lazy-loading methods depending on desired browser support.

1. The [loading](https://web.dev/iframe-lazy-loading/) attribute can be used with images and iframes commonly used to load third-party embeds like those for YouTube or Google Maps.
2. A custom implementation using the [IntersectionObserver API](https://developers.google.com/web/updates/2016/04/intersectionobserver) allows you to detect when the observed element enters or exits the browser's viewport.
3. [Lazy-sizes](https://github.com/aFarkas/lazysizes) - a popular JavaScript library that implements lazy-loading for you.

A variation of lazy-loading embeds uses a static or dynamic facade displayed to users on page loads. Instead of the map embed, you can use a static image of the actual embed. Solutions include the [Map Static API](https://developers.google.com/maps/documentation/maps-static/overview) for maps, [Tweetpik](https://tweetpik.com/) for Twitter embeds, [lite-youtube-embed](https://github.com/paulirish/lite-youtube-embed) for YouTube, [React-live-chat-loader](https://github.com/calibreapp/react-live-chat-loader) for chat widgets. A comprehensive discussion on these techniques is available [here](https://web.dev/embed-best-practices/).

A few caveats concerning lazy-loading and facades:

- The behavior for the YouTube facade is slightly different on iOS and Safari on macOS 11+. Tapping/clicking the first time loads the actual video embed. Users will have to tap again to play the video.
- Lazy loading can lead to layout shifts and affect the user experience if the size of the embed is not specified. To prevent layout shifts, you should specify the size for all lazy-loaded embeds or their container elements.

#### Self-host 3P scripts to prevent round trips

**Applicable to:** JavaScript files, fonts

Although preconnect or dns-prefetch allows you to initiate connections to third-party origins early, the connections are still required. Also, with third-party origins, you need to rely on their caching strategy, which may not be optimal.

Self-hosting a copy of the scripts on the same origin offers you more control over the loading and caching process used for the scripts. Self-hosting reduces the time required for DNS lookup and allows you to improve the caching strategy for the scripts using [HTTP caching](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching). You can also use [HTTP/2 server push](https://www.smashingmagazine.com/2017/04/guide-http2-server-push/) to push scripts that you know the user will need. A great example of how self-hosting third-party scripts is [Casper.com](https://casper.com/) which improved the start render time for its home page by 1.7 seconds by self-hosting third-party scripts provided by [Optimizely](https://www.optimizely.com/).

With self-hosted copies of third-party scripts, you have to ensure that you update your copy regularly based on changes to the original. Without updates, the script may become outdated, missing important fixes or changes corresponding to dependencies. Self-hosting on a server instead of a CDN will also prevent you from taking advantage of [edge-caching](https://www.cloudflare.com/learning/cdn/glossary/edge-server/) mechanisms employed by CDNs.

#### Use service workers to cache scripts where possible

**Applicable to:** JavaScript files, fonts

Self-hosting may not be an option for scripts that change frequently. You can use service workers to improve caching for such third-party scripts while also utilizing CDN edge caching. This technique gives you better control over the frequency of re-fetches over the network. This technique can be combined with preconnects to further reduce the network cost of the fetch operation. You can also load resources such that requests for non-essential third-party scripts are deferred till the page reaches a key user moment.

#### Follow the ideal loading sequence

Consider the above guidance for different types of third-parties and their value to the page. Based on the intended use for each resource, you can follow the ideal resource loading sequence to interlace first-party and third-party resources optimally for faster page loads.


### Best practices by script type

Some scripts are easier to optimize than others. The general consensus was that most users do not interact with a site until a certain threshold of content is visible.

#### Non-critical JavaScript

Most third-parties like chat widgets or analytics scripts are not critical to the user experience and can be delayed. Using the `defer` script attribute is the most common method to delay the loading and execution of these scripts.

Advertising or analytics teams may worry about the impact of deferring scripts on the visibility and advertising revenue of the app. The [Telegraph case study](https://medium.com/the-telegraph-engineering/improving-third-party-web-performance-at-the-telegraph-a0a1000be5) is often cited in this context, where deferring all scripts did not skew any analytics or advertising metrics. Instead, the First Ad Loaded metric improved by an average of 4 seconds.

#### Bot detection/ReCaptcha

Since you would want to prevent bots from accessing web forms, developers would usually load these scripts as early as possible. However, ReCaptcha has a sizable JS payload and main thread footprint, so there is motivation to defer loading it until required. Few methods to optimize this script are:

1. Load it only on a few pages with form inputs from the user that may get spammed by a bot.
2. [Lazy load the script](https://dev.to/uf4no/improve-page-performance-lazy-loading-recaptcha-442o) when the user interacts with form elements, for example, on form focus.
3. Use resource hints to establish early connections when you need the script to execute on page load.

#### Google Tag Manager (GTM)

Large sites often provide [Google Tag Manager](https://marketingplatform.google.com/about/resources/tag-manager-product-overview/) access to marketing teams or agencies. This allows them to add new marketing tags to all pages on the site for better tracking. Performance is not a primary concern for the marketing team, and all of them may not know that inconsiderately adding tags can slow down the site. Optimization of GTM scripts is more about [controlling who accesses GTM](https://www.tunetheweb.com/blog/adding-controls-to-google-tag-manager/) and monitoring the changes that they make.

You can start by ensuring that the site owners own the account rather than an external agency. This allows you to define granular access permissions for who can add, edit and publish tags. Better collaboration between development and marketing departments can be set up to audit new tags and remove unused tags.

Your site may not need GTM on all pages. Pages should be audited individually so that unnecessary GTM inclusions can be removed. Sites that use cookie banners can also opt not to load GTM if the user declines cookies. Finally, if you must load GTM on a page, you can defer the scripts to fire after loading the main content.

#### A/B Testing and Personalization

Sites conduct [A/B tests](https://www.optimizely.com/optimization-glossary/ab-testing/) to check which version of the webpage performs better. A/B tests can significantly affect the performance of pages on which they are run, with each test adding as much as 1 sec to the loading time. Currently, many A/B tests are sourced externally through third-parties, and developers have little control over the JavaScript code executed to alter the UI for these tests.

[Site personalization](https://www.dynamicyield.com/lesson/web-personalization/) is a related concept which involves running scripts to provide a tailored experience for different users based on known data. These scripts are again heavy and difficult to optimize. Like A/B testing scripts, personalization scripts also need to run early as the rendered UI depends on the script's output. Developing a [custom server-based solution for A/B testing](https://www.fasterize.com/en/blog/a-b-testing-imperative-to-marketing-perilous-to-web-performance/) and personalization is the ideal method to optimize A/B testing. However, it may not always be feasible.

To optimize third-party A/B testing scripts, you can limit the number of users who receive the script. Google's Optimize allows the configuration of [rules for targeting users](https://support.google.com/optimize/answer/6283420#zippy=%2Cin-this-article).

#### YouTube and map embeds

These embeds are heavy, and developers must explore lazy-loading or click-to-load patterns to load the embeds to optimize them. Use of solutions like [lite-youtube-embed](https://github.com/paulirish/lite-youtube-embed) is encouraged while noting that double-tap/click is required in iOS/macOS-Safari to play the video using this facade.

#### Social media embeds

Some social media embeds provide an option to lazy-load their scripts (e.g., [data-lazy in Facebook](https://developers.facebook.com/docs/plugins/embedded-posts/) embeds). You can explore this to improve performance. Another alternative is to use image facades created manually or using tools like [tweetpik](https://tweetpik.com/).


### Out-of-the-box optimization

To optimize third-parties, development teams should understand the nuances of resource hints, lazy loading, HTTP caching, and service workers and then implement these in their solutions. Some frameworks and libraries have encapsulated these best practices in a way that developers can easily use.

[Partytown created by Builder.io](https://github.com/BuilderIO/partytown) is an experimental library that helps run resource-intensive scripts on a [web worker](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API) instead of the main thread. Their philosophy is that the main thread should be dedicated to your code, and any scripts that are not required by the critical path can be sandboxed and isolated to a web worker. Partytown allows you to configure access to the main thread APIs such as cookies, localStorage, userAgent, etc. API calls may also be logged with arguments to get a better insight into what the scripts do.

JavaScript proxies and a service worker handle communication between the web worker and the main thread. Partytown scripts must be self-hosted on the same server as the HTML documents. It may be used with React or Next.js apps or even without any framework. Each third-party script that can execute in a web server should set the type attribute of its opening script tag to text/partytown as follows.

```html
<script type="text/partytown">// Third-party analytics scripts</script>
```

The library also provides a React Partytown component that you can directly include in your React or Next.js projects:

```js
import { Partytown } from '@builder.io/partytown/react';
import Document, { Html, Head, Main, NextScript } from 'next/document';

export default class MyDocument extends Document {
 render() {
   return (
     <Html>
       <Head>
         <Partytown />
       </Head>
       <body>
         <Main />
         <NextScript />
       </body>
     </Html>
   );
 }
```

Partytown also includes React components for common analytics libraries such as Google Tag Manager:

```js
import { Partytown, GoogleTagManager, GoogleTagManagerNoScript } from '@builder.io/partytown/react';
import Document, { Html, Head, Main, NextScript } from 'next/document';

export default class MyDocument extends Document {
 render() {
   return (
     <Html>
       <Head>
         <GoogleTagManager containerId={'GTM-XXXXX'} />
         <Partytown />
       </Head>
       <body>
         <GoogleTagManagerNoScript containerId={'GTM-XXXXX'} />
         <Main />
         <NextScript />
       </body>
     </Html>
   );
 }
```


### Next.js `Script` component

Next.js 11 was released in mid-2021 with components based on the [Conformance](https://web.dev/conformance/) methodology introduced by Google's Aurora team. The [Next.js Script component](https://nextjs.org/docs/basic-features/script) uses conformance by providing a customizable template that improves loading performance. The Script component encapsulates the `<script>` tag and allows you to set the loading priority for third-party scripts using the strategy attribute. The strategy attribute can take three values:

1. **beforeInteractive**: Use this for critical scripts that the browser should execute before the page becomes interactive. (e.g., bot detection)
2. **afterInteractive**: Use this for scripts that the browser can run after the page is interactive. (e.g., tag managers) This is the default strategy applied and is equivalent to loading the script with `defer`
3. **lazyOnload**: Use this for scripts that can be lazily loaded when the browser is idle.

Setting the strategy helps Next.js automatically apply the optimizations and best practices to load the script while ensuring the best loading sequence.

Before:

```js
import Head from "next/head";

export default function Home() {
  return (
    <>
      <Head>
        <script async src="https://example.com/samplescript.js" />
      </Head>
    </>
  );
}
```

After:

```js
import Script from 'next/script'

export default function Home() {
 return (
   <>
     <Script src="https://example.com/samplescript.js" />
   </>
 )
}
```

#### Load polyfills early

In situations where you want specific polyfills that apply to core content to be loaded early, you can use the beforeInteractive strategy:

```js
import Script from "next/script";

export default function Home() {
  return (
    <>
      <Script
        src="https://polyfill.io/v3/polyfill.min.js?features=IntersectionObserverEntry%2CIntersectionObserver"
        strategy="beforeInteractive"
      />
    </>
  );
}
```

#### Lazy-load social media embeds

Social media embeds, especially those not visible on page load, can be delayed or lazy-loaded. You can use the `lazyOnload` strategy:

```js
import Script from "next/script";

export default function Home() {
  return (
    <>
      <Script
        src="https://connect.facebook.net/en_US/sdk.js"
        strategy="lazyOnload"
      />
    </>
  );
}
```

#### Execute code conditionally on load

There may be some code that needs to execute after a specific third-party has been loaded. This can be specified in the onLoad attribute of the script component:

```js
<Script
  src={url} // consent management
  strategy="beforeInteractive"
  onLoad={() => {
    // If loaded successfully, then you can load other scripts in sequence
  }}
/>
```

#### Using inline scripts within the script tag

Inline scripts that need to execute based on load of a third-party component may also be included in the Script component:

```js
import Script from 'next/script'

<Script id="show-banner" strategy="lazyOnload">
 {`document.getElementById('banner').removeClass('hidden')`}
</Script>

// or

<Script
 id="show-banner"
 dangerouslySetInnerHTML={{
   __html: `document.getElementById('banner').removeClass('hidden')`
 }}
/>
```

Here the inline script is used to change the visibility of a third-party banner ad after it is lazy-loaded. Note that inline scripts may also be included using the dangerouslySetInnerHTML attribute.

#### Forward attributes to third-party scripts

You can set specific attribute values that can be used by the third-party script in the Script component:

```js
import Script from "next/script";

export default function Home() {
  return (
    <>
      <Script
        src="https://www.google-analytics.com/analytics.js"
        id="analytics"
        nonce="XUENAJFW"
        data-test="analytics"
      />
    </>
  );
}
```

#### Load analytics scripts

There are different ways to include analytics on your site, using Google Analytics (GA) and Google Tag Manager (GTM). You can use the Script component to load gtag.js or analytics.js scripts optimally on your Next.js site.

GTM may be enabled for all the pages on the site by including the script component inside \_app.js:

```js
import Script from "next/script";

function MyApp({ Component, pageProps }) {
  return (
    <>
      {/* Google Tag Manager - Global base code */}
      <Script
        strategy="afterInteractive"
        dangerouslySetInnerHTML={{
          __html: `
           (function(w,d,s,l,i){w[l]=w[l]||[];w[l].push({'gtm.start':
           new Date().getTime(),event:'gtm.js'});var f=d.getElementsByTagName(s)[0],
           j=d.createElement(s),dl=l!='dataLayer'?'&l='+l:'';j.async=true;j.src=
           'https://www.googletagmanager.com/gtm.js?id='+i+dl;f.parentNode.insertBefore(j,f);
           })(window,document,'script','dataLayer', '${GTM_ID}');
         `,
        }}
      />
      <Component {...pageProps} />
    </>
  );
}
export default MyApp;
```

Note that in both examples above, the analytics scripts are loaded with strategy = afterInteractive.


### Conclusion

When composing your web pages combining resources from your servers with those from other corners of the web, you must monitor the interplay between these resources frequently. You could start by sequencing the resources correctly and following best practices. You can also rely on frameworks or solutions that have built-in these best practices into their design.

As the site grows, performance reporting and regular audits can help eliminate redundancies and optimize scripts that affect performance. Lastly, we can always hope that third-parties with commonly known performance issues will optimize code at their end or expose APIs that enable workarounds to address these issues.

## Source

- [patterns.dev/vanilla/third-party](https://patterns.dev/vanilla/third-party)
