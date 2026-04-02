---
name: dynamic-components
description: Dynamically switch between components using Vue's special `<component>` element with the `is` attribute.
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

# Dynamic Components

Dynamic components constitute the ability to dynamically change (i.e. switch) between components by binding an `is` attribute to the reserved `<component>` element.

We'll go through an example to best understand how dynamic components work. Assume we have separate components titled `Home`, `Feed`, and `History` that simply display text dictating what component it is.

## When to Use

- Use this when you need to render different components based on user interaction (e.g., tabs, views)
- This is helpful as an alternative to multiple `v-if`/`v-else` blocks for conditional component rendering

## Instructions

- Bind the `is` attribute on `<component>` to a reactive value referencing the component definition
- Use a `tabs` object mapping names to imported component definitions
- Wrap `<component>` with `<KeepAlive>` to preserve state when switching between dynamic components

## Details

```html
<!-- Home -->
<template><div class="tab">Home component</div></template>

<!-- Feed -->
<template><div class="tab">Feed component</div></template>

<!-- History -->
<template><div class="tab">History component</div></template>
```

Our goal is to build an interface that surfaces a list of tabs that can be clicked. Depending on what tab is clicked, we want to dynamically render a certain component.

When clicking between the tabs, we want components to be dynamically unmounted and mounted without the use of routing. Though something like this could be achieved by conditionally rendering child templates with the help of directives like [`v-if` and `v-else`](https://vuejs.org/guide/essentials/conditional.html), this is a perfect use case of Vue dynamic components.

In the parent `App` component of our app, we can first import the three individual components to have them available in the template. We'll also create a `currentTab` reactive property that is given an initial value of `"Home"`.

```html
<script setup>
  import { ref } from "vue";
  import Home from "./components/Home.vue";
  import Feed from "./components/Feed.vue";
  import History from "./components/History.vue";

  const currentTab = ref("Home");
  const tabs = {
    Home,
    Feed,
    History,
  };
</script>
```

Note that our `tabs` object references the actual component definitions and not just the component names.

In the `App` component template, we'll look to render three separate tab buttons — one for each component we intend to display. We'll use the [`v-for` directive](https://vuejs.org/api/built-in-directives.html#v-for) to help achieve this.

```html
<template>
  <div class="demo">
    <button
      v-for="(_, tab) in tabs"
      :key="tab"
      :class="['tab-button', { active: currentTab === tab }]"
      @click="currentTab = tab"
    >
      {{ tab }}
    </button>
  </div>
</template>
```

To dynamically render a certain child component, we'll bind an `is` attribute to the reserved `<component>` element. The value attached to the `is` attribute should correspond to the child component that we want to render dynamically.

```html
<template>
  <div class="demo">
    <button
      v-for="(_, tab) in tabs"
      :key="tab"
      :class="['tab-button', { active: currentTab === tab }]"
      @click="currentTab = tab"
    >
      {{ tab }}
    </button>
    <component :is="tabs[currentTab]" class="tab"></component>
  </div>
</template>

<script setup>
  import { ref } from "vue";
  import Home from "./components/Home.vue";
  import Feed from "./components/Feed.vue";
  import History from "./components/History.vue";

  const currentTab = ref("Home");

  const tabs = {
    Home,
    Feed,
    History,
  };
</script>
```

With the dynamic `<component />` element placed in our template, the child components are now dynamically unmounted and mounted depending on which tab has been selected.

### Preserving state

Preserving state can be an important consideration to keep in mind when using dynamic components. By default, when a component is unmounted, its state is lost. However, Vue provides a way to preserve the state of dynamic components using the `<KeepAlive>` component.

To preserve the state of dynamic components, we can wrap the `<component>` element with the `<KeepAlive>` component.

```html
<template>
  <div class="demo">
    <!--  -->
    <KeepAlive>
      <component :is="tabs[currentTab]" class="tab"></component>
    </KeepAlive>
  </div>
</template>

<script setup>
  // ...
</script>
```

With the `<KeepAlive>` component wrapping the `<component>` element, the state of the dynamic components will be preserved when they are unmounted. This means that any data or component state will be maintained, and the component will retain its previous state when it is mounted again.

To see an example of this, we can update each of our child components to contain a simple counter that increments.

```html
<!-- Repeat this counter example for Home, Feed, and History -->
<template>
  <div class="tab">
    Home component
    <p>Counter: {{ counter }}</p>
    <button @click="incrementCounter">Increment</button>
  </div>
</template>

<script setup>
  import { ref } from "vue";

  const counter = ref(0);

  const incrementCounter = () => {
    counter.value++;
  };
</script>
```

With these changes, the counter state for each respective child component is kept preserved even as we dynamically switch between components.

By using the `<KeepAlive>` component, we can enhance the behavior of dynamic components by preserving their state and providing a smoother user experience when switching between tabs.

## Source

- [patterns.dev/vue/dynamic-components](https://patterns.dev/vue/dynamic-components)

### References

- [Dynamic Components | Vue Documentation](https://vuejs.org/guide/essentials/component-basics.html#dynamic-components)
- [KeepAlive | Vue Documentation](https://vuejs.org/guide/built-ins/keep-alive.html)
