# Vite + Lit Starter Template

A starter template for building UI solutions with [Lit](https://lit.dev/) Web Components, [Vite](https://vitejs.dev/), and [Aurelia's](https://aurelia.io/) Dependency Injection and EventAggregator.

## Commands

```bash
npm run dev      # Dev server on port 9000 (auto-opens browser)
npm run build    # tsc && vite build
npm run preview  # Preview production build
```

## Architecture

```
src/
  main.ts              # Bootstrap: import order is critical (see below)
  app.ts               # Root component <my-element>
  index.css            # Global styles
  core/
    di.ts              # Creates and globalizes the Aurelia DI container
    decorators.ts      # Custom @instance property-injection decorator
  components/          # Lit Web Components
  services/            # Singleton services resolved via DI
```

## Key Patterns

### Initialization Order (main.ts)

Always import in this order — metadata must be available before DI and decorators run:

```ts
import 'reflect-metadata'; // 1. Decorator metadata polyfill
import './core/di'; // 2. Create & globalize DI container
import './core/decorators'; // 3. Register custom decorators
import './app'; // 4. Root component
```

### Dependency Injection — `@instance`

Services are injected as class properties using the custom `@instance` decorator, **not** via constructors:

```ts
import { instance } from '../core/decorators';
import { ApiService } from '../services/api-service';

class MyComponent extends LitElement {
  @instance({ type: ApiService }) private apiService!: ApiService;
}
```

The Aurelia `Container` is a singleton (set global in `di.ts`). Services registered with it are singletons by default.

### Creating a New Service

Place services in `src/services/`. Services are plain classes — the DI container resolves them automatically:

```ts
// src/services/my-service.ts
export class MyService {
  doSomething() { ... }
}
```

Inject with `@instance({ type: MyService })` in any component.

### Creating a New Component

Place Lit components in `src/components/`. Use `@customElement` and Lit's `@property` / `@state` decorators:

```ts
import { LitElement, html } from 'lit';
import { customElement, property } from 'lit/decorators.js';

@customElement('my-widget')
export class MyWidget extends LitElement {
  @property() label = '';
  render() {
    return html`<div>${this.label}</div>`;
  }
}
```

### Cross-Component Communication — EventAggregator

Use Aurelia's `EventAggregator` (not native DOM events) for cross-component messaging:

```ts
import { EventAggregator } from 'aurelia-event-aggregator';
import { instance } from '../core/decorators';

class MyComponent extends LitElement {
  @instance({ type: EventAggregator }) private ea!: EventAggregator;

  connectedCallback() {
    super.connectedCallback();
    this.subscription = this.ea.subscribe('MY_EVENT', (data) => { ... });
  }

  disconnectedCallback() {
    super.disconnectedCallback();
    this.subscription?.dispose(); // Always dispose subscriptions
  }
}
```

## TypeScript Notes

- `experimentalDecorators: true` and `emitDecoratorMetadata: true` are required — do not remove them from `tsconfig.json`.
- `useDefineForClassFields: false` is required for Aurelia metadata reflection to work with class properties.
- `strict` mode is enabled.

## Vite Config

The config file is named `xvite.config.ts` (not `vite.config.ts`). Uses `vite-plugin-node-polyfills` for Node.js compatibility.
