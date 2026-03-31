---
name: data-provider
description: Utilize renderless components for managing and providing data to child components via scoped slots.
paths:
  - "**/*.vue"
  - "**/composables/**/*.ts"
metadata:
  author: patterns.dev
  version: "1.0"
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
- Use the ask questions tool if you need to clarify requirements with the user

## Details

### Data Provider Pattern

The data provider pattern is a design pattern that complements the renderless component pattern in Vue by focusing on providing data and state management capabilities to components _without being concerned about how the data is rendered or displayed_.

In the data provider pattern, a data provider component encapsulates the logic for fetching, managing, and exposing data to its child components. The child components can then consume this data and use it in their own rendering or behavior.

This pattern promotes separation of concerns, as the data provider component takes care of data-related tasks, while the child components can focus on presentation and interaction.

Let's illustrate the data provider pattern with an example. Consider a simple application that displays the setup of a funny joke followed by its punchline. To help us show different jokes randomly, we'll use the free public API endpoint [https://official-joke-api.appspot.com/random_joke](https://official-joke-api.appspot.com/random_joke) that returns a random joke in JSON format.

```shell
# https://official-joke-api.appspot.com/random_joke

{
  "type": "general",
  "setup": "How good are you at Power Point?",
  "punchline": "I Excel at it.",
  "id": 129
}
```

We'll first create a data provider component called `DataProvider` that will hold the responsibility of fetching the joke from the API. In the `<script>` section of the component, we'll import the `ref()` and `reactive()` functions from the Vue library, assign the endpoint URL value to a constant, and set up `data` and `loading` reactive properties to capture the data and loading status of our API request.

```html
<script setup>
  import { ref, reactive } from "vue";

  const API_ENDPOINT_URL = "https://official-joke-api.appspot.com/random_joke";

  const data = reactive({
    setup: null,
    punchline: null,
  });

  const loading = ref(false);
</script>
```

We can then create a `fetchJoke()` function in our `DataProvider` component to handle the API call.

```js
const fetchJoke = async () => {
  loading.value = true;
  try {
    const response = await fetch(API_ENDPOINT_URL);
    const jokeData = await response.json();
    data.setup = jokeData.setup;
    data.punchline = jokeData.punchline;
  } catch (error) {
    console.error("Error fetching joke:", error);
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

## Helpful Resources

- [Scoped Slots | Vue Documentation](https://vuejs.org/guide/components/slots.html#scoped-slots)
- [Renderless Components | Vue Documentation](https://vuejs.org/guide/components/slots.html#renderless-components)
