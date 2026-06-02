---
name: ember-tooling
description: Use the modern app-blueprint, ember-cli generators, Embroider build setups (Vite/Webpack), and modern TypeScript configs (app-tsconfig, Glint).
---

## Ember CLI and Modern Build Tooling

### 1. The Modern App Blueprint

To generate a brand new Ember application using the modern Vite-based architecture, always use the `@ember/app-blueprint`.

```bash
npx ember-cli@latest init --blueprint @ember/app-blueprint
```

This avoids legacy Webpack/Broccoli defaults and sets up WarpDrive, Vite, and modern linting out of the box.

### 2. Generators

Always use `ember-cli` generators to scaffold new files. They ensure the correct boilerplate, location, and automatically create the corresponding test files.

```bash
# Generate a component
ember generate component my-component

# Generate a route
ember generate route my-route

# Generate a service
ember generate service my-service

# Generate a modifier
ember generate modifier my-modifier
```

**Note**: To use `.gjs`/`.gts` by default, ensure the project has `@embroider/router` or similar Polaris tooling configured, or pass the specific blueprint if required by the project.

### 3. Embroider

Modern Ember replaces the legacy Broccoli build system (`ember-cli-build.js` returning `app.toTree()`) with **Embroider**. Embroider compiles Ember applications into standard npm packages, allowing them to be bundled by Vite or Webpack.

If you are maintaining an `ember-cli-build.js` file for an Embroider project, it will look like this:

```javascript
// ember-cli-build.js
'use strict';

const EmberApp = require('ember-cli/lib/broccoli/ember-app');

module.exports = function (defaults) {
  let app = new EmberApp(defaults, {
    // Add options here
  });

  const { Webpack } = require('@embroider/webpack');
  return require('@embroider/compat').compatBuild(app, Webpack, {
    staticAddonTestSupportTrees: true,
    staticAddonTrees: true,
    staticHelpers: true,
    staticModifiers: true,
    staticComponents: true,
  });
};
```

### 4. TypeScript Configuration (@ember/app-tsconfig / @ember/library-tsconfig)

When configuring TypeScript for a modern Ember app, always extend the official `@ember/app-tsconfig` rather than writing compiler options from scratch. This ensures compatibility with Ember's build pipeline. For Ember addons, use `@ember/library-tsconfig` instead.

```json
{
  "extends": "@ember/app-tsconfig",
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "my-app/*": ["app/*"]
    }
  }
}
```

### 5. Glint (TypeScript for Templates)

Modern Ember uses **Glint** to type-check `.gjs`/`.gts` files and `.hbs` templates.

- Run `npx glint` to typecheck the project.
- In a `.gts` file, ensure component signatures are properly defined.

```typescript
import Component from '@glimmer/component';

interface MyComponentSignature {
  Args: {
    title: string;
    count?: number;
  };
  Blocks: {
    default: [];
  };
  Element: HTMLDivElement;
}

export default class MyComponent extends Component<MyComponentSignature> {
  // ...
}
```

### 6. Optional Features (@ember/optional-features)

Ember allows enabling or disabling specific framework features using the `@ember/optional-features` package. This is often used to disable legacy behaviors in older codebases or to prepare for an upgrade.

```bash
# Enable a feature
ember feature:enable feature-name

# Disable a feature (e.g., to drop legacy jQuery integration)
ember feature:disable jquery-integration
```

## Gotchas & Guardrails

- **Do not manually create files**: It is error-prone. Always use `ember g ...`.
- **Broccoli Addons**: When configuring modern Embroider builds, legacy Broccoli-based addons might fail if `staticAddonTrees` is true. If build errors occur, selectively disable static analysis for those specific legacy addons in the `compatBuild` options.
