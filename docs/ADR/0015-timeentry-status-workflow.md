# ADR 0015: TimeEntry Status Workflow with CodeList & Customizing

## Status

Accepted – Iteration 11 (Status Lifecycle & Closing Flow)

## Context and Problem Statement

Until now, TimeEntries could be changed arbitrarily. For billing processes and audit trails, however, we need an explicit status lifecycle:

- **Open (O):** newly recorded or generated entries
- **Processed (P):** entries that have been manually adjusted
- **Done (D):** content-complete but not yet released for billing
- **Released (R):** final release (read-only)

The solution must…

- protect both **UI** and **backend** from unauthorized status transitions,
- allow tenants to maintain **custom codes**,
- support **multi-select** for Done/Release (via sequential bound actions),
- define allowed transitions based on master data.

## Decision Factors

- **Configurability:** status codes must not be hardwired in code (customizing requirement).
- **Consistency across layers:** data model, business logic, and UI must use identical status information.
- **Governance:** final statuses (`Released`) must be locked server-side.
- **UX:** the UI should only offer allowed actions (`allowDoneAction`, `allowReleaseAction`).

## Considered Options

### Option A – CodeList `TimeEntryStatuses` + Customizing Defaults + Actions **(chosen)**

- _Pros:_
  - Master data captures transitions (`from_code`, `to_code`) and action availability.
  - Customizing provides default codes for Open/Processed/Done/Released.
  - New bound actions (`markTimeEntryDone`, `releaseTimeEntry`) enable status transitions with server validation directly on the affected entries.
  - UI annotations can access `status.allowDoneAction` / `allowReleaseAction` directly and control operations.
- _Cons:_
  - Additional entity + CSV seeds need maintenance.
  - More commands/handlers increase service container complexity.

### Option B – Hard-coded TimeEntry (enum/string switch)

- _Pros:_
  - No additional table needed.
  - Lower initial effort.
- _Cons:_
  - No customizing possible; status transitions only in code.
  - UI cannot derive availability from master data.
  - Extensions (additional statuses, tenant-specific transitions) become costly.

## Decision

Option A delivers the required level of configurability while still enabling centralized business rule control. `TimeEntryStatuses` extends the data model, `Customizing` holds the current codes, and two commands orchestrate each entry’s status transitions with repository validation. `TimeEntryHandlers` block changes once `Released` is reached. UI annotations use master data to enable or disable actions contextually. This achieves business traceability, consistent locking logic, and clean extensibility.

## Consequences

**Positive**

- The status lifecycle is transparently represented in master data and documentation.
- Customizing can replace codes without code changes.
- The UI only shows allowed actions and uses value helps for status transitions.
- The backend guarantees that `Released` entries remain immutable.

**Negative**

- Higher initial effort: additional commands, tests, documentation.
- Administrators must keep master data (CodeList + Customizing) in sync.

## References

- `db/data-model.cds` – `TimeEntryStatuses` entity + `TimeEntries.status`.
- `db/data/io.nimble-TimeEntryStatuses.csv` & `db/data/io.nimble-Customizing.csv`.
- `srv/track-service/track-service.cds` – new actions, projection & field control.
- `srv/track-service/handler/commands/time-entry/*` – new status commands.
- `srv/track-service/annotations` – UI line items, field groups, action groups.
- `tests/track-service.test.js` – integration tests for status transitions & release.
- `docs/ARCHITECTURE.md` – sections 5.3/5.4 (status model & commands).
