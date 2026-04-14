# 📋 Strategy Pattern

**Files:** `srv/handler/strategies/*.ts`

The **Strategy Pattern** enables interchangeable algorithms for different generation scenarios. Each strategy encapsulates a specific algorithm and is independently exchangeable:

```typescript
/**
 * MonthlyGenerationStrategy - generates all working days of a month
 */
export class MonthlyGenerationStrategy {
  constructor(
    private timeEntryFactory: TimeEntryFactory,
    private customizingService: CustomizingService,
  ) {}

  generateMissingEntries(userID: string, user: User, existingDates: Set<string>): TimeEntry[] {
    const monthData = DateUtils.getCurrentMonthData();

    const userDefaults = this.customizingService.getUserDefaults();
    const workingDaysPerWeek = user.workingDaysPerWeek ?? userDefaults.fallbackWorkingDays;
    const newEntries: TimeEntry[] = [];

    for (let day = 1; day <= monthData.daysInMonth; day++) {
      const currentDate = new Date(monthData.year, monthData.month, day);

      // skip weekends & existing entries
      if (!DateUtils.isWorkingDay(currentDate, workingDaysPerWeek)) {
        continue;
      }

      const dateString = DateUtils.toLocalDateString(currentDate);
      if (existingDates.has(dateString)) continue;

      // factory creates perfectly calculated entries
      const entry = this.timeEntryFactory.createDefaultEntry(userID, currentDate, user);
      newEntries.push(entry);
    }

    return newEntries;
  }
}
```

**Features:**

- 🔄 interchangeable algorithms without code changes
- 🎯 each strategy knows its specific business logic
- 🏭 utilizes factory for consistent entry creation
- 📅 weekend detection and date utilities
- ⚡ performance-optimized with sets for lookup
