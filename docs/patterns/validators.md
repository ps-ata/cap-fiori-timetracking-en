# ✅ Validator Pattern (7 Validators)

**Files:** `srv/handler/validators/*.ts`

The **Validator Pattern** encapsulates complex validation logic in reusable classes. Each validator focuses on a specific domain and follows the **Single Responsibility Principle**:

```typescript
/**
 * ProjectValidator - validates project references
 */
export class ProjectValidator {
  constructor(private projectRepository: ProjectRepository) {}

  async validateActive(tx: Transaction, projectId: string): Promise<void> {
    const project = await this.projectRepository.findByIdActive(tx, projectId);
    if (!project) {
      throw new Error('Project is invalid or inactive.');
    }
  }
}

/**
 * ActivityTypeValidator - validates activity codes
 */
export class ActivityTypeValidator {
  constructor(private activityTypeRepository: ActivityTypeRepository) {}

  async validateExists(tx: Transaction, code: string): Promise<void> {
    const activity = await this.activityTypeRepository.findByCode(tx, code);
    if (!activity) {
      throw new Error('Invalid activity code.');
    }
  }
}

/**
 * TimeEntryValidator - orchestrates entry validation
 */
export class TimeEntryValidator {
  constructor(
    private projectValidator: ProjectValidator,
    private activityTypeValidator: ActivityTypeValidator,
    private timeEntryRepository: TimeEntryRepository,
  ) {}

  async validateReferences(tx: Transaction, entryData: Partial<TimeEntry>): Promise<void> {
    // delegates to specialized validators
    if (entryData.project_ID) {
      await this.projectValidator.validateActive(tx, entryData.project_ID);
    }
    if (entryData.activity_code) {
      await this.activityTypeValidator.validateExists(tx, entryData.activity_code);
    }
  }
}
```

**Features:**

- ✅ **Separation of concerns** – each validator has one responsibility
- 🎯 **Domain-specific rules** – TimeEntry, Project, ActivityType, Generation, Balance
- 🔗 **Validator composition** – TimeEntryValidator uses Project & ActivityType validators
- 🛡️ **Consistent error messages** with logging
- 🧪 **Isolated testability** – pure business logic without CAP dependencies

**Our 7 validators:**

- `ProjectValidator` – project activity validation
- `ActivityTypeValidator` – activity code validation
- `WorkLocationValidator` – work location validation
- `TravelTypeValidator` – travel type validation
- `TimeEntryValidator` – entry validation + change detection (uses Project, ActivityType, WorkLocation, TravelType)
- `GenerationValidator` – user, state code, year validation
- `BalanceValidator` – year/month plausibility check
