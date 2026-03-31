---
name: provide-inject
description: Have nested components access data without prop drilling using Vue's provide/inject mechanism.
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

# Provide/Inject

When managing data between parent and child components, Vue gives us the ability to use something known as **props** to pass data down from parent to child. Props can only flow in one direction, from parent components to child components (and further down). When state changes occur on parent elements, Vue will re-render components that depend on those values.

Using props works well in most cases. However, when working in large applications with a large number of components in the component tree, props can become hard to maintain since props need to be declared in _each and every component_ in the component tree.

## When to Use

- Use this when you need to pass data through deeply nested component trees without prop drilling
- This is helpful for application-wide data like themes, locale, or authentication state

## Instructions

- Use `provide()` in a parent or ancestor component to make data available to all descendants
- Use `inject()` in child components to access provided data
- Use app-level `app.provide()` for data needed across the entire application (e.g., plugins)
- Prefer props for data isolated to a specific set of components; use provide/inject for cross-cutting concerns
- Be aware that debugging can be harder with provide/inject in large apps with many providers

## Details

When considering how data can be managed between a large number of components, it's often best to work towards a solution that allows the management of application-level state in a maintainable and manageable manner (e.g. creating a reusable store, using Pinia, etc.). We talk about this in more detail in the [State Management](/vue/state-management) guide.

However, Vue also provides a certain pattern to help avoid the need for complex prop drilling in a Vue application known as the provide/inject pattern.

### Provide/Inject

The `provide()` function in Vue allows us to pass data through a component tree without the need to _prop-drill_ (i.e., pass props down manually at every level). On the other hand, the `inject()` option is used in child components to access the provided data or methods from their parent or any ancestor component.

We'll go through a simple example to illustrate how this can be done. Suppose we have a parent component called `App` that wants to share a piece of data with its child component, `ChildComponent`. Instead of passing this data as a prop, we can use `provide()` in the parent component to make the data available to all its child components.

```html
<template>
  <div id="app">
    <ChildComponent />
  </div>
</template>

<script setup>
  import { provide } from "vue";
  import ChildComponent from "./components/ChildComponent";

  provide("data", "Data from parent!");
</script>
```

We can then access this provided data in the `ChildComponent` with the help of the `inject()` function.

```html
<template>
  <div>
    <p>{{ data }}</p>
  </div>
</template>

<script setup>
  import { inject } from "vue";

  const data = inject("data");
</script>
```

By specifying `inject("data")` in the child component (`ChildComponent`), we directly access the provided `data` value from the parent component. We then bind `data` to the template to display its value.

With provide/inject, we would notice the same behavior even if we had numerous child components within the component hierarchy tree. Data from the parent `<App />` component will be rendered in any deeply nested child component without the need to prop-drill data through every component in the tree, thanks to provide/inject!

In addition to being able to `provide()` data from a parent component, we can lift the `provide()` up to the app level as well (i.e. where we instantiate our Vue application).

```js
import { createApp } from "vue";
import App from "./App.vue";
import "./styles.css";

const app = createApp(App);

// app-level provide
app.provide("data", "Data from parent!");

app.mount("#app");
```

Since app-level provides make data available to _all_ components, they are often helpful when creating [plugins](https://vuejs.org/guide/reusability/plugins.html) — self-contained code that adds functionality to the entire Vue app.

### Props vs. provide/inject

When do we choose between props and the provide/inject pattern? Both approaches have their advantages and disadvantages.

#### With props:

- We follow a clear pattern of passing data incrementally from one level to another (advantage).
- However, if our component hierarchy tree contains a large number of components, the process of passing props data one level at a time can become cumbersome (disadvantage).

#### With provide/inject

- Child components can directly access data from parent components located multiple levels above, eliminating the need for passing down data at each level (advantage).
- However, when bugs arise, debugging can be more challenging with provide/inject. This challenge becomes more pronounced in large-scale applications with numerous different providers (disadvantage).

The provide/inject pattern is most suitable for application-wide client data, such as theme information, locale/language preferences, and user authentication details. These types of data are better managed with provide/inject since any component within the application may require access to them at any given time.

On the other hand, props are ideal when data needs to be isolated within a specific set of components only.

### Helpful resources

- [Provide / Inject | Vue Documentation](https://vuejs.org/guide/components/provide-inject.html)

## Source

- [patterns.dev/vue/provide-inject](https://patterns.dev/vue/provide-inject)
