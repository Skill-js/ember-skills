---
name: ember-data
description: Handle data fetching and caching using modern WarpDrive (EmberData) RequestManager and Cache paradigms. Avoid legacy Adapters and Serializers.
---

## Modern EmberData (WarpDrive)

EmberData has evolved into a highly modular, decoupled system often referred to as **WarpDrive**. It shifts away from the monolithic `DS.Model`, `RESTAdapter`, and `JSONAPISerializer` towards a document-centric cache and `RequestManager`.

### 1. RequestManager vs Legacy Adapters

Do not create legacy Adapters or Serializers unless strictly maintaining an old codebase. For new data fetching logic, configure and use a `RequestManager`.

```javascript
// app/services/request-manager.js
import RequestManager from '@ember-data/request';
import Fetch from '@ember-data/request/fetch';
import Service from '@ember/service';

export default class CustomRequestManager extends Service {
  manager = new RequestManager();

  constructor() {
    super(...arguments);
    // Add the fetch handler
    this.manager.use([Fetch]);
  }

  request(options) {
    return this.manager.request(options);
  }
}
```

### 2. Fetching Data with RequestManager

You can inject your request manager anywhere and fire requests. It handles caching, deduplication, and parsing natively.

```javascript
import Component from '@glimmer/component';
import { inject as service } from '@ember/service';
import { tracked } from '@glimmer/tracking';

export default class UserProfile extends Component {
  @service requestManager;
  @tracked user = null;

  constructor() {
    super(...arguments);
    this.loadData();
  }

  async loadData() {
    const { content } = await this.requestManager.request({
      url: `/api/users/${this.args.id}`
    });
    this.user = content;
  }
}
```

### 3. Modern Store and Cache

When you need a global normalized cache (like the classic Store), use `@ember-data/store`.

```javascript
import Store, { CacheHandler } from '@ember-data/store';
import RequestManager from '@ember-data/request';
import Fetch from '@ember-data/request/fetch';

export default class StoreService extends Store {
  constructor() {
    super(...arguments);
    this.requestManager = new RequestManager();
    this.requestManager.use([Fetch]);
    this.requestManager.useCache(CacheHandler);
  }
}
```

Then you can use modern Store APIs like `store.request(query)`.

## Gotchas & Guardrails

- **Avoid `DS.Model`**: Modern architectures prefer treating data as POJOs or using more lightweight representation classes over the heavy legacy `DS.Model`.
- **Avoid `.save()`**: Do not call `model.save()`. Issue a standard `POST`/`PATCH` request using the `RequestManager`.
- **JSON:API Is Not Required**: The legacy Ember Data strongly assumed JSON:API formatting. WarpDrive's RequestManager does not.
