---
name: factory-pattern
description: Use factory functions to create new objects without the `new` keyword.
license: MIT
metadata:
  author: patterns.dev
  version: "1.1"
paths:
  - "**/*.js"
  - "**/*.ts"
related_skills:
  - "module-pattern"
  - "singleton-pattern"
---

# Factory Pattern

With the factory pattern we can use **factory functions** in order to create new objects. A function is a factory function when it returns a new object without the use of the `new` keyword!

Say that we need many users for our application. We can create new users with a `firstName`, `lastName`, and `email` property. The factory function adds a `fullName` property to the newly created object as well, which returns the `firstName` and the `lastName`.

## When to Use

- Use this when you need to create multiple objects that share the same properties
- This is helpful when object creation depends on a certain environment or configuration

## Instructions

- Use factory functions to return custom objects based on current environment or user-specific configuration
- Prefer ES6 arrow functions for concise factory function definitions
- Consider using class constructors instead when memory efficiency matters, as instances share prototype methods

## Details

```js
const createUser = ({ firstName, lastName, email }) => ({
  firstName,
  lastName,
  email,
  fullName() {
    return `${this.firstName} ${this.lastName}`;
  },
});
```

Perfect! We can now easily create multiple users by invoking the `createUser` function.

The factory pattern can be useful if we're creating relatively complex and configurable objects. It could happen that the values of the keys and values are dependent on a certain environment or configuration. With the factory pattern, we can easily create new objects that contain the custom keys and values!

```js
const createObjectFromArray = ([key, value]) => ({
  [key]: value,
});

createObjectFromArray(["name", "John"]); // { name: "John" }
```

### Pros

The factory pattern is useful when we have to create multiple smaller objects that share the same properties. A factory function can easily return a custom object depending on the current environment, or user-specific configuration.

### Cons

In JavaScript, the factory pattern isn't much more than a function that returns an object without using the `new` keyword. ES6 arrow functions allow us to create small factory functions that implicitly return an object each time.

However, in many cases it may be more memory efficient to create new instances instead of new objects each time.

```js
class User {
  constructor(firstName, lastName, email) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.email = email;
  }

  fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}

const user1 = new User({
  firstName: "John",
  lastName: "Doe",
  email: "john@doe.com",
});

const user2 = new User({
  firstName: "Jane",
  lastName: "Doe",
  email: "jane@doe.com",
});
```

## Source

- [patterns.dev/vanilla/factory-pattern](https://patterns.dev/vanilla/factory-pattern)

### References

- [JavaScript Factory Functions with ES6+](https://medium.com/javascript-scene/javascript-factory-functions-with-es6-4d224591a8b1) - Eric Elliott
