# ADR 0002: Command Pattern for Business Logic

## Status

Accepted - Early service consolidation

## Context and Problem Statement

The TimeEntry logic includes validations, time calculations, dependencies on multiple repositories, and bound actions (e.g. Recalculate). Early iterations with logic in handlers led to methods that were hard to test and duplicated code for CREATE and UPDATE. We needed a way to store business rules reusable and transactionally consistent.

## Decision Factors

- Reuse the same rule sets for handlers, actions, and future automations.
- Explicitly document which dependencies an operation requires.
- Simple logging hooks per business transaction (`logger.command*`).
- Testability through pure classes without CAP-specific binding.

## Considered Options

### Option A - Logic directly in handlers or services

- Handlers call repositories and utilities directly.
- Bound actions must reimplement their rules.

### Option B - Separate command layer for each operation

- Command classes encapsulate execution (`srv/track-service/handler/commands/**`).
- Handlers only pass request and transaction (`srv/track-service/handler/handlers/TimeEntryHandlers.ts`).
- Dependencies are injected by the ServiceContainer.

## Decision

We choose Option B. Each business transaction gets a command class that receives all dependencies in the constructor. Handlers remain limited to request preparation and error handling. Bound actions like `handleRecalculate` use the same commands, avoiding duplicate implementations.

## Consequences

- Positive: Commands can be unit-tested in isolation since they only need `Transaction` and plain objects.
- Positive: Logging and error handling follow a consistent structure via `logger.commandStart/End`.
- Positive: New operations (e.g. `GetVacationBalance`) can be quickly attached via commands without modifying handler logic.
- Negative: More files and boilerplate per use case.
- Negative: Developers must keep dependencies in the ServiceContainer up to date or commands will fail at runtime.

## References

- `srv/track-service/handler/commands/time-entry/CreateTimeEntryCommand.ts`
- `srv/track-service/handler/commands/time-entry/UpdateTimeEntryCommand.ts`
- `srv/track-service/handler/commands/balance/GetMonthlyBalanceCommand.ts`
- `srv/track-service/handler/handlers/TimeEntryHandlers.ts`
- `srv/track-service/handler/container/ServiceContainer.ts`
