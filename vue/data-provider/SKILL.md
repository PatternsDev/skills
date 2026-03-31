---
name: data-provider
description: Utilize renderless components for managing and providing data to child components via scoped slots.
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

# Data Provider Pattern

In a previous article, we've come to learn how renderless components help separate the logic of a component from its presentation. This becomes useful when we need to create reusable logic that can be applied to different UI implementations.

Renderless components also allow us to leverage another helpful pattern known as the **data provider pattern**.

## When to Use

- Use this when multiple components need to consume the same data but display it differently
- This is helpful for centralizing data-fetching logic without coupling it to specific UI components

## Instructions

- Create a data provider component whose template is a single `<slot>` with scoped slot props
- Pass data, loading state, and action methods as scoped slot props to child components
- Use `v-slot` destructuring in the parent to access provided data
- Keep child components focused purely on presentation; the data provider handles all data logic

## Details

### Data Provider Pattern

The data provider pattern is a design pattern that complements the renderless component pattern in Vue by focusing on providing data and state management capabilities to components _without being concerned about how the data is rendered or displayed_.

In the data provider pattern, a data provider component encapsulates the logic for fetching, managing, and exposing data to its child components. The child components can then consume this data and use it in their own rendering or behavior.

This pattern promotes separation of concerns, as the data provider component takes care of data-related tasks, while the child components can focus on presentation and interaction.

Let's illustrate the data provider pattern with an example. Consider a simple application that displays the setup of a funny joke followed by its punchline. To keep the example self-contained, we'll use a local in-memory data source instead of depending on an external API.

```js
const jokes = [
  { id: 1, setup: "Why did the dev go broke?", punchline: "Because they used up all their cache." },
  { id: 2, setup: "Why do functions love TypeScript?", punchline: "Because it keeps their arguments in order." },
];
```

We'll first create a data provider component called `DataProvider` that will hold the responsibility of loading a joke. In the `<script>` section of the component, we'll import the `ref()` and `reactive()` functions from Vue, define a local data source, and set up `data` and `loading` reactive properties to capture the selected joke and loading state.

```html
<script setup>
  import { ref, reactive } from "vue";

  const jokes = [
    { id: 1, setup: "Why did the dev go broke?", punchline: "Because they used up all their cache." },
    { id: 2, setup: "Why do functions love TypeScript?", punchline: "Because it keeps their arguments in order." },
  ];

  const data = reactive({
    setup: null,
    punchline: null,
  });

  const loading = ref(false);
</script>
```

We can then create a `fetchJoke()` function in our `DataProvider` component to simulate loading data asynchronously.

```js
const fetchJoke = async () => {
  loading.value = true;
  try {
    await new Promise((resolve) => setTimeout(resolve, 300));
    const jokeData = jokes[Math.floor(Math.random() * jokes.length)];
    data.setup = jokeData.setup;
    data.punchline = jokeData.punchline;
  } catch (error) {
    console.error("Error loading joke:", error);
  } finally {
    loading.value = false;
  }
};
```

With the fetch function ready, we can call it when the component mounts using the `onMounted()` lifecycle hook.

```js
import { ref, reactive, onMounted } from "vue";

// ...

onMounted(() => {
  fetchJoke();
});
```

The **key** element in a data provider component is that its template consists purely of a single `<slot>` element. This slot will provide the fetched data and the relevant method to its child components using **scoped slots**.

```html
<template>
  <slot :data="data" :loading="loading" :fetchJoke="fetchJoke"></slot>
</template>
```

The `DataProvider` component passes `data`, `loading`, and `fetchJoke` as scoped slot props. This means any child component placed inside the `DataProvider` can access these properties.

Now, let's create a `JokeCard` component that will present the joke data.

```html
<template>
  <div class="joke-card">
    <p v-if="loading">Loading...</p>
    <div v-else>
      <p class="setup">{{ data.setup }}</p>
      <p class="punchline">{{ data.punchline }}</p>
    </div>
    <button @click="fetchJoke">Get Another Joke</button>
  </div>
</template>

<script setup>
  defineProps(["data", "loading", "fetchJoke"]);
</script>
```

The `JokeCard` component is a simple presentational component. It expects `data`, `loading`, and `fetchJoke` as props, and renders the joke data along with a button to fetch a new joke.

Now, to bring it all together, we use the `DataProvider` component in our `App` component. We wrap the `JokeCard` component inside the `DataProvider` and pass the scoped slot props to it:

```html
<template>
  <DataProvider v-slot="{ data, loading, fetchJoke }">
    <JokeCard :data="data" :loading="loading" :fetchJoke="fetchJoke" />
  </DataProvider>
</template>

<script setup>
  import DataProvider from "./components/DataProvider.vue";
  import JokeCard from "./components/JokeCard.vue";
</script>
```

With this setup, the `DataProvider` handles all data fetching and management, while the `JokeCard` focuses solely on displaying the data. This clean separation makes it easy to swap out the presentational component for a different one without touching the data-fetching logic.

The data provider pattern is especially useful when:

- Multiple components need to consume the same data but display it differently.
- You want to centralize data fetching logic without tightly coupling it to specific UI components.
- You want to keep your components focused on a single responsibility.

## Source

- [patterns.dev/vue/data-provider](https://patterns.dev/vue/data-provider)

### References

- [Scoped Slots | Vue Documentation](https://vuejs.org/guide/components/slots.html#scoped-slots)
- [Renderless Components | Vue Documentation](https://vuejs.org/guide/components/slots.html#renderless-components)
