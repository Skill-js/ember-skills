---
name: ember-resources
description: Manage complex reactive and asynchronous state cleanly using the ember-resources pattern.
---

## Ember Resources

In modern Ember, managing asynchronous state (like data fetching, polling, or websockets) manually inside components or services often leads to complex, hard-to-maintain teardown logic (memory leaks).

The `ember-resources` pattern solves this by encapsulating reactive state into a "Resource" that is bound to the lifecycle of its host context (e.g., a Component). When the component is destroyed, the resource automatically cleans up.

### 1. Writing a Function-based Resource

The simplest way to use `ember-resources` is via the `resource` wrapper around an async function.

```javascript
// app/components/user-profile.gjs
import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';
import { resource, use } from 'ember-resources';

export default class UserProfile extends Component {
  @tracked userId = 1;

  // The resource re-evaluates automatically whenever `this.userId` changes
  @use userData = resource(({ on }) => {
    let state = new TrackedObject({ isPending: true, data: null, error: null });
    
    // An abort controller can be wired to the resource's cleanup hook
    let controller = new AbortController();
    on.cleanup(() => controller.abort());

    fetch(`/api/users/${this.userId}`, { signal: controller.signal })
      .then(res => res.json())
      .then(data => {
        state.data = data;
        state.isPending = false;
      })
      .catch(err => {
        if (err.name !== 'AbortError') {
          state.error = err;
          state.isPending = false;
        }
      });

    return state;
  });

  <template>
    {{#if this.userData.isPending}}
      <p>Loading...</p>
    {{else if this.userData.error}}
      <p>Error: {{this.userData.error.message}}</p>
    {{else}}
      <p>Name: {{this.userData.data.name}}</p>
    {{/if}}
  </template>
}
```

### 2. Why Resources?

- **Auto-tracking**: They seamlessly integrate with `@tracked` properties. If a tracked argument passed to the resource changes, the resource is torn down and re-invoked.
- **Encapsulation**: You no longer need `willDestroy` hooks in your components just to abort a fetch or clear an interval.

## Gotchas & Guardrails

- **Do not use Controllers/Services for local async state**: AI agents often default to putting local component fetch logic into a Service or Route. Favor Resources for state that is highly localized and reactive.
- **Addon Requirement**: Make sure `ember-resources` is installed in the project. If it isn't, and you cannot install it, fall back to doing manual `try/catch` and `willDestroy` in the component, but standard modern Ember practice heavily favors Resources.
