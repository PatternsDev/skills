---
name: renderless-components
description: Components that don't render their own markup, providing logic to children via scoped slots.
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

# Renderless Components

Renderless components are a pattern in Vue that **separates the logic of a component from its presentation**. The pattern provides a way to encapsulate functionality without _dictating the visual representation of the component_. In other words, a renderless component focuses solely on the logic and behavior, while leaving the rendering to the parent component.

Renderless components are particularly useful when we need to create reusable logic that can be applied to different UI implementations. By abstracting the logic into a renderless component, we can easily reuse it in various contexts without duplicating code.

## When to Use

- Use this when you need to reuse logic across components with completely different visual representations
- This is helpful for providing a component-based API in a component library

## Instructions

- Create a component that provides data and methods through a single `<slot>` with scoped slot props
- Use `v-slot` destructuring in parent components to access the provided data and methods
- Prefer composables over renderless components in Vue 3 to avoid extra component nesting
- Use renderless components when you want template-level composition or a component-based API

## Details

### Toggle, toggle, toggle

Imagine you have a toggle UI element that needs to be used in different parts of your application, but each instance may have a different visual representation. Some toggles might be displayed as buttons, while others might be checkboxes or switches.

We could just create three different toggle components, however, we can observe that each toggle element has the same logic and behavior. Each toggle has an inactive and active state that's being tracked with a component data property (e.g. `checked`). When a toggle is clicked, its component state is switched from inactive to active and vice versa (i.e. `checked = !checked`).

Right away, we can see that we can create a more reusable pattern by extracting the common logic and behavior in such a way that we don't have to repeatedly define the state and toggle methods in each individual toggle component. This is a great case to use [composables](/vue/composables) since composables will allow us to encapsulate and share the common stateful logic across the different toggle components.

**useCheckboxToggle**:

```js
import { ref } from "vue";

export function useCheckboxToggle() {
  const checkbox = ref(false);

  const toggleCheckbox = () => {
    checkbox.value = !checkbox.value;
  };

  return {
    checkbox,
    toggleCheckbox,
  };
}
```

With this composable, we can now use the `useCheckboxToggle()` function in our various toggle components to share the common state and toggle logic.

However, there's another approach we can take that leverages Vue's slot mechanism — the **renderless component** pattern.

### The Renderless Component

A renderless component in Vue is a component that encapsulates logic and provides data to its children via **scoped slots**, without rendering any markup of its own. The parent component decides how the data is presented.

Here's a simple renderless `Toggle` component:

```html
<script setup>
  import { ref } from "vue";

  const checked = ref(false);

  const toggle = () => {
    checked.value = !checked.value;
  };
</script>

<template>
  <slot :checked="checked" :toggle="toggle"></slot>
</template>
```

The `Toggle` component doesn't render any HTML of its own. It only provides data (`checked` and `toggle`) through a scoped slot. The parent component can now consume this data and render whatever UI it wants.

**Using the renderless Toggle as a button:**

```html
<template>
  <Toggle v-slot="{ checked, toggle }">
    <button @click="toggle">
      {{ checked ? "ON" : "OFF" }}
    </button>
  </Toggle>
</template>
```

**Using the renderless Toggle as a checkbox:**

```html
<template>
  <Toggle v-slot="{ checked, toggle }">
    <label>
      <input type="checkbox" :checked="checked" @change="toggle" />
      {{ checked ? "Checked" : "Unchecked" }}
    </label>
  </Toggle>
</template>
```

**Using the renderless Toggle as a switch:**

```html
<template>
  <Toggle v-slot="{ checked, toggle }">
    <div
      class="switch"
      :class="{ active: checked }"
      @click="toggle"
    >
      <div class="switch-handle"></div>
    </div>
  </Toggle>
</template>
```

In all three cases, the same `Toggle` renderless component provides the toggle logic, but the rendering is entirely different!

### Composables vs. Renderless Components

Both composables and renderless components achieve the goal of reusing logic across components. However, there are some differences:

**Composables:**
- Logic is encapsulated in a regular JavaScript function.
- Can be used directly in `<script setup>` or `setup()`.
- Don't involve any additional component layers.

**Renderless components:**
- Logic is encapsulated in a Vue component.
- Use scoped slots to pass data to children.
- Add an extra component layer in the template.

In general, composables are the preferred approach in Vue 3 since they don't add extra component nesting. However, renderless components can be useful when you want to provide a component-based API (e.g., in a component library) or when you need template-level composition.

## Source

- [patterns.dev/vue/renderless-components](https://patterns.dev/vue/renderless-components)

### References

- [Renderless Components | Vue Documentation](https://vuejs.org/guide/components/slots.html#renderless-components)
- [Composables | Vue Documentation](https://vuejs.org/guide/reusability/composables.html)
