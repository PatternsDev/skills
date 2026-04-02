---
name: async-components
description: Optimize web app performance by loading components asynchronously only when they are needed.
paths:
  - "**/*.vue"
license: MIT
metadata:
  author: patterns.dev
  version: "1.1"
related_skills:
  - "composables"
  - "components"
---

# Async Components

When developing large web applications, performance is paramount. The speed with which a page loads and the responsiveness of its interactive elements can greatly impact user experience. As web applications grow in size and complexity, it can become important to ensure that large bundles of code are loaded only when needed. Enter asynchronous components in Vue.

Components are the fundamental building blocks for constructing the UI. Typically, when we use components, they're automatically loaded and parsed, even if they aren't immediately needed.

## When to Use

- Use this when components have large bundle sizes and aren't needed on initial page load
- This is helpful for modals, dialogs, or any UI that is conditionally rendered based on user action

## Instructions

- Use `defineAsyncComponent()` with dynamic `import()` to load components on demand
- Provide `loadingComponent` and `errorComponent` options for better user experience
- Combine with `v-if` to trigger async loading only when the component is actually needed
- Use the `delay` and `timeout` options for fine-grained control over loading behavior

## Details

Asynchronous components, on the other hand, allow us to define components in a way that they're loaded and parsed only when they're required or when certain conditions are met.

Assume we had a simple modal component that becomes rendered when a button is clicked from the parent. The `Modal.vue` component file will only contain template and styles that dictate how the modal appears.

```html
<template>
  <div class="modal-mask">
    <div class="modal-container">
      <div class="modal-body">
        <h3>This is the modal!</h3>
      </div>

      <div class="modal-footer">
        <button class="modal-default-button" @click="$emit('close')">OK</button>
      </div>
    </div>
  </div>
</template>
```

In the parent `App` component, we can render the modal component and a button that when clicked toggles the visibility of the modal component with the help of a reactive boolean value (`showModal`).

```html
<template>
  <button id="show-modal" @click="showModal = true">Show Modal</button>
  <Modal v-if="showModal" :show="showModal" @close="showModal = false" />
</template>

<script setup>
  import { ref } from "vue";
  import Modal from "./components/Modal.vue";

  const showModal = ref(false);
</script>
```

From this example, we can see that the modal component is shown only under a specific circumstance — when the user clicks the `Show Modal` button. Despite this, the JavaScript bundle associated with the component **is loaded automatically when the entire webpage is loaded** even before the modal is made visible.

This is fine for the majority of cases. However, under conditions where the bundle size of the modal is really large and/or the application has a multitude of such components, this can lead to a delayed initial load time. With every added bundle, even if it's related to components that are rarely used, the time it takes for the initial page to load grows.

### defineAsyncComponent

This is where Vue allows us to divide an app into smaller chunks by loading components asynchronously with the help of the [`defineAsyncComponent()`](https://vuejs.org/api/general.html#defineasynccomponent) function.

```js
import { defineAsyncComponent } from "vue";

const AsyncComp = defineAsyncComponent(() => {
  return new Promise((resolve, reject) => {
    // ...load component from the server
    resolve(/* loaded component */);
  });
});
```

The `defineAsyncComponent()` function accepts a loader function that returns a Promise that resolves to the imported component. However, instead of defining our async component function like the above, we can leverage [dynamic imports](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/import) to load an ECMAScript module asynchronously.

```js
import { defineAsyncComponent } from "vue";

export const AsyncComp = defineAsyncComponent(() =>
  import("./components/MyComponent.vue")
);
```

Let's see this in action for our modal example. We'll create a new file titled `AsyncModal.js`:

```js
import { defineAsyncComponent } from "vue";

export const AsyncModal = defineAsyncComponent(() => import("./Modal.vue"));
```

In our parent `App` component, we'll now import and use the `AsyncModal` asynchronous component in place of the `Modal` component.

```html
<template>
  <button id="show-modal" @click="showModal = true">Show Modal</button>
  <AsyncModal v-if="showModal" :show="showModal" @close="showModal = false" />
</template>

<script setup>
  import { ref } from "vue";
  import { AsyncModal } from "./components/AsyncModal";

  const showModal = ref(false);
</script>
```

With this small change, our modal component will now be asynchronously loaded! When our application webpage initially loads, the bundle for the `Modal` component _is no longer loaded automatically upon page load_. When we click the button to trigger the modal to be shown, the bundle is then asynchronously loaded as the modal component is being rendered.

### Loading and error UI

With `defineAsyncComponent()`, Vue provides developers with more than just a means of asynchronously loading components. It also offers capabilities to display feedback to users during the loading process and handle any potential errors.

#### loadingComponent

There may be times we may want to provide visual feedback to users while a component is being fetched. To achieve this, `defineAsyncComponent()` has a `loadingComponent` option that lets us specify a component to show during the loading phase.

```js
import { defineAsyncComponent } from "vue";
import Loading from "./Loading.vue";

export const AsyncModal = defineAsyncComponent({
  loader: () => import("./Modal.vue"),
  loadingComponent: Loading,
});
```

As the modal component becomes asynchronously loaded, the user will now be presented with a `Loading...` message.

#### errorComponent

In certain conditions (e.g. poor internet connections), there may be chances that the asynchronous component fails to load. The `defineAsyncComponent()` function offers the `errorComponent` option to handle such situations, allowing us to specify a component to be displayed when there's a loading error.

```js
import { defineAsyncComponent } from "vue";
import Loading from "./Loading.vue";
import Error from "./Error.vue";

export const AsyncModal = defineAsyncComponent({
  loader: () => import("./Modal.vue"),
  loadingComponent: Loading,
  errorComponent: Error,
});
```

When the modal component fails to load, the `Error` component template will be shown.

The `defineAsyncComponent()` function accepts further options like `delay`, `timeout`, `suspensible`, and `onError()` which provide developers with more granular control over the asynchronous loading behavior and user experience. Be sure to check out the [API documentation](https://vuejs.org/api/general.html#defineasynccomponent) for more details on these properties.

The `defineAsyncComponent()` function can help in breaking down the initial load of a Vue application into manageable chunks by deferring the loading of certain components until they're needed. This can help improve page load times and overall application performance especially when an application has numerous components that have a large bundle size.

## Source

- [patterns.dev/vue/async-components](https://patterns.dev/vue/async-components)

### References

- [Async Components | Vue Documentation](https://vuejs.org/guide/components/async.html#async-components)
