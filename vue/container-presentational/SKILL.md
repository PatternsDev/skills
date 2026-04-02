---
name: container-presentational
description: Enforce separation of concerns by separating the view from the application logic in Vue components.
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

# Container/Presentational Pattern

In 2015, Dan Abramov wrote an article titled ["Presentational and Container Components"](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0) that changed the way many developers thought about component architecture in React. He introduced a pattern that separated components into two categories:

1. **Presentational Components (or Dumb Components)**: These are concerned with how things look. They don't specify how the data is loaded or mutated but rather receive data and callbacks exclusively via props.
2. **Container Components (or Smart Components)**: These are concerned with how things work. They provide the data and behavior to presentational or other container components.

## When to Use

- Use this when you want a clear separation between data-fetching logic and UI rendering
- This is helpful for making presentational components reusable and easy to test

## Instructions

- Container components handle data fetching and state; presentational components handle rendering via props
- Prefer composables over container components in Vue 3 for the same separation of concerns
- Keep presentational components stateless — they receive data only through props
- Use the `useDogImages()` composable pattern as a modern alternative to container wrappers

## Details

While this pattern was mainly associated with React, its fundamental principle was adopted and adapted in various forms across other libraries and frameworks.

Dan's distinction offered a clearer and more scalable way to structure JavaScript applications. By clearly defining the responsibilities of different types of components, developers could ensure better reusability of the UI components (presentational) and logic (containers).

However, with the emergence of hooks in React and the Composition API in Vue 3, the clear boundary between presentational and container components began to blur. Hooks and the Composition API began allowing developers to encapsulate and reuse state and logic without necessarily being confined to a class-based container component or the Options API. With that being said, the pattern can still be helpful at certain times.

Let's say we want to create an application that fetches 6 dog images, and renders these images on the screen.

To follow the container/presentational pattern, we want to enforce the separation of concerns by separating this process into two parts:

1. **Presentational Components**: Components that care about _**how**_ data is shown to the user. In this example, that's the rendering of the list of dog images.
2. **Container Components**: Components that care about _**what**_ data is shown to the user. In this example, that's fetching the dog images.

Fetching the dog images deals with **application logic**, whereas displaying the images only deals with the **view**.

### Presentational Component

A presentational component receives its data through `props`. Its primary function is to simply **display the data it receives** the way we want them to, including styles, _without modifying_ that data.

When rendering the dog images, we simply want to map over each dog image that was fetched from the API, and render those images. We can create a `DogImages` component that receives the data through `props`, and renders the data it received.

```html
<template>
  <div>
    <div v-for="(dog, index) in dogs" :key="index">
      <img :src="dog" alt="Dog" />
    </div>
  </div>
</template>

<script setup>
  defineProps(["dogs"]);
</script>
```

The `DogImages` component is a presentational component. Presentational components are _usually_ stateless: they do not contain their own Vue state, unless they need a state for UI purposes. Presentational components receive their data from **container components**.


### Container Component

The primary function of container components is to **pass data** to presentational components, which they _contain_. Container components themselves usually don't render any other components besides the presentational components that care about their data. Since they don't render anything themselves, they usually do not contain any styling either.

We need to create a **container component** that fetches this data, and passes this data to the presentational component `DogImages` in order to display it on the screen.

```html
<template>
  <DogImages :dogs="dogs" />
</template>

<script setup>
  import { ref, onMounted } from "vue";
  import DogImages from "./DogImages.vue";

  const dogs = ref([]);

  onMounted(async () => {
    const response = await fetch(
      "https://dog.ceo/api/breed/labrador/images/random/6"
    );
    const { message } = await response.json();
    dogs.value = message;
  });
</script>
```

Combining these two components together makes it possible to separate handling application logic with the view.

### Composables

In many cases, the Container/Presentational pattern can be replaced with composables. The introduction of the Composition API made it easy for developers to add statefulness without needing a container component to provide that state.

Instead of having the data fetching logic in a container component, we can create a custom composable that fetches the images, and returns the array of dogs.

```js
import { ref, onMounted } from "vue";

export function useDogImages() {
  const dogs = ref([]);

  onMounted(async () => {
    const response = await fetch(
      "https://dog.ceo/api/breed/labrador/images/random/6"
    );
    const { message } = await response.json();
    dogs.value = message;
  });

  return { dogs };
}
```

By using this composable, we no longer need the wrapping container component to fetch the data. Instead, we can use this composable directly in our presentational `DogImages` component!

```html
<template>
  <div>
    <div v-for="(dog, index) in dogs" :key="index">
      <img :src="dog" alt="Dog" />
    </div>
  </div>
</template>

<script setup>
  import { useDogImages } from "../composables/useDogImages";

  const { dogs } = useDogImages();
</script>
```

By using the `useDogImages` composable, we still separated the application logic from the view. We're simply using the returned data from the composable, without modifying that data within the component.

Composables make it easy to separate logic and view in a component, just like the Container/Presentational pattern. It saves us the extra layer that was necessary in order to wrap the presentational component within the container component.

## Source

- [patterns.dev/vue/container-presentational](https://patterns.dev/vue/container-presentational)

### References

- [Presentational and Container Components - Dan Abramov](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)
- [Composables | Vue Documentation](https://vuejs.org/guide/reusability/composables.html)
