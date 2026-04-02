---
name: bundle-splitting
description: Split your code into smaller bundles to reduce initial load time and improve performance.
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

# Bundle Splitting

When building a modern web application, bundlers such as [Webpack](https://webpack.js.org/) or [Rollup](https://rollupjs.org/guide/en/) take an application's source code, and bundle this together into one or more bundles. When a user visits a website, the bundle is requested and loaded in order to display the data to the user's screen.

JavaScript engines such as [V8](https://v8.dev/docs) are able to parse and compile data that's been requested by the user as it's being loaded. Although modern browsers have evolved to parse and compile the code as quickly and performantly as possible, the developer is still in charge of optimizing two steps in the process: the loading time and execution time of the requested data. We want to make sure we're keeping the execution time as short as possible to prevent blocking the main thread.

## When to Use

- Use this when your application has a large JavaScript bundle that affects load times
- This is helpful when you want to reduce First Contentful Paint (FCP) and Largest Contentful Paint (LCP)
- Use this when parts of your code are only needed for specific user interactions or routes

## Instructions

- Use bundlers like Webpack or Rollup to split code into multiple smaller bundles
- Separate code that isn't needed for the initial render into its own bundle
- Consider the impact on Time To Interactive (TTI) and prioritize critical rendering code

## Details

Even though modern browsers are able to stream the bundle as it arrives, it can still take a significant time before the first pixel is painted on the user's device. The bigger the bundle, the longer it can take before the engine reaches the line on which the first rendering call has been made. Until that time, the user has to stare at a blank screen.

We want to display data to the user as quickly as possible. A larger bundle leads to an increased amount of loading time, processing time, and execution time. It would be great if we could reduce the size of this bundle, in order to speed things up.

Instead of requesting one giant bundle that contains unnecessary code, we can split the bundle into multiple smaller bundles!

By bundle-splitting the application, we can reduce the time it takes to load, process and execute a bundle! By reducing the loading and execution time, we can reduce the time it takes before the first content has been painted on the user's screen, the First Contentful Paint (FCP), and the time it takes before the largest component has been rendered to the screen, the Largest Contentful Paint (LCP).

Although being able to see data on our screen is great, we don't just want to _see_ the content. In order to have a fully functioning application, we want users to be able to _interact_ with it as well! The UI only becomes interactive after the bundle has been loaded and executed. The time it takes before all content has been painted to the screen and has been made interactive, is called the Time To Interactive (TTI).

A bigger bundle doesn't necessarily mean a longer execution time. It could happen that we loaded a ton of code that the user won't even use! Maybe some parts of the bundle will only get executed on a certain user interaction, which the user may or may not do!

The engine still has to load, parse and compile code that's not even used on the initial render before the user is able to see anything on their screen. Although the parsing and compilation costs can be practically ignored due to the browser's performant way of handling these two steps, fetching a larger bundle than necessary can hurt the performance of your application. Users on low-end devices or slower networks will see a significant increase in loading time before the bundle has been fetched.

Instead of initially requesting parts of the code that don't have a high priority in the current navigation, we can separate this code from the code that's needed in order to render the initial page.

By bundle-splitting the large bundle into two smaller bundles, `main.bundle.js` and `emoji-picker.bundle.js`, we reduce the initial loading time by fetching a smaller amount of data.

## Source

- [patterns.dev/vanilla/bundle-splitting](https://patterns.dev/vanilla/bundle-splitting)
