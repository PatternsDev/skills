---
name: view-transitions
description: Animate page transitions using the View Transitions API for smooth visual DOM changes.
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

# Animating View Transitions

**Note**: The View Transitions API for Single-Page Applications is available in Chrome 111+

## When to Use

- Use this when you want to animate transitions between different page states or navigations
- This is helpful for creating polished, app-like navigation experiences in web applications

## Instructions

- Use `document.startViewTransition(callback)` to animate DOM changes
- Assign unique `view-transition-name` CSS properties to elements that should transition between states
- Check for browser support before using the API (`if (document.startViewTransition)`)
- Minimize the time the DOM is frozen by starting transitions after data fetching completes
- Consider CSS animation fallbacks for browsers that don't yet support the View Transitions API

## Details

### Introduction to View Transitions

The [View Transitions API](https://developer.chrome.com/docs/web-platform/view-transitions/) offers a simple way to transition any visual DOM change from one state to the next. This might include small changes such as toggling some content, or broader changes such as navigating from one page to the next.

The JavaScript API centers around `document.startViewTransition(callback)`, where `callback` is a function that typically updates the DOM to the new state.

Let's take toggling a `<details>` element as a simple example:

```js
if (document.startViewTransition) {
  // (check for browser support)
  document.addEventListener("click", function (event) {
    if (event.target.matches("summary")) {
      event.preventDefault(); // (we'll toggle the element ourselves)
      const details = event.target.closest("details");
      document.startViewTransition(() => details.toggleAttribute("open"));
    }
  });
}
```

`document.startViewTransition` takes a screenshot of the current DOM before calling the callback. Here, our callback just toggles the `open` attribute. Once complete, the browser can then transition between the initial screenshot and the new version.

These old and new versions are presented as pseudo elements and can be referenced in CSS with `::view-transition-old(root)` and `::view-transition-new(root)` respectively. For example, to emphasize the transition, we can lengthen the `animation-duration` like so:

```css
::view-transition-old(root),
::view-transition-new(root) {
  animation-duration: 2s;
}
```

View transitions are also capable of animating multiple changes with more advanced animations that go beyond the default crossfade. By giving specific elements a CSS `view-transition-name`, and a `containment` of `layout` or `paint`, the API gives developers granular control over how the elements transition, including their width, height, and position. These advanced transitions can really help communicate the flow from one page to the next.

Take a photo gallery as an example: the most obvious transition is the size and position of the photo, which is automatically achieved when the `<img>` element on each page is given the same unique `view-transition-name`, and a CSS `containment` value of `layout`. The `view-transition-name`s can be hard-coded in the style attributes, or added dynamically (e.g. in a `onclick` handler), as long as they're unique to the page and added before the transition is started.

The photo details beneath require a little more styling. We give each line element its own `view-transition-name`:

```css
figcaption h2 {
  contain: layout;
  view-transition-name: photo-heading;
}
figcaption div {
  contain: layout;
  view-transition-name: photo-location-time;
}
figcaption dl {
  contain: layout;
  view-transition-name: photo-meta;
}
```

This generates _transition groups_ for each area, which are just like the new/old screenshots mentioned earlier, but only cover an area of the page rather than the whole document. And just as the whole document transition elements could be targeted with `::view-transition-old(root)` and `::view-transition-new(root)`, these transition groups can be targeted with `::view-transition-old(NAME)` and `::view-transition-new(NAME)`. Note that the details text is not present on the photo grid page, therefore when transitioning from the grid to the photo page, there'll only be a `::view-transition-new(NAME)`, _not_ a `::view-transition-old(NAME)`, and vice versa when navigating the other way. So we can target these cases using the `:only-child` pseudo class and customize the animation. For the `photo-heading` group:

```css
/* Enter */
::view-transition-new(photo-heading):only-child {
  animation: 300ms ease 50ms both fade-in, 300ms ease 50ms both slide-up;
}

/* Exit */
::view-transition-old(photo-heading):only-child {
  animation: 200ms ease 150ms both fade-out, 200ms ease 150ms both slide-down;
}
```

That's the basics of the API. [Jake Archibald's excellent View Transitions article](https://developer.chrome.com/docs/web-platform/view-transitions) covers the details well. For now, let's see how we might transition full page navigations.

### Page Navigations

A typical page navigation looks something like:

1. User clicks a link
2. Request is made for data
3. DOM is updated with the response

To apply a view transition in this flow, there are a couple of considerations.

First, is minimizing the time that the screen is in a frozen state. You may have noticed that once a view transition has started, the DOM will be not interactive until the callback completes. If we start the transition when the user clicks the link, they could be waiting a while with a frozen UI. To minimize this annoyance, ideally `document.startViewTransition` should be called after the request has completed. That way, we're ready for the change, and the DOM can be updated as swiftly as possible.

Second, we need to be sure the initial DOM screenshot has been captured before we update the DOM. When working with page navigations in third-party frameworks, we don't have full control over the rendering process; the DOM is automatically updated when the response is received. Therefore we don't have a standalone function we can pass to `document.startViewTransition` that will tidily perform the DOM update. We may need to intercept, pause, and resume rendering to give the illusion we have a single function that updates the DOM.

Nicely enough, if we return a promise from our DOM update callback, the view transition API will wait for its resolution before performing the animation. We can use this feature to handle the timing issues mentioned above.

#### React Component Example

To tackle the issues above, we'll create a React class component as it's easier to explain the flow compared to a functional component. We'll use the following lifecycle methods to control rendering:

- `shouldComponentUpdate`: we'll return `false` here and start the view transition — this will buy us some time for the screenshot capture to complete
- `forceUpdate`: to manually re-render the component after the screenshot capture
- `componentDidUpdate`: to notify the view transition API that the DOM has updated

Here's how it looks:

```js
import { Component } from "react";

export default class ViewTransition extends Component {
  shouldComponentUpdate() {
    if (!document.startViewTransition) return true; // skip when not supported

    document.startViewTransition(() => this.#updateDOM());
    return false; // don't update the component, we'll do this manually
  }

  #updateDOM() {
    // now we know the screenshot has been taken, we can force render
    // (which skips `shouldComponentUpdate`)
    this.forceUpdate();
    // set up a promise that will resolve when the component renders
    return new Promise((resolve) => {
      this.#rendered = resolve;
    });
  }

  render() {
    return this.props.children;
  }

  #rendered = () => {};

  componentDidUpdate() {
    // resolve the `updateDOM` promise to notify the View Transition API
    // that the DOM has been updated
    this.#rendered();
  }
}
```

**Note:** The [Next.js App Router](https://beta.nextjs.org/docs/app-directory-roadmap) is in beta at the time of writing and best-practices around it and the pages directory may be subject to change.

To use this in a Next.js app, first we'll disable React strict mode in development. Strict mode runs its checks by rendering the component twice. This interferes with the `ViewTransition` rendering flow in development so we'll disable it globally and re-enable it for child components with the `StrictMode` component.

```js
// next.config.js
const nextConfig = {
  reactStrictMode: false,
};

module.exports = nextConfig;
```

Next, in `pages/_app.js`, we'll wrap `Component` in our `ViewTransition` and `StrictMode` component, and we should begin to see animated transitions:

```js
// pages/_app.js
import "@/styles/globals.css";
import { StrictMode } from "react";
import ViewTransition from "@/components/ViewTransition";

export default function App({ Component, pageProps }) {
  return (
    <ViewTransition>
      <StrictMode>
        <Component {...pageProps} />
      </StrictMode>
    </ViewTransition>
  );
}
```

View the [Next.js demonstration](https://frolicking-dusk-29be0a.netlify.app/), the [live Next.js demo](https://photography-view-transitions-nextjs.vercel.app/) and its [source](https://github.com/domchristie/photography-view-transitions-nextjs).

**Note**: the React documentation advises against using `shouldComponentUpdate` and `forceUpdate`, stating they should only be used for performance optimizations, and that `shouldComponentUpdate` is not guaranteed to be called. As page animations are an enhancement, and this component will work even if `shouldComponentUpdate` is not called, this caveat is acceptable.

#### An Alternative Approach without View Transitions

One necessary downside of the View Transitions API for page transitions is that it needs the new page HTML before animating. This can take time and leaves the user without any feedback after they click a link. A spinner may fill the gap, but we could buy some time by animating out elements as soon as the user clicks a link, then animate in the new HTML when it arrives. This is similar to how standard iOS navigations slide across immediately whilst loading the next screen.

1. User clicks a link
2. Elements are animated out; meanwhile the request is made for data
3. Wait for both the response and the animations to complete
4. Animate in the response

The main difference between this approach and that of the View Transitions API, is that it can't _transition_ elements between one state to the next because at the time it animates out, it doesn't have the new HTML in order to do so.

Both approaches are useful depending on the situation. For example, if there are shared elements from one page to the next, you might opt for a view transition, whereas if the change is significant with few shared elements, you could benefit from the immediate feedback of an exit animation.

To implement this, we'll need to hook into routing events, which will depend on the framework or library you're using. In particular, we'll need to be notified when the user navigates. With Next.js, we can use the [`routeChangeStart` router event](https://nextjs.org/docs/api-reference/next/router#routerevents) to start the exit animations, but let's look at how we might achieve this _without_ Next.js, React, or fully client-rendered HTML.

#### Animating Server-side Rendered Multi-page Applications with Turbo and Turn

**Note**: There are plans for the View Transition API to work for multi-page navigations, i.e. without JavaScript. However, the JavaScript API may still be needed for more advanced transitions.

[Turbo](https://turbo.hotwired.dev/), part of the [Hotwire](https://hotwired.dev/) suite of libraries (not to be confused with [Vercel's Turbo](https://turbo.build/)), offers a rendering approach that progressively enhances multi-page applications (MPAs). It aims to achieve SPA speeds without having to architect your code as a fully client-rendered application, and does so by capturing link clicks and form submissions, performing the request with JavaScript, and replacing the `<body>` with the new `<body>` from the response. In this way, it's a hybrid approach: the HTML is generated on the server, but the DOM is updated via JavaScript.

[Turn](https://github.com/domchristie/turn) is a library for animating page navigations using Turbo. It supports both animation approaches (although currently view transitions are experimental). Turn adds `turn-before-exit`, `turn-exit`, and `turn-enter` classes to the `<html>` element at the appropriate times, providing a way for developers to customize the animations.

To get it working, add `data-turn-exit` and `data-turn-enter` attributes to the elements you wish to animate, then apply your CSS styles. For example, for a fade-in/fade-out:

```css
html.turn-exit [data-turn-exit] {
  animation-name: fade-out;
  animation-duration: 0.3s;
  animation-fill-mode: forwards;
}

html.turn-enter [data-turn-enter] {
  animation-name: fade-in;
  animation-duration: 0.6s;
  animation-fill-mode: forwards;
}

@keyframes fade-out {
  0% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}

@keyframes fade-in {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}
```

Then import the `Turn` library into your application's JavaScript and call `Turn.start()`.

It works by hooking into Turbo's rendering events, and controlling the flow as needed:

1. `turbo:visit`: just before the request starts, add the `turn-exit` class
2. `turbo:before-render`: after the request has completed but before the new HTML renders (similar to React's `shouldComponentUpdate`), pause rendering to wait for any exit animations to complete
3. `turbo:render`: once the new HTML has been rendered, remove `turn-exit` class and add the `turn-enter` class
4. once the exit animations complete, remove the `turn-enter` class

Turn also has experimental support for view transitions, enabled by setting `Turn.config.experimental.viewTransitions = true`. This will use view transitions where supported, and fallback to the CSS animation approach.

### Summary

Page transitions can be a great way to communicate changes from one page to the next. The new built-in View Transitions API can perform complex transitions when provided with the old and new states. By hooking into framework events, we can communicate to the API these state changes. For page navigations, ideally the transitions should occur after the request has finished to avoid the DOM being in an inactive state.

An alternative (or complementary) approach is to perform exit animations immediately, as soon as the user has clicked a link. This has the benefit of buying some time for the request to complete before the new HTML arrives.

## Source

- [patterns.dev/vanilla/view-transitions](https://patterns.dev/vanilla/view-transitions)
