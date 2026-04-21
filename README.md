# Vite + Lit Starter Template

A starter template for building modern UI applications with [Lit](https://lit.dev/) Web Components, [Vite](https://vitejs.dev/), and [Aurelia's](https://aurelia.io/) Dependency Injection and EventAggregator.

The included demo renders a list of users fetched from a mock API service, with click-to-select functionality wired through the EventAggregator.

Two Aurelia primitives power the architecture:

- **`Container`** — an Inversion of Control (IoC) container that manages service lifetimes and automatically injects dependencies into components, keeping them decoupled from construction concerns.
- **`EventAggregator`** — a pub/sub message bus that allows components to communicate without holding direct references to each other.

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

### Dependency Injection — Aurelia `Container` + `@instance`

This project uses Aurelia's `Container` as an IoC container to manage service instances and their dependencies. Rather than manually constructing services or passing them through component trees, the container resolves and provides them automatically. This brings several concrete benefits:

- **Decoupling** — components declare what they need, not how to build it.
- **Singleton management** — the container ensures a single shared instance of each service across the entire application.
- **Testability** — services can be swapped out or mocked at the container level without touching component code.
- **Scalability** — adding a new dependency to a component is a one-line change regardless of how complex the service's own dependencies are.

Because Lit Web Components are not instantiated by the container (the browser creates them), a custom `@instance` property decorator is provided in `src/core/decorators.ts`. It hooks into the globally registered container at property initialisation time, making injection feel natural and requiring zero boilerplate beyond the decorator itself:

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

### Cross-Component Communication — Aurelia `EventAggregator`

Aurelia's `EventAggregator` is used as the primary mechanism for communication between components. It implements the pub/sub (publish/subscribe) pattern, acting as a central message bus: publishers fire named events with a payload, and any number of subscribers react to them — without either side holding a reference to the other.

This approach is preferred over native DOM events for complex inter-component communication because:

- **No shared references** — components never import or depend on each other directly.
- **Many-to-many** — multiple components can publish or subscribe to the same event independently.
- **Decoupled orchestration** — component interactions can be added, changed, or removed without modifying the components themselves.

The `EventAggregator` is itself resolved via the DI container using `@instance`:

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
