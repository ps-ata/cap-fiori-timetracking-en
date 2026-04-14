# ADR 0003: Centralized Time Calculation via Factory and Services

## Status

Accepted - After initial TimeEntry prototypes

## Context and Problem Statement

Time values such as `durationHoursNet`, `overtimeHours`, and `expectedDailyHoursDec` must remain consistent on the server side, whether entries are created manually or automatically. Early tests showed deviations because handlers used different calculations or UI values leaked through. Generated entries (weekend, holiday, default) also need shared rules.

## Decision Factors

- Single definition of time calculation including rounding and error handling.
- Reusable generation of entries for monthly and yearly runs.
- Strict control to prevent UI clients from manipulating calculated fields.
- Access to user-specific target hours in every operation.

## Considered Options

### Option A - Duplicate calculations in handlers or commands

- Each handler calculates duration, break, and overtime itself.
- Generation creates objects ad hoc.

### Option B - TimeEntryFactory plus TimeCalculationService as central source

- `TimeEntryFactory` creates work, non-work, default, weekend, and holiday entries (`srv/track-service/handler/factories/TimeEntryFactory.ts`).
- `TimeCalculationService` handles conversions and rounding (`srv/track-service/handler/services/TimeCalculationService.ts`).
- Commands call factory methods and only adopt returned values (`srv/track-service/handler/commands/time-entry/CreateTimeEntryCommand.ts`).

## Decision

We implement Option B. All time calculations go through TimeCalculationService, while TimeEntryFactory produces domain objects. Commands validate inputs, request target hours through UserService, and apply calculated values to the request. Generation strategies use the same factory for consistent UI values.

## Consequences

- Positive: Consistent rounding to two decimal places and unified error messages.
- Positive: Pre-calculated values from automatic entries can be accepted unchanged (see `CreateTimeEntryCommand.execute`).
- Positive: Tests only need to verify the factory and calculation service, not every handler variant.
- Negative: Developers must introduce new fields in the factory and service before they can be used in commands.
- Negative: Strong coupling to UserService for target hours, causing a dependency on `Transaction`.

## References

- `srv/track-service/handler/factories/TimeEntryFactory.ts`
- `srv/track-service/handler/services/TimeCalculationService.ts`
- `srv/track-service/handler/services/UserService.ts`
- `srv/track-service/handler/commands/time-entry/CreateTimeEntryCommand.ts`
- `srv/track-service/handler/strategies/YearlyGenerationStrategy.ts`
