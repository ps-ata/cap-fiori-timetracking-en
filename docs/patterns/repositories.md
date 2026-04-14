# 💾 Repository Pattern (6 Repositories)

**Files:** `srv/handler/repositories/*.ts`

The **Repository Pattern** abstracts data access and encapsulates all database operations. Each entity has its own repository with domain-specific logic:

```typescript
/**
 * TimeEntryRepository - data access for time entries
 */
export class TimeEntryRepository {
  private TimeEntries: any;

  constructor(entities: any) {
    this.TimeEntries = entities.TimeEntries;
  }

  /**
   * loads entry by user/date
   */
  async getEntryByUserAndDate(
    tx: Transaction,
    userId: string,
    workDate: string,
    excludeId?: string,
  ): Promise<TimeEntry | null> {
    const whereClause: any = { user_ID: userId, workDate };
    if (excludeId) whereClause.ID = { '!=': excludeId };

    const entry = await tx.run(SELECT.one.from(this.TimeEntries).where(whereClause));
    return entry || null; // 🎯 no throw! pure data query
  }

  /**
   * batch insert for performance
   */
  async insertBatch(tx: Transaction, entries: TimeEntry[]): Promise<void> {
    await tx.run(INSERT.into(this.TimeEntries).entries(entries));
  }
}
```

**Features:**

- 💾 complete abstraction of the data layer
- 🔍 domain-specific queries (e.g., `getEntryByUserAndDate`)
- ⚡ performance optimization with batch operations
- 🎯 pure data access without business logic (separation of concerns!)
- 🧪 perfectly mockable for unit tests

**Our 6 repositories:**

- `TimeEntryRepository` – CRUD + queries + batch insert
- `UserRepository` – user lookup by email/ID
- `ProjectRepository` – validation of active projects
- `ActivityTypeRepository` – validation of activity codes
- `WorkLocationRepository` – validation of work locations
- `TravelTypeRepository` – validation of travel types
