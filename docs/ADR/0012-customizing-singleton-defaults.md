# ADR 0012: Customizing Singleton for Global Defaults

## Status

Accepted – Iteration 6 (Global Defaults Refactoring)

## Context and Problem Statement

So far, many global default values were hard-coded in the code or stored as CDS `default` values. This includes, among others, work start time (08:00), break duration, entry type codes (`W`, `O`, `H`), source codes (`UI`, `GENERATED`), fallback values for weekly hours/workdays, and thresholds for balance, vacation, and sick leave calculations. This hard-coding caused the following issues:

1. **High change effort:** Every functional adjustment required a code change and deployment.
2. **Inconsistencies:** Different components maintained their own defaults (factories, commands, validators, services, UI).
3. **Lack of transparency:** There was no central place where key users could inspect or maintain defaults.
4. **Testing pain:** Unit tests had to mock values or maintain duplicates; integration tests had no controlled source.
5. **Internationalization & locale:** `DateUtils` was hard-coded to `de-DE`, even though other tenants are possible.

We need a central, maintainable source for all non-user-specific defaults with consistent access.

## Decision Factors

- **Single source of truth:** All components should use the same default value.
- **Runtime configurability:** Key users should be able to change defaults without deployment.
- **Performant access:** Values should be loaded once and reused multiple times (caching).
- **Testability:** Unit tests should be able to mock defaults easily; integration tests need initial seed data.
- **Security:** Default maintenance should be restricted by role.
- **Clarity:** The solution should avoid redundant tables or complex migration paths.

## Considered Options

### Option A – Hard-coding in code/schema (status quo)

- **Pros:** No additional effort, known implementation.
- **Cons:** No configurability, inconsistencies, increased testing and maintenance effort.

### Option B – Multiple configuration tables per area

- For example, separate entities for time defaults, balances, UI, etc.
- **Pros:** High granularity, theoretically explicit ownership per area.
- **Cons:** Fragmented maintenance, high overhead (multiple tables, services, authorization rules).

### Option C – Singleton entity `Customizing` with service layer (chosen)

- A single entity holds all global default values; `CustomizingService` caches the data and provides type-safe access.
- **Pros:** Single source of truth, easy service-based access, clear authorization, low code noise.
- **Cons:** Larger entity with many fields; content validation must occur in the service.

## Decision

We implement **Option C**. The `Customizing` entity is defined as a singleton in the data model (`db/data-model.cds`) and initially loaded via CSV (`db/data/io.nimble-Customizing.csv`). A new `CustomizingService` reads the record on service startup, caches it, and provides type-safe getters for time entry, user, balance, vacation/sick leave, and holiday API defaults. `TrackService.init()` calls `customizingService.initialize()` and then configures `DateUtils`. Business components (factories, commands, validators, services) obtain their defaults exclusively through `CustomizingService`.

## Consequences

**Positive**

- Single source of truth for all global defaults.
- Changes can be made without deployment (Fiori Elements object page for key users).
- Tests can mock defaults centrally; integration tests get deterministic seed data.
- `DateUtils` and `HolidayService` use configurable locale and API parameters.
- Authorization distinguishes between read access (all users) and maintenance (admins).

**Negative**

- Additional entity/fields increase maintenance effort (validation, documentation).
- Incorrect values in customizing can break system behavior; monitoring/validation are required.
- Migrating existing hard-coded defaults to the service required initial refactoring effort.

## References

- `db/data-model.cds` – definition of the `Customizing` entity
- `db/data/io.nimble-Customizing.csv` – initial seed data
- `srv/track-service/handler/services/CustomizingService.ts` – access & caching
- `srv/track-service/handler/container/ServiceContainer.ts` – wiring & authorization
- `srv/track-service/track-service.ts` – initialization & DateUtils configuration
- `srv/track-service/annotations/ui/customizing-ui.cds` – Fiori Elements object page
