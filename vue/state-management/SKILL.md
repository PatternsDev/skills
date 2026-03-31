---
name: state-management
description: Manage application-level state between components using stores, Pinia, or the Composition API.
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

# State Management

**Vue components are the building blocks of Vue apps** by allowing us to couple markup (HTML), logic (JS), and styles (CSS) within them.

## When to Use

- Use this when you need to share reactive data between sibling or deeply nested components
- This is helpful for managing global application state beyond simple parent-child prop passing

## Instructions

- Use props for parent-to-child data flow and custom events for child-to-parent communication
- Create a simple store using `reactive()` for small applications
- Use Pinia (the official Vue state management library) for larger apps needing devtools, plugins, and TypeScript support
- Choose the state management approach based on your app's complexity — don't over-engineer small apps

## Details

Here's an example of a Single-File component that displays a series of numbers from a data property:

```html
<template>
  <div>
    <h2>The numbers are {{ numbers }}!</h2>
  </div>
</template>

<script setup>
  import { ref } from "vue";

  const numbers = ref([1, 2, 3]);
</script>
```

The `ref()` function prepares the component to be _reactive_. If a reactive property value that's being used in the template changes, the component view will re-render to show the change.

What if `numbers` was a data value that needed to be accessed from another component? If we want to share `numbers` between multiple components, `numbers` doesn't only become component-level data _but also_ application-level data. This brings us to the topic of **State Management** - the management of application level data.

### Props

Vue gives us the ability to use **props** to pass data from the parent down to the child. Using props is fairly simple. All we essentially need to do is bind a value to the prop attribute where the child component is being rendered.

**ParentComponent**:

```html
<template>
  <div>
    <ChildComponent :numbers="numbers" />
  </div>
</template>

<script setup>
  import { ref } from "vue";
  import ChildComponent from "./ChildComponent";

  const numbers = ref([1, 2, 3]);
</script>
```

**ChildComponent**:

```html
<template>
  <div>
    <h2>{{ numbers }}</h2>
  </div>
</template>

<script setup>
  const { numbers } = defineProps(["numbers"]);
</script>
```

### Component Events

What if we needed to find a way to communicate information in the opposite direction? We can't use `props` since `props` can only be used to pass data in a uni-directional format (from parent down to child). To facilitate having the child component notify the parent about something, we can use custom events.

Custom events in Vue are dispatched as native CustomEvents and are used for communication between components.

**ChildComponent**:

```html
<template>
  <div>
    <h2>{{ numbers }}</h2>
    <input v-model="number" type="number" />
    <button @click="$emit('number-added', Number(number))">
      Add new number
    </button>
  </div>
</template>

<script setup>
  const { numbers } = defineProps(["numbers"]);
</script>
```

**ParentComponent**:

```html
<template>
  <div>
    <ChildComponent :numbers="numbers" @number-added="(n) => numbers.push(n)" />
  </div>
</template>

<script setup>
  import { ref } from "vue";
  import ChildComponent from "./ChildComponent";

  const numbers = ref([1, 2, 3]);
</script>
```

### Simple State Management

We can use props to pass data downwards and custom events to send messages upwards. How would we be able to either pass data or facilitate communication between two different sibling components?

A simple way to manage application-level state is to create a store pattern that involves sharing a data store between components. The store can manage the state of our application as well as the methods that are responsible for changing the state.

```js
import { reactive } from "vue";

export const store = reactive({
  numbers: [1, 2, 3],
  addNumber(newNumber) {
    this.numbers.push(newNumber);
  },
});
```

The store contains a `numbers` array and an `addNumber` method that accepts a payload and directly updates the store's `numbers` value.

With Vue 3.x, we're able to import and use the `reactive()` function to declare reactive state from a JavaScript object. When this reactive state gets changed with the `addNumber()` method, any component that uses this reactive state will automatically update!

**NumberDisplay**:

```html
<template>
  <div>
    <h2>{{ store.numbers }}</h2>
  </div>
</template>

<script setup>
  import { store } from "../store.js";
</script>
```

**NumberSubmit**:

```html
<template>
  <div>
    <input v-model="numberInput" type="number" />
    <button @click="store.addNumber(numberInput)">Add new number</button>
  </div>
</template>

<script setup>
  import { ref } from "vue";
  import { store } from "../store.js";

  const numberInput = ref(0);
</script>
```

When we say components interact with one another here, we're using the term 'interact' loosely. The components aren't going to do anything to each other but instead invoke changes to one another _through_ the store.

If we take a closer look at all the pieces that directly interact with the store, we can establish a pattern:

- The method in `NumberSubmit` has the responsibility to directly act on the store method, so we can label it as a **store action**.
- The store method has a certain responsibility as well - to directly mutate the store state. So we'll say it's a **store mutation**.
- `NumberDisplay` doesn't really care about what type of methods exist in the store or in `NumberSubmit`, and is only concerned with getting information from the store. So we'll say `NumberDisplay` is a **store getter** of sorts.

An **action** commits to a **mutation**. The **mutation** mutates state which then affects the view/components. View/components retrieve store data with **getters**. We're starting to get closer to a more structured manner to handling application-level state.

### Pinia

[Pinia](https://pinia.vuejs.org/) is a state management pattern and library for Vue.js that provides a more structured and scalable way to handle application-level state.

Pinia is an alternative to other state management solutions like [Vuex](https://vuex.vuejs.org/) and is now the official state management library for Vue. It provides a simple and efficient way to create and manage stores, which encapsulate state, actions, and getters.

In Pinia, we can define a store using the `defineStore()` function. Here we're using the Composition API syntax to define a `useNumbersStore()` function to create a `numbers` store.

```js
import { ref } from "vue";
import { defineStore } from "pinia";

export const useNumbersStore = defineStore("numbers", () => {
  const numbers = ref([1, 2, 3]);

  function addNumber(newNumber) {
    this.numbers.push(newNumber);
  }

  return { numbers, addNumber };
});
```

We can then create a Pinia instance and install it in our Vue app.

```js
import { createApp } from "vue";
import { createPinia } from "pinia";
import App from "./App.vue";

const app = createApp(App);
const pinia = createPinia();

app.use(pinia);
app.mount("#app");
```

In the `NumberDisplay` component:

```html
<template>
  <div>
    <h2>{{ store.numbers }}</h2>
  </div>
</template>

<script setup>
  import { useNumbersStore } from "../store";

  const store = useNumbersStore();
</script>
```

In the `NumberSubmit` component:

```html
<template>
  <div>
    <input v-model="numberInput" type="number" />
    <button @click="store.addNumber(numberInput)">Add new number</button>
  </div>
</template>

<script setup>
  import { ref } from "vue";
  import { useNumbersStore } from "../store";

  const store = useNumbersStore();
  const numberInput = ref(0);
</script>
```

For such a simple implementation like this, a Pinia store may not really be necessary and behaves very similarly to just using a store created with the `reactive()` function. With that said, Pinia offers additional capabilities for more complex use-cases such as the ability to extend Pinia features with plugins, have devtools support, and have more appropriate TypeScript support and server-side rendering support.

### What's the correct way?

Each method for managing application-level state comes with its advantages and disadvantages.

#### Simple Store

- **Pro**: Relatively easy to establish.
- **Con**: State and possible state changes aren't explicitly defined.

#### Pinia

- **Pro**: Devtools support, plugins + typescript + server-side rendering support
- **Con**: Additional boilerplate.

At the end of the day, it's up to us to understand what's needed in our application and what the best approach may be.

### Helpful resources

- [Props | Vue Documentation](https://vuejs.org/guide/components/props.html#props)
- [Component Events | Vue Documentation](https://vuejs.org/guide/components/events.html#component-events)
- [Simple State Management with Reactivity API | Vue Documentation](https://vuejs.org/guide/scaling-up/state-management.html#simple-state-management-with-reactivity-api)
- [Core concepts | Pinia](https://pinia.vuejs.org/core-concepts/)

## Source

- [patterns.dev/vue/state-management](https://patterns.dev/vue/state-management)
