---
name: module-pattern
description: Split your code into smaller, reusable pieces with proper encapsulation using ES2015 modules.
license: MIT
metadata:
  author: patterns.dev
  version: "1.1"
paths:
  - "**/*.js"
  - "**/*.ts"
related_skills:
  - "singleton-pattern"
  - "observer-pattern"
---

# Module Pattern

As your application and codebase grow, it becomes increasingly important to keep your code maintainable and separated. The module pattern allows you to split up your code into smaller, reusable pieces.

Besides being able to split your code into smaller reusable pieces, modules allow you to keep certain values within your file _private_. Declarations within a module are scoped (_encapsulated_) to that module, by default. If we don't explicitly export a certain value, that value is not available outside that module. This reduces the risk of name collisions for values declared in other parts of your codebase, since the values are not available on the global scope.

## When to Use

- Use this when you need to organize code into maintainable, encapsulated units
- This is helpful when you want to keep certain values private to a module and avoid global scope pollution
- Use this to enable tree-shaking and reduce bundle sizes

## Instructions

- Use ES2015 `import`/`export` syntax for module definitions
- Use named exports for multiple values and default exports for the primary value of a module
- Keep non-exported values private to reduce naming collision risks
- Use dynamic `import()` for on-demand module loading to reduce initial bundle size

## Details

### ES2015 Modules

ES2015 introduced built-in JavaScript modules. A module is a file containing JavaScript code, with some difference in behavior compared to a normal script.

Let's look at an example of a module called `math.js`, containing mathematical functions.

```js
export function add(x, y) {
  return x + y;
}

export function multiply(x) {
  return x * 2;
}

export function subtract(x, y) {
  return x - y;
}

export function square(x) {
  return x * x;
}
```

We have a `math.js` file containing some simple mathematical logic. We have functions that allow users to add, multiply, subtract, and get the square of values that they pass.

In order to make the functions from `math.js` available to other files, we first have to _export_ them. In order to export code from a module, we can use the `export` keyword. One way of exporting the functions, is by using _named exports_: we can simply add the `export` keyword in front of the parts that we want to publicly expose.

We can then import the values in another file using the `import` keyword. To let JavaScript know from which module we want to import these functions, we need to add a `from` value and the relative path to the module.

```js
import { add, multiply, subtract, square } from "./math.js";
```

A great benefit of having modules, is that we _only have access to the values that we explicitly exported_ using the `export` keyword. Values that we didn't explicitly export using the `export` keyword, are only available within that module.

Let's create a value that should only be referenceable within the `math.js` file, called `privateValue`.

```js
const privateValue = "This is a value private to the module!";

export function add(x, y) {
  return x + y;
}

export function multiply(x) {
  return x * 2;
}

export function subtract(x, y) {
  return x - y;
}

export function square(x) {
  return x * x;
}
```

Notice how we didn't add the `export` keyword in front of `privateValue`. Since we didn't export the `privateValue` variable, we don't have access to this value outside of the `math.js` module!

By keeping the value private to the module, there is a reduced risk of accidentally polluting the global scope. You don't have to fear that you will accidentally overwrite values created by developers using your module, that may have had the same name as your private value: it prevents naming collisions.


Sometimes, the names of the exports could collide with local values. In this case, we can _rename_ the imported values, by using the `as` keyword.

```js
import {
  add as addValues,
  multiply as multiplyValues,
  subtract,
  square,
} from "./math.js";

function add(...args) {
  return args.reduce((acc, cur) => cur + acc);
}

function multiply(...args) {
  return args.reduce((acc, cur) => cur * acc);
}

/* From math.js module */
addValues(7, 8);
multiplyValues(8, 9);
subtract(10, 3);
square(3);

/* From index.js file */
add(8, 9, 2, 10);
multiply(8, 9, 2, 10);
```

Besides named exports, you can also use a _default export_. You can only have **one** default export per module.

```js
export default function add(x, y) {
  return x + y;
}

export function multiply(x) {
  return x * 2;
}

export function subtract(x, y) {
  return x - y;
}

export function square(x) {
  return x * x;
}
```

The difference between named exports and default exports, is the way the value is exported from the module, effectively changing the way we have to import the value.

Previously, we had to use the brackets for our named exports: `import { module } from 'module'`.
With a default export, we can import the value _without_ the brackets: `import module from 'module'`.

```js
import add, { multiply, subtract, square } from "./math.js";

add(7, 8);
multiply(8, 9);
subtract(10, 3);
square(3);
```

Since JavaScript knows that this value is always the value that was exported by default, we can give the imported default value another name than the name we exported it with.

We can also import all exports from a module, meaning all named exports _and_ the default export, by using an asterisk `*` and giving the name we want to import the module as.

```js
import * as math from "./math.js";

math.default(7, 8);
math.multiply(8, 9);
math.subtract(10, 3);
math.square(3);
```

In this case, we're importing _all_ exports from a module. Be careful when doing this, since you may end up unnecessarily importing values.

Using the `*` only imports all exported values. Values private to the module are still not available in the file that imports the module, unless you explicitly exported them.

### React

When building applications with React, you often have to deal with a large amount of components. Instead of writing all of these components in one file, we can separate the components in their own files, essentially creating a module for each component.

We can split components into separate files:

- `TodoList.js` for the `List` component
- `Button.js` for the customized `Button` component
- `Input.js` for the customized `Input` component

Throughout the app, we don't want to use the default `Button` and `Input` component, imported from a UI library. Instead, we want to use our custom version of the components, by adding custom styles to it defined in the `styles` object in their files. Rather than importing the default `Button` and `Input` component each time in our application and adding custom styles to it over and over, we can now simply import the default `Button` and `Input` component once, add styles, and export our custom component.

Notice how we can have an object called `style` in both `Button.js` and `Input.js`. Since this value is _module-scoped_, we can reuse the variable name without risking a name collision.

### Dynamic import

When importing all modules on the top of a file, all modules get loaded before the rest of the file. In some cases, we only need to import a module based on a certain condition. With a **dynamic import**, we can import modules on demand.

```js
import("module").then((module) => {
  module.default();
  module.namedExport();
});

// Or with async/await
(async () => {
  const module = await import("module");
  module.default();
  module.namedExport();
})();
```

By dynamically importing modules, we can reduce the page load time. We only have to load, parse, and compile the code that the user really needs, _when_ the user needs it.

Besides being able to import modules on-demand, the `import()` function can receive an expression. It allows us to pass template literals, in order to dynamically load modules based on a given value.

```js
const res = await import(`../assets/dog${num}.png`);
```

This way, we're not dependent on hard-coded module paths. It adds flexibility to the way you can import modules based on user input, data received from an external source, the result of a function, and so on.


With the module pattern, we can encapsulate parts of our code that should not be publicly exposed. This prevents accidental name collision and global scope pollution, which makes working with multiple dependencies and namespaces less risky. In order to be able to use ES2015 modules in all JavaScript runtimes, a transpiler such as [Babel](https://babeljs.io/) is needed.

## Source

- [patterns.dev/vanilla/module-pattern](https://patterns.dev/vanilla/module-pattern)
