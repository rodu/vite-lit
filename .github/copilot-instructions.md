This project is a Vite + Lit Web Components starter using Aurelia's Dependency Injection and EventAggregator.

## Project Structure

```
src/
  main.ts              # Bootstrap entry point — import order is critical
  app.ts               # Root component <my-element>
  index.css            # Global styles
  core/
    di.ts              # Creates and globalizes the Aurelia DI container
    decorators.ts      # Custom @instance property-injection decorator
  components/          # Lit Web Components
  services/            # Singleton services resolved via DI
```

The Vite config file is `xvite.config.ts` (not `vite.config.ts`).

## Bootstrap Order (main.ts)

Always import in this exact order — metadata must be available before DI and decorators run:

```ts
import 'reflect-metadata'; // 1. Decorator metadata polyfill
import './core/di'; // 2. Create & globalize DI container
import './core/decorators'; // 3. Register custom decorators
import './app'; // 4. Root component
```

## Dependency Injection

Inject services as class properties using the `@instance` decorator — never via constructors:

```ts
import { instance } from '../core/decorators';
import { MyService } from '../services/my-service';

class MyComponent extends LitElement {
  @instance({ type: MyService }) private myService!: MyService;
}
```

The Aurelia `Container` is a singleton. All services are singletons by default.

## Creating a New Service

Place services in `src/services/`. Services are plain classes — no registration needed:

```ts
// src/services/my-service.ts
export class MyService {
  doSomething() { ... }
}
```

## Creating a New Component

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

## Cross-Component Communication

Use Aurelia's `EventAggregator` for cross-component messaging — not native DOM events:

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

## TypeScript Requirements

- `experimentalDecorators: true` and `emitDecoratorMetadata: true` must remain in `tsconfig.json`
- `useDefineForClassFields: false` is required for Aurelia metadata reflection to work
- `strict` mode is enabled
