# 🏭 Factory Pattern (2 Factories!)

## **TimeEntryFactory** – domain object creation

**File:** `srv/handler/factories/TimeEntryFactory.ts`

knows all business rules and creates perfectly calculated TimeEntry objects:

```typescript
const factory = container.getFactory<TimeEntryFactory>('timeEntry');

// work time data (used in commands)
const workData = await factory.createWorkTimeData(userService, tx, userId, startTime, endTime, breakMin);
// → calculates automatically: gross, net, overtime, undertime, uses customizing fallbacks

// non-work time data (vacation, sick leave)
const nonWorkData = await factory.createNonWorkTimeData(userService, tx, userId);

// complete entries for generation (use customizing defaults)
const workEntry = factory.createDefaultEntry(userId, date, user);
const weekendEntry = factory.createWeekendEntry(userId, date);
const holidayEntry = factory.createHolidayEntry(userId, date, 'New Year');
```

## **HandlerFactory** – handler instance creation

**File:** `srv/handler/factories/HandlerFactory.ts`

creates handler instances with dependencies from the ServiceContainer:

```typescript
class HandlerFactory {
  constructor(private container: ServiceContainer) {}

  createTimeEntryHandlers(): TimeEntryHandlers {
    return new TimeEntryHandlers(
      this.container.getCommand<CreateTimeEntryCommand>('createTimeEntry'),
      this.container.getCommand<UpdateTimeEntryCommand>('updateTimeEntry'),
    );
  }

  createAllHandlers() {
    return {
      timeEntry: this.createTimeEntryHandlers(),
      generation: this.createGenerationHandlers(),
      balance: this.createBalanceHandlers(),
    };
  }
}
```

**Features:**

- 🏭 encapsulates handler instantiation
- 🔗 resolves dependencies from container
- 🧪 perfect for unit tests
