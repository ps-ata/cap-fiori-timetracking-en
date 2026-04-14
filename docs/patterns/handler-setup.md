# 🏗️ Builder Pattern (Fluent API)

**File:** `srv/handler/setup/HandlerSetup.ts`

builder with fluent API for elegant handler setup:

```typescript
class HandlerSetup {
  static create(container: ServiceContainer, registry: HandlerRegistry): HandlerSetup {
    return new HandlerSetup(container, registry);
  }

  withTimeEntryHandlers(): this {
    this.handlers.timeEntry = this.factory.createTimeEntryHandlers();
    this.registrar.registerTimeEntryHandlers(this.handlers.timeEntry);
    return this;
  }

  withAllHandlers(): this {
    return this.withTimeEntryHandlers().withGenerationHandlers().withBalanceHandlers();
  }

  apply(service: ApplicationService): void {
    this.registry.apply(service);
  }
}
```

**usage in TrackService:**

```typescript
private setupHandlers(): void {
  this.registry = new HandlerRegistry();

  HandlerSetup
    .create(this.container, this.registry)
    .withAllHandlers()
    .apply(this);
}
```

**Features:**

- ⛓️ chainable API
- 🎨 very elegant and readable
- 🔧 flexible – can selectively add handlers
- 🧩 combines factory + registrar
