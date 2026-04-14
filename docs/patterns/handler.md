# 🊭 Handler Pattern (3 handler classes)

**Files:** `srv/handler/handlers/*.ts`

the **Handler Pattern** separates event handling from business logic. handlers are the "orchestrators" that react to CAP events and delegate the actual work to commands:

```typescript
/**
 * TimeEntryHandlers – handlers for TimeEntry CRUD operations
 */
export class TimeEntryHandlers {
  constructor(
    private createCommand: CreateTimeEntryCommand,
    private updateCommand: UpdateTimeEntryCommand,
  ) {}

  /**
   * Handler: create time entry (before CREATE)
   * delegates business logic to command
   */
  async handleCreate(req: any): Promise<void> {
    try {
      const tx = cds.transaction(req) as any;
      const calculatedData = await this.createCommand.execute(tx, req.data);

      // transfer calculated data to request
      // CAP framework then automatically performs the INSERT
      Object.assign(req.data, calculatedData);
    } catch (error: any) {
      req.reject(error.code || 400, error.message);
    }
  }

  /**
   * Handler: update time entry (before UPDATE)
   */
  async handleUpdate(req: any): Promise<void> {
    try {
      const tx = cds.transaction(req) as any;
      const calculatedData = await this.updateCommand.execute(tx, req.data);
      Object.assign(req.data, calculatedData);
    } catch (error: any) {
      req.reject(error.code || 400, error.message);
    }
  }
}
```

**Features:**

- 🊭 clear separation: handler = orchestration, command = business logic
- 🔗 dependency injection of commands
- 🛡️ central error handling
- 📋 grouping by domain (CRUD / generation / balance)
- 🎯 thin layer – only delegation, no business logic

**Our 3 handler classes:**

- `TimeEntryHandlers` – CRUD operations (CREATE/UPDATE/DELETE)
- `GenerationHandlers` – bulk generation (monthly/yearly)
- `BalanceHandlers` – balance queries (monthly/current/recent)
