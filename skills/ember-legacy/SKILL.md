---
name: ember-legacy
description: Identify and avoid legacy Ember.js patterns (Mixins, Classic Components, Ember.Object) unless explicitly maintaining older codebases.
---

## Ember Legacy Guardrails

Ember.js has evolved significantly. AI coding agents often hallucinate legacy patterns because they dominated the internet for years.

**If you are writing new code in a modern Ember Octane or Polaris project, you MUST avoid the following patterns:**

### 1. `Ember.Object` and Mixins

**Never** use `Ember.Object.extend` or `Ember.Mixin.create`. They were the foundation of the classic object model but are completely removed in modern codebases.

**Legacy (Do NOT do this):**
```javascript
import EmberObject from '@ember/object';
import Mixin from '@ember/object/mixin';

const MyMixin = Mixin.create({
  someProp: true
});

const Person = EmberObject.extend(MyMixin, {
  name: 'John'
});
```

**Modern Solution:** Use native JavaScript `class` syntax. If you need shared behavior, use utility functions, class inheritance, or composition.

### 2. `get()` and `set()`

**Never** use `this.get('propertyName')` or `this.set('propertyName', value)`.

**Legacy (Do NOT do this):**
```javascript
this.set('count', this.get('count') + 1);
```

**Modern Solution:** Mutate properties directly. Use `@tracked` to ensure the template updates.
```javascript
this.count = this.count + 1;
// or
this.count++;
```

### 3. Classic Components (`@ember/component`)

**Never** use `Ember.Component` or `import Component from '@ember/component'`.

Classic components relied on two-way data binding, an implicit `this` context in the template, and a wrapper element `tagName`.

**Legacy (Do NOT do this):**
```javascript
import Component from '@ember/component';

export default Component.extend({
  tagName: 'span',
  classNames: ['my-span']
});
```

**Modern Solution:** Always use **Glimmer Components** (`import Component from '@glimmer/component'`). Glimmer components have no wrapper element (you define the wrapper in the template), and arguments are passed via `this.args` and `@argName`.

### 4. `@computed` and Observers

**Never** use `@computed` or `addObserver`.

**Legacy (Do NOT do this):**
```javascript
import { computed } from '@ember/object';

export default EmberObject.extend({
  firstName: '',
  lastName: '',
  fullName: computed('firstName', 'lastName', function() {
    return `${this.get('firstName')} ${this.get('lastName')}`;
  })
});
```

**Modern Solution:** Use native JS getters (`get propertyName()`). They will automatically track any `@tracked` properties they access.

### 5. `moduleForComponent`

**Never** use `moduleForComponent` in testing.

**Modern Solution:** Use `setupRenderingTest(hooks)` from `@ember/test-helpers`.

### 6. The `actions` hash and `{{action}}` modifier

**Never** use the `actions: {}` hash in your classes, and **never** use the `{{action}}` modifier or helper in templates.

**Legacy (Do NOT do this):**
```javascript
export default Component.extend({
  actions: {
    save() { /* ... */ }
  }
});
```
```handlebars
<button {{action "save"}}>Save</button>
```

**Modern Solution:** Use the `@action` decorator (only strictly required if passing the function as an argument, though common practice) and the native `{{on}}` modifier with the `{{fn}}` helper.
```javascript
import { action } from '@ember/object';

export default class MyComponent extends Component {
  @action
  save(arg1) { /* ... */ }
}
```
```handlebars
<button type="button" {{on "click" (fn this.save "myArg")}}>Save</button>
```

### 7. `{{input}}` and Two-Way Binding

**Never** use the built-in `{{input}}` or `{{textarea}}` curly helpers. They rely on legacy two-way data binding and obscure native attributes.

**Legacy (Do NOT do this):**
```handlebars
{{input value=this.myProp}}
```

**Modern Solution:** Use native HTML elements with one-way data binding and an explicit event handler.
```handlebars
<input value={{this.myProp}} {{on "input" this.updateProp}} />
```

### 8. Controllers for Component Logic

**Avoid** putting heavy application logic or complex component state in Controllers (`@ember/controller`). 

**Modern Solution:** In modern Ember, controllers are generally only used for handling Route Query Parameters. Encapsulate business logic and UI state into Glimmer Components, Services, or `ember-resources`.
