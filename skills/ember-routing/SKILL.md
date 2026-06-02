---
name: ember-routing
description: Manage Ember.js routes, model hooks, dynamic segments, and use the RouterService for programmatic navigation.
---

## Modern Ember Routing

Ember.js utilizes a file-system based routing architecture. Each route typically corresponds to a URL segment and is responsible for fetching the data needed to render a template.

### 1. Route Definition

Routes are defined in `app/router.js` (or `app/router.ts`) and implemented in `app/routes/`.

```javascript
// app/router.js
Router.map(function () {
  this.route('posts', function() {
    this.route('post', { path: '/:post_id' }); // dynamic segment
  });
});
```

### 2. The Model Hook

The `model` hook fetches data. In modern Ember, it's common to fetch plain JSON via `fetch` or use modern Ember Data patterns.

```javascript
// app/routes/posts/post.js
import Route from '@ember/routing/route';

export default class PostRoute extends Route {
  async model(params) {
    const response = await fetch(`/api/posts/${params.post_id}`);
    if (!response.ok) {
      throw new Error('Network response was not ok');
    }
    return response.json();
  }
}
```

### 3. Programmatic Navigation (RouterService)

To navigate programmatically (e.g., after a form submission), inject the `RouterService`. **Avoid using legacy Controller `transitionToRoute`**.

```javascript
// app/components/post-form.gjs
import Component from '@glimmer/component';
import { inject as service } from '@ember/service';
import { on } from '@ember/modifier';

export default class PostForm extends Component {
  @service router;

  savePost = async () => {
    // ... save logic
    this.router.transitionTo('posts.post', newPostId);
  }

  <template>
    <button type="button" {{on "click" this.savePost}}>Save</button>
  </template>
}
```

### 4. Template Navigation

Use the `<LinkTo>` component for declarative navigation in templates.

```handlebars
<LinkTo @route="posts.post" @model={{@post.id}}>
  Read more
</LinkTo>
```

## Gotchas & Guardrails

- **Controller `transitionToRoute` is legacy**: Always use `@service router` and `this.router.transitionTo(...)`.
- **Query Parameters**: When working with query params, note that they are typically defined on the *Controller*, not the Route, though `this.router.transitionTo('route', { queryParams: { q: 'value' } })` is used to navigate.
- **`setupController`**: Only override `setupController` in a Route if you absolutely must pass additional, non-model data to the controller. Usually, returning a hash from the `model` hook is cleaner.
