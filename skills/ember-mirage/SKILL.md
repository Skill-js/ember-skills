---
name: ember-mirage
description: Setup API mocking for tests and development using the modern ember-mirage package (avoid the legacy ember-cli-mirage).
---

## Modern API Mocking with Mirage

Mirage is the standard solution for mocking backend APIs in Ember applications, particularly for acceptance and integration tests.

### 1. Use `ember-mirage` (NOT `ember-cli-mirage`)

Modern Ember applications (using Embroider or Vite) must use the **`ember-mirage`** package. 

**Never** install or configure the legacy `ember-cli-mirage` addon, as its older Broccoli build hooks conflict with modern bundlers like Vite.

```bash
# Correct modern installation
ember install ember-mirage
```

### 2. Configuration

Mirage configuration usually lives in `mirage/config.js`.

```javascript
// mirage/config.js
import { discoverEmberDataModels } from 'ember-mirage';
import { createServer } from 'miragejs';

export default function(config) {
  let finalConfig = {
    ...config,
    models: { ...discoverEmberDataModels(), ...config.models },
    routes() {
      this.namespace = '/api';

      this.get('/users', (schema, request) => {
        return schema.users.all();
      });

      this.post('/users', (schema, request) => {
        let attrs = JSON.parse(request.requestBody);
        return schema.users.create(attrs);
      });
    },
  };

  return createServer(finalConfig);
}
```

### 3. Using Mirage in Tests

Import `setupMirage` from `ember-mirage/test-support` to start the mock server before each test.

```javascript
// tests/acceptance/users-test.js
import { module, test } from 'qunit';
import { visit, currentURL } from '@ember/test-helpers';
import { setupApplicationTest } from 'my-app/tests/helpers';
import { setupMirage } from 'ember-mirage/test-support';

module('Acceptance | users', function(hooks) {
  setupApplicationTest(hooks);
  setupMirage(hooks); // Starts Mirage

  test('it shows users', async function(assert) {
    // Seed the mock database
    this.server.createList('user', 5);

    await visit('/users');

    assert.strictEqual(currentURL(), '/users');
    assert.dom('.user-card').exists({ count: 5 });
  });
});
```

## Gotchas & Guardrails

- **Vite Compatibility**: Legacy `ember-cli-mirage` is a major blocker for migrating to Vite. If you encounter an older app, migrating to `ember-mirage` is often step one.
- **WarpDrive Compatibility**: If the app uses modern WarpDrive (EmberData), ensure Mirage is configured to return payloads matching what your RequestManager expects, as modern data layers are not strictly bound to JSON:API.
