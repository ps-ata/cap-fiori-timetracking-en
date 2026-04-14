# 🏗️ ServiceContainer Pattern (Dependency Injection)

**File:** `srv/handler/container/ServiceContainer.ts`

The **ServiceContainer** is our DI container. It manages **all** dependencies centrally:

```typescript
// on service startup
const container = new ServiceContainer();
container.build(entities); // auto-wiring!

// type-safe resolution
const userService = container.getService<UserService>('user');
const createCommand = container.getCommand<CreateTimeEntryCommand>('createTimeEntry');
```

**Features:**

- 🎯 6 categories: repositories (6), services (5), validators (7), strategies (2), commands (10), factories (2)
- 🔗 auto-wiring of dependencies
- 🛡️ type-safe with generics
- 🧪 perfect for unit tests
