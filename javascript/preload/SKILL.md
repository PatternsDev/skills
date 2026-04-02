---
name: preload
description: Inform the browser of critical resources before they are naturally discovered to speed up loading.
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

# Preload

[Preload](https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content) (`<link rel="preload">`) is a [browser optimization](https://web.dev/uses-rel-preload/) that allows critical resources (that may be discovered late) to be requested earlier. If you are comfortable thinking about how to manually order the loading of your key resources, it can have a positive impact on loading performance and metrics in the [Core Web Vitals](https://web.dev/vitals). That said, preload is not a panacea and requires an awareness of some trade-offs.

## When to Use

- Use this when critical resources (fonts, scripts, images) are discovered late in the loading process
- This is helpful for improving Time To Interactive (TTI) and Largest Contentful Paint (LCP)

## Instructions

- Use `<link rel="preload">` for resources needed immediately on the current page
- Be careful not to delay First Contentful Paint by preloading too many resources
- Use `as` attribute to specify the resource type (script, style, font, image)
- For fonts and other CORS-fetched resources, set `crossorigin` on the preload to match the eventual request mode
- Only preload resources that must be visible within ~1 second of initial render

## Details

```html
<link rel="preload" href="emoji-picker.js" as="script">
...
</head>
<body>
  ...
  <script src="stickers.js" defer></script>
  <script src="video-sharing.js" defer></script>
  <script src="emoji-picker.js" defer></script>
```

When optimizing for metrics like [Time To Interactive](https://web.dev/tti) or [First Input Delay](https://web.dev/fid), preload can be useful to load JavaScript bundles (or chunks) that are necessary for interactivity. Keep in mind that great care is needed when using preload as you want to avoid improving interactivity at the cost of delaying resources (like hero images or fonts) necessary for [First Contentful Paint](https://web.dev/fcp) or [Largest Contentful Paint](https://web.dev/lcp).

If you are trying to optimize the loading of first-party JavaScript, you can also consider using `<script defer>` in the document `<head>` vs. `<body>` to help with early discover of these resources.

### Preload in single-page apps

While **prefetching** is a great way to cache resources that may be requested some time soon, we can **preload** resources that need to be used instantly. Maybe it's a certain font that is used on the initial render, or certain images that the user sees right away.

Say our `EmojiPicker` component should be visible instantly on the initial render. Although it should not be included in the main bundle, it _should_ get loaded in parallel. Just like _prefetch_, we can add a magic comment in order to let Webpack know that this module should be preloaded.

```js
const EmojiPicker = import(/* webpackPreload: true */ "./EmojiPicker");
```

> Webpack 4.6.0+ allows preloading of resources by adding `/* webpackPreload: true */` to the import. In order to make preloading work in older versions of webpack, you'll need to add the [`preload-webpack-plugin`](https://github.com/GoogleChromeLabs/preload-webpack-plugin) to your webpack config.

After building the application, we can see that the `EmojiPicker` will be preloaded.

```
 Asset                             Size       Chunks                          Chunk Names
    emoji-picker.bundle.js         1.49 KiB   emoji-picker [emitted]          emoji-picker
    vendors~emoji-picker.bundle.js 171 KiB    vendors~emoji-picker [emitted]  vendors~emoji-picker
    main.bundle.js                 1.34 MiB   main  [emitted]                 main

Entrypoint main = main.bundle.js
(preload: vendors~emoji-picker.bundle.js emoji-picker.bundle.js)
```

The actual output is visible as a `link` tag with `rel="preload"` in the `head` of our document.

```html
<link rel="preload" href="emoji-picker.bundle.js" as="script" />
<link rel="preload" href="vendors~emoji-picker.bundle.js" as="script" />
```

The preloaded `EmojiPicker` could be loaded in parallel with the initial bundle. Unlike `prefetch`, where the browser still had a say in whether it thinks it's got a good enough internet connection and bandwidth to actually prefetch the resource, a **preloaded** resource will get preloaded no matter what.

Instead of having to wait until the `EmojiPicker` gets loaded after the initial render, the resource will be available to us instantly! As we're loading assets with smarter ordering, the initial loading time may increase significantly depending on your users device and internet connection. Only preload the resources that have to be visible ~1 second after the initial render.

### Preload + the `async` hack

Should you wish for browsers to download a script as high-priority, but not block the parser waiting for a script, you can take advantage of the preload + async hack below. The download of other resources may be delayed by the preload in this case, but this is a trade-off a developer has to make:

```html
<link rel="preload" href="emoji-picker.js" as="script">
<script src="emoji-picker.js" async>
```

### Font preloads must use `crossorigin`

Fonts are fetched as CORS resources, even when they are self-hosted on the same origin. This means the preload request and the eventual `@font-face` request need to use the same fetch mode, or the preload cannot be reused.

If you preload a font **without** `crossorigin`, the browser will typically make a `no-cors` preload request and later a separate `cors` request when CSS discovers the font. That leads to a double fetch of the same file and wastes bandwidth.

**Avoid:**

```html
<link rel="preload" href="/fonts/inter-roman.woff2" as="font" type="font/woff2">
```

**Prefer:**

```html
<link
  rel="preload"
  href="/fonts/inter-roman.woff2"
  as="font"
  type="font/woff2"
  crossorigin
>
```

And make sure the `@font-face` matches the same resource:

```css
@font-face {
  font-family: "Inter";
  src: url("/fonts/inter-roman.woff2") format("woff2");
  font-display: swap;
}
```

This same rule applies more broadly: if the eventual consumer fetches a resource with CORS semantics, the preload should match that mode too.

### Preload in Chrome 95+

Thanks to some [fixes](https://twitter.com/patmeenan/status/1436374668450177026) to preload's [queue-jumping](https://docs.google.com/document/d/1ZEi-XXhpajrnq8oqs5SiW-CXR3jMc20jWIzN5QRy1QA/edit?usp=sharing) behavior in Chrome 95+, the feature is slightly safer to use more broadly. Pat Meenan of Chrome's new recommendations for preload suggest:

- Putting it in HTTP headers will jump ahead of everything else
- Generally, preloads will load in the order the parser gets to them for anything >= Medium so be careful putting preloads at the beginning of the HTML.
- Font preloads are probably best towards the end of the head or beginning of the body
- Import preloads should be done after the script tag that needs the import (so the actual script gets loaded/parsed first)
- Image preloads will have a low priority and should be ordered relative to async scripts and other low/lowest priority tags

### Conclusions

Again, use preload sparingly and always measure its impact in production. If the preload for your image is earlier in the document than it is, this can help browsers discover it (and order relative to other resources). When used incorrectly, preloading can cause your image to delay First Contentful Paint (e.g CSS, Fonts) - the opposite of what you want. Also note that for such reprioritization efforts to be effective, it also depends on [servers prioritizing requests](https://github.com/andydavies/http2-prioritization-issues#cdns--cloud-hosting-services) correctly.

You may also find `<link rel="preload">` to be helpful for cases where you need to fetch scripts [without executing](https://developer.mozilla.org/en-US/docs/Web/HTML/Preloading_content#scripting_and_preloads) them.

A variety of web.dev articles touch on how to use Preload to:

- [Preload key scripts required for interactivity](https://web.dev/uses-rel-preload/)
- [Preload your Largest Contentful Paint image](https://web.dev/preload-responsive-images/)
- [Load fonts while preventing layout shifts](https://web.dev/preload-optional-fonts/)

## Source

- [patterns.dev/vanilla/preload](https://patterns.dev/vanilla/preload)
