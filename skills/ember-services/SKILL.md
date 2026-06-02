---
name: ember-services
description: Implement singleton state and logic using Ember Services and Dependency Injection (@service).
---

## Modern Ember Services

Services are long-lived singleton objects that hold state or encapsulate cross-cutting concerns (e.g., authentication, feature flags, shopping carts). 

### 1. Creating a Service

Services are simple native classes extending `@ember/service`. You can use `@tracked` for reactive state.

```javascript
// app/services/shopping-cart.js
import Service from '@ember/service';
import { tracked } from '@glimmer/tracking';

export default class ShoppingCartService extends Service {
  @tracked items = [];

  addItem(item) {
    // Note: To trigger reactivity on arrays, reassign or use tracked built-ins
    this.items = [...this.items, item];
  }

  get totalItems() {
    return this.items.length;
  }
}
```

### 2. Injecting a Service

To use a service in a Component, Controller, or Route, inject it using the `@service` decorator.

```javascript
// app/components/cart-badge.gjs
import Component from '@glimmer/component';
import { inject as service } from '@ember/service';

export default class CartBadge extends Component {
  // Property name matches the service file name (camelCased)
  @service shoppingCart;

  // Or specify explicitly: @service('shopping-cart') cart;

  <template>
    <div class="badge">
      Items in cart: {{this.shoppingCart.totalItems}}
    </div>
  </template>
}
```

### 3. Service Initialization & Teardown

If your service needs to perform setup (like subscribing to events) or teardown, avoid classic `init()`. Instead, rely on the `constructor` and the `willDestroy` hook.

```javascript
export default class WebSocketService extends Service {
  constructor() {
    super(...arguments);
    this.socket = new WebSocket('ws://example.com');
  }

  willDestroy() {
    super.willDestroy(...arguments);
    this.socket.close();
  }
}
```

## Gotchas & Guardrails

- **Array/Object Reactivity**: Mutating an array via `.push()` or an object via `obj.key = val` does **not** trigger `@tracked` reactivity in Ember Octane. You must reassign the property (e.g. `this.items = [...this.items, newItem]`) or use `TrackedArray`/`TrackedObject` from external addons.
- **Service Naming**: The name of the injected property must perfectly camelCase match the dashed file name of the service if you don't explicitly pass the string to `@service('service-name')`.
- **Complex Async State**: If a service's main job is managing complex asynchronous state (like polling or managing a fetch lifecycle), strongly consider using the `ember-resources` pattern instead of a raw service.
