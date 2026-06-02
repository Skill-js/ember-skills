---
name: ember-concurrency
description: Manage complex asynchronous tasks, debounce, throttle, and prevent double-submissions using the ember-concurrency addon.
---

## Modern Async with Ember Concurrency

While `ember-resources` is excellent for reactive/derived state, **`ember-concurrency`** is the standard for managing complex, user-driven asynchronous operations (like saving forms, debouncing search inputs, or polling).

### 1. Defining a Task

Tasks are defined using the `task` wrapper and generator functions (`*` and `yield` instead of `async`/`await`). 

**Note on modern syntax:** In Octane/Polaris, we use the `task` wrapper function instead of the legacy `@task` decorator, to remain compatible with standard JS environments without custom babel transforms.

```javascript
import Component from '@glimmer/component';
import { task, timeout } from 'ember-concurrency';

export default class SearchComponent extends Component {
  
  searchTask = task(async (query) => {
    // Wait for the user to stop typing
    await timeout(500); 
    
    let response = await fetch(`/api/search?q=${query}`);
    return await response.json();
  }).restartable(); // Automatically cancels the previous run if triggered again
}
```

*Wait, `ember-concurrency` now supports `async`/`await` natively instead of generators! Always use `async`/`await` combined with `timeout` inside `ember-concurrency` v3+.*

### 2. Task Modifiers (Concurrency Control)

The real power of `ember-concurrency` lies in its task modifiers, which dictate how the task behaves if invoked multiple times concurrently.

- `.drop()`: Ignores new invocations if the task is already running. **Use this to prevent double-submissions on save buttons.**
- `.restartable()`: Cancels the currently running task and starts a new one. **Use this for debouncing inputs.**
- `.enqueue()`: Runs invocations sequentially, waiting for the previous to finish.

```javascript
// Example: Prevent double-submission
saveUserTask = task(async (userData) => {
  await fetch('/api/users', { method: 'POST', body: JSON.stringify(userData) });
}).drop();
```

### 3. Using Tasks in Templates

Tasks expose derived state properties that you can bind to directly in the template, eliminating the need for `@tracked isSaving = false;`.

- `task.isRunning`: True if the task is currently executing.
- `task.isIdle`: True if the task is not running.
- `task.lastSuccessful.value`: The return value of the last successful run.
- `task.perform()`: Invokes the task.

```handlebars
<button 
  type="button" 
  disabled={{this.saveUserTask.isRunning}} 
  {{on "click" (fn this.saveUserTask.perform @user)}}
>
  {{#if this.saveUserTask.isRunning}}
    Saving...
  {{else}}
    Save User
  {{/if}}
</button>
```

## Gotchas & Guardrails

- **Do not mix up `ember-resources` and `ember-concurrency`**: Use resources for state that automatically updates when an input changes. Use concurrency for specific actions triggered by user events (clicking, typing).
- **Auto-cancellation**: Unlike native `Promise`, if the component is destroyed while a task is `yield`ing or `await`ing, the task is automatically canceled, preventing memory leaks and "calling set on destroyed object" errors.
