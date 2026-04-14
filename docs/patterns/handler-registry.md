# 📋 HandlerRegistry Pattern

**File:** `srv/handler/registry/HandlerRegistry.ts`

structured event-handler registration with **before/on/after** support:

```typescript
registry.register({
  type: 'before',
  event: 'CREATE',
  entity: TimeEntries,
  handler: handlers.handleCreate.bind(handlers),
  description: 'Validate and enrich time entry before creation',
});

registry.apply(service);
```

# 📋 Registrar Pattern

**File:** `srv/handler/registry/HandlerRegistrar.ts`

separates registration logic from business logic:

```typescript
class HandlerRegistrar {
  constructor(private registry: HandlerRegistry) {}

  registerTimeEntryHandlers(handlers: TimeEntryHandlers): void {
    this.registry.register({
      type: 'before',
      event: 'CREATE',
      entity: TimeEntries,
      handler: handlers.handleCreate.bind(handlers),
      description: 'Validate and enrich time entry before creation',
    });
    // ... more registrations
  }

  registerAllHandlers(handlers: { ... }): void {
    this.registerTimeEntryHandlers(handlers.timeEntry);
    this.registerGenerationHandlers(handlers.generation);
    this.registerBalanceHandlers(handlers.balance);
  }
}
```

**Features:**

- 📋 structured registration
- 🎯 separation of concerns
- 🔄 reusable
