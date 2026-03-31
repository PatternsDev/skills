---
name: render-functions
description: Create component templates programmatically with JavaScript using Vue's `h()` render function or JSX.
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

# Render Functions

Vue recommends for us to use templates (i.e. the `<template></template>` syntax) to construct the markup of our Vue components. However, we're also given the opportunity to directly use something known as **render functions** to build the markup of our components as well.

Vue, at build time, takes the templates we create for our components and compiles them to render functions. It's at these compiled render functions, where Vue builds a virtual representation of nodes that make up the virtual DOM.

## When to Use

- Use this when you need complex dynamic rendering logic that's hard to express with template directives
- This is helpful for component library development where flexibility and low-level control are needed

## Instructions

- Use the `h()` function with three arguments: tag/component, props/attributes, and children
- Use JSX with `@vue/babel-plugin-jsx` as a more readable alternative to raw `h()` calls
- Prefer Vue templates for most application code — render functions are for advanced cases
- Remember that Vue JSX uses `class` (not `className`) and single curly braces `{}`

## Details

> If you're interested, the [Rendering Mechanism](https://vuejs.org/guide/extras/rendering-mechanism.html) section of the Vue documentation goes into more detail on the concept of the virtual DOM and Vue's internal rendering mechanism.

By using render functions, we skip the compile step that Vue takes to compile our templates, and are able to construct our component templates with the help of programmatic JavaScript.

### But why?

Render functions come into play when we require a higher level of customization and flexibility that's not easily achievable with the standard template syntax. In a nutshell, you may prefer to use render functions:

- When you need to dynamically render components or elements based on complex logic that can be cumbersome to express within a template.
- You want to have a direct hand on the Virtual DOM for advanced manipulations.
- You want to use JSX for building the template of your components.

Outside of these unique cases, Vue's template syntax should remain the go-to method for constructing component markup.

### Render functions

Assume we had the following component that contains a `<div>` element encompassing a `<header>` element. The text content of the `<header>` element simply displays the value of a `message` prop.

```html
<template>
  <div class="render-card">
    <header class="card-header card-header-title">{{ message }}</header>
  </div>
</template>

<script setup>
  const { message } = defineProps(["message"]);
</script>
```

We'll recreate the markup of the component step by step with the help of the render function — i.e. the `h()` function.

`h` is short for **hyperscript** which is a term often used in virtual DOM implementations to denote JavaScript syntax that produces HTML. In simple terms, the `h()` function is the render function that allows us to create the "virtual" representation of the DOM nodes that Vue uses to track and subsequently render on the page.

The `h()` function takes three arguments of its own:

1. An HTML tag name or a component definition.
2. The props/attributes to be passed onto the element (event listeners, class attributes, etc.).
3. Child nodes of the parent node.

Here's the full render function equivalent:

```html
<template>
  <render />
</template>

<script setup>
  import { h } from "vue";

  const { message } = defineProps(["message"]);

  const render = () => {
    return h(
      "div",
      {
        class: "render-card",
      },
      [
        h(
          "header",
          {
            class: "card-header card-header-title",
          },
          message
        ),
      ]
    );
  };
</script>
```

We can now render the above component in the parent `App.vue` instance and pass a value of `"Hello World!"` to the `message` prop.

```html
<template>
  <RenderComponent message="Hello world!" />
</template>

<script setup>
  import RenderComponent from "./components/RenderComponent.vue";
</script>
```

The component constructed with a render function produces the exact same output to its template equivalent.

### JSX

JSX is a syntax extension that allows us to write HTML-like code within JavaScript. With Vue, JSX can be used as an alternative to the `h()` function to construct render functions.

Vue's JSX support isn't built in like in React. We need to use a specific Babel plugin — [`@vue/babel-plugin-jsx`](https://github.com/vuejs/babel-plugin-jsx) — to have our JSX code transformed into the appropriate `h()` function calls.

Here's the same render function component we've built before but now recreated with JSX:

```jsx
<script setup>
  const { message } = defineProps(["message"]);

  const render = () => {
    return (
      <div class="render-card">
        <header class="card-header card-header-title">{message}</header>
      </div>
    );
  };
</script>
```

Since JSX is closer to JavaScript than to HTML, Vue JSX components use `class` instead of `className` and variables are embedded with single curly braces `{}` instead of double curly braces `{{ }}`.

### When to use render functions

- If your component has complex, conditional rendering logic that is hard to express with template directives, render functions (with or without JSX) can provide a cleaner solution.
- If you want more direct control over the virtual DOM.
- In library/component-kit development where flexibility and low-level control are needed.

For most typical application development, Vue templates offer the right level of expressiveness and readability. Render functions and JSX are powerful tools to reach for when templates aren't enough.

## Source

- [patterns.dev/vue/render-functions](https://patterns.dev/vue/render-functions)

### References

- [Render Functions & JSX | Vue Documentation](https://vuejs.org/guide/extras/render-function.html)
- [`@vue/babel-plugin-jsx` | GitHub](https://github.com/vuejs/babel-plugin-jsx)
