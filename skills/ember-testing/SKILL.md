---
name: ember-testing
description: Write modern async tests in Ember using QUnit, @ember/test-helpers, setupRenderingTest, and setupApplicationTest.
---

## Modern Ember Testing

Ember provides robust out-of-the-box testing. Modern Ember tests are primarily `async`/`await` and rely heavily on `@ember/test-helpers`.

### 1. Rendering Tests (Components)

Use `setupRenderingTest` to test components in isolation. Do **not** use `moduleForComponent`.

If you are using Polaris `.gjs`/`.gts` Template Tags, you can pass the component class directly into the `render` helper, removing the need to resolve it by string name from the container.

```javascript
// tests/integration/components/hello-world-test.js
import { module, test } from 'qunit';
import { setupRenderingTest } from 'my-app/tests/helpers';
import { render, click } from '@ember/test-helpers';
import HelloWorld from 'my-app/components/hello-world'; // Import the .gjs component directly

module('Integration | Component | hello-world', function (hooks) {
  setupRenderingTest(hooks);

  test('it increments the count', async function (assert) {
    // Render the component using a local template tag
    await render(<template><HelloWorld @name="Agent" /></template>);

    assert.dom('h2').hasText('Hello Agent!');
    assert.dom('p').hasText('Count: 0');

    await click('button');

    assert.dom('p').hasText('Count: 1');
  });
});
```

### 2. Application Tests (Acceptance)

Use `setupApplicationTest` for testing full user flows.

```javascript
// tests/acceptance/login-test.js
import { module, test } from 'qunit';
import { visit, currentURL, fillIn, click } from '@ember/test-helpers';
import { setupApplicationTest } from 'my-app/tests/helpers';

module('Acceptance | login', function (hooks) {
  setupApplicationTest(hooks);

  test('visiting /login and authenticating', async function (assert) {
    await visit('/login');
    assert.strictEqual(currentURL(), '/login');

    await fillIn('input[type="email"]', 'agent@example.com');
    await fillIn('input[type="password"]', 'secret');
    await click('button[type="submit"]');

    assert.strictEqual(currentURL(), '/dashboard');
  });
});
```

### 3. Common Test Helpers

Always import these from `@ember/test-helpers`:
- `render(template)`
- `click(selector)`
- `fillIn(selector, text)`
- `typeIn(selector, text)`
- `triggerEvent(selector, eventName)`
- `find(selector)` / `findAll(selector)`
- `waitFor(selector)`

## Gotchas & Guardrails

- **Always `await`**: Practically every test helper in Ember is asynchronous. Always use `await click(...)`, `await fillIn(...)`. Forgetting `await` is the #1 cause of flaky or failing tests.
- **Sync Assertions**: Do not put `assert` statements inside `then()`. Use `async`/`await` and write assertions synchronously at the top level of the block.
- **`assert.dom`**: Prefer `qunit-dom` assertions like `assert.dom('.my-class').exists()` over traditional `assert.ok(document.querySelector('.my-class'))`.
