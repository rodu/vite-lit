# Vite + Lit Starter Template

A starter template for building modern UI applications with [Lit](https://lit.dev/) Web Components, [Vite](https://vitejs.dev/), and [Aurelia's](https://aurelia.io/) Dependency Injection and EventAggregator.

The included demo renders a list of users fetched from a mock API service, with click-to-select functionality wired through the EventAggregator.

## Tech Stack

| Tool                                                                            | Purpose                                |
| ------------------------------------------------------------------------------- | -------------------------------------- |
| [Lit](https://lit.dev/)                                                         | Web Components with reactive rendering |
| [Vite](https://vitejs.dev/)                                                     | Fast dev server and bundler            |
| [aurelia-dependency-injection](https://github.com/aurelia/dependency-injection) | Property-based DI container            |
| [aurelia-event-aggregator](https://github.com/aurelia/event-aggregator)         | Pub/sub cross-component messaging      |

## Getting Started

```bash
npm install
npm run dev      # Start dev server at http://localhost:9000
```

## Commands

| Command           | Description                                        |
| ----------------- | -------------------------------------------------- |
| `npm run dev`     | Start dev server (auto-opens browser on port 9000) |
| `npm run build`   | Type-check and build for production                |
| `npm run preview` | Preview the production build locally               |

## Project Structure

```
src/
  main.ts              # Entry point — import order is critical
  app.ts               # Root component <my-element>
  index.css            # Global styles
  core/
    di.ts              # Creates and globalizes the Aurelia DI container
    decorators.ts      # Custom @instance property-injection decorator
  components/          # Lit Web Components
  services/            # Singleton services resolved via DI
```

## Key Patterns

### Dependency Injection — `@instance`

Services are injected as class properties using the custom `@instance` decorator:

```ts
import { instance } from '../core/decorators';
import { ApiService } from '../services/api-service';

class MyComponent extends LitElement {
  @instance({ type: ApiService }) private apiService!: ApiService;
}
```

Services are singletons by default — the same instance is shared across all components.

### Adding a New Service

Create a plain class in `src/services/` and inject it anywhere with `@instance`:

```ts
// src/services/my-service.ts
export class MyService {
  doSomething() { ... }
}
```

### Adding a New Component

Create a Lit component in `src/components/`:

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

Use the `EventAggregator` for decoupled messaging between components:

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
    this.subscription?.dispose(); // Always dispose to avoid memory leaks
  }
}
```

## TypeScript Requirements

The following `tsconfig.json` settings are required and must not be removed:

```json
{
  "experimentalDecorators": true,
  "emitDecoratorMetadata": true,
  "useDefineForClassFields": false
}
```

`useDefineForClassFields: false` is necessary for Aurelia's metadata reflection to work correctly with class properties. `strict` mode is enabled.

## Notes

- The Vite config file is named `xvite.config.ts` (not `vite.config.ts`).
- Bootstrap import order in `main.ts` matters — `reflect-metadata` must be first, followed by `di`, `decorators`, then component imports.
