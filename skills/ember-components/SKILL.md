---
name: ember-components
description: Write modern Ember UI components using Polaris template tags (.gjs/.gts), Glimmer components, and @tracked state. Avoid legacy Ember components.
---

## Modern Ember Components

When writing UI components for modern Ember.js (Octane/Polaris), always use **Glimmer Components** (`@glimmer/component`) and **native ES classes**.

### 1. File Format: Template Tags (`.gjs` / `.gts`)

If the project supports Polaris paradigms (Ember 4.x/5.x+ with Glint/Embroider), author components using a single file with `<template>` tags instead of separate `.js` and `.hbs` files.

```javascript
// app/components/hello-world.gjs
import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';
import { on } from '@ember/modifier';

export default class HelloWorld extends Component {
  @tracked count = 0;

  increment = () => {
    this.count++;
  }

  <template>
    <div class="hello-world">
      <h2>Hello {{@name}}!</h2>
      <p>Count: {{this.count}}</p>
      <button type="button" {{on "click" this.increment}}>+1</button>
    </div>
  </template>
}
```

### 2. State & Reactivity

- Use `@tracked` for any reactive state.
- Use native JS getters (`get propertyName() {}`) for derived state. 
- **Never** use `@computed` or `.get()` / `.set()`. Mutate state directly (e.g., `this.count++`).

```javascript
import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';

export default class UserProfile extends Component {
  @tracked firstName = 'John';
  @tracked lastName = 'Doe';

  // Derived state auto-tracks the @tracked properties it accesses
  get fullName() {
    return `${this.firstName} ${this.lastName}`;
  }
}
```

### 3. Component Arguments (Args)

Arguments passed to a component are accessible via `this.args` in JS, and `@` in the template.

- **Args are read-only.** Never attempt to mutate `this.args` directly.
- Instead, invoke actions passed in via args.

```javascript
// Caller: <MyForm @initialValue="Hello" @onSubmit={{this.save}} />

export default class MyForm extends Component {
  @tracked value = this.args.initialValue;

  submit = () => {
    this.args.onSubmit(this.value);
  }
}
```

### 4. DOM Manipulation (Modifiers)

**Never use classic lifecycle hooks** like `didInsertElement` for DOM interactions. Instead, use Element Modifiers.

```javascript
import Component from '@glimmer/component';
import { modifier } from 'ember-modifier';

const autofocus = modifier((element) => {
  element.focus();
});

export default class MyInput extends Component {
  <template>
    <input type="text" {{autofocus}} />
  </template>
}
```

## Gotchas & Guardrails

- **Strict Mode in `.gjs`/`.gts`**: You must explicitly import components, helpers, and modifiers to use them in a `<template>`. They are not magically resolved from the global namespace.
- **`Ember.Component`**: Never import `Component from '@ember/component'`. Always use `@glimmer/component`.
- **`this` context**: In templates, backing class properties require `this.` (e.g., `{{this.count}}`), while arguments use `@` (e.g., `{{@title}}`).
