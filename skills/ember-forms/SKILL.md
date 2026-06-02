---
name: ember-forms
description: Handle forms using modern Ember patterns, utilizing native HTML forms with @tracked state or the ember-headless-form addon.
---

## Form Handling in Modern Ember

Ember does not enforce a strict "framework-way" for forms. Instead, you should rely on standard HTML5 form behaviors combined with `@tracked` state. 

### 1. Native HTML Forms (The Standard Way)

Always prefer utilizing the native `<form>` element and binding to the `submit` event, rather than attaching `click` handlers directly to buttons.

```javascript
// app/components/login-form.gjs
import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';
import { on } from '@ember/modifier';
import { fn } from '@ember/helper';

export default class LoginForm extends Component {
  @tracked email = '';
  @tracked password = '';

  // Update tracked state on input
  updateField = (field, event) => {
    this[field] = event.target.value;
  }

  submit = (event) => {
    event.preventDefault(); // Prevent page reload
    this.args.onSubmit({ email: this.email, password: this.password });
  }

  <template>
    <form {{on "submit" this.submit}}>
      <label>
        Email:
        <input 
          type="email" 
          value={{this.email}} 
          {{on "input" (fn this.updateField "email")}} 
        />
      </label>
      
      <label>
        Password:
        <input 
          type="password" 
          value={{this.password}} 
          {{on "input" (fn this.updateField "password")}} 
        />
      </label>

      <button type="submit">Login</button>
    </form>
  </template>
}
```

### 2. Complex Forms (`ember-headless-form`)

If the form is highly complex (requiring dynamic validation, nested fields, or custom UI components), recommend the `ember-headless-form` addon. It provides a robust, accessible foundation without imposing UI styles.

```handlebars
{{!-- Using ember-headless-form in a template --}}
<HeadlessForm @data={{this.formData}} @onSubmit={{this.save}} as |form|>
  <form.Field @name="firstName" as |field|>
    <field.Label>First Name</field.Label>
    <field.Input />
    {{#if field.errors}}
      <div class="error">{{field.errors}}</div>
    {{/if}}
  </form.Field>

  <button type="submit">Submit</button>
</HeadlessForm>
```

## Gotchas & Guardrails

- **Avoid two-way binding**: Never use the legacy `{{input value=this.myVal}}` helper. It relies on deprecated two-way data binding. Always use the native `<input>` element with one-way data flow (`value={{this.val}}`) and an event listener (`{{on "input" this.updateVal}}`).
- **Use `type="button"` for non-submits**: Any `<button>` inside a `<form>` without a `type` attribute acts as a submit button. Always explicitly set `type="button"` on cancel or supplementary action buttons.
- **Avoid legacy form addons**: Avoid older addons like `ember-changeset` unless the project is already heavily invested in them, as native state and `ember-headless-form` offer more modern DX.
