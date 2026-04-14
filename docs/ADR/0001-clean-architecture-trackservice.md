# ADR 0001: Clean Architecture Layer for TrackService

## Status

Accepted - Project start (historical)

## Context and Problem Statement

The backend had to support complex time management rules, external holiday data, and multiple UI5 clients from the beginning. The standard CAP practice of defining event handlers directly in the service led early prototypes to strong dependencies between data access, validation, and orchestration. We needed a structure that ensures testability, reuse, and clear responsibilities.

## Decision Factors

- Consistent separation of infrastructure, application, and domain logic (Clean Architecture).
- Replaceability of implementations for repositories, services, and commands without code duplication.
- Ability to register handlers in a controlled way and later validate or test them automatically.
- Simple observability through centralized initialization and logging.

## Considered Options

### Option A - CAP standard with directly registered handlers

- Handlers are distributed across the service using `this.before(...)`.
- Dependencies are wired through direct imports.

### Option B - Custom Clean Architecture layer with container and registry

- ServiceContainer constructs repositories, services, validators, strategies, factories, and commands (`srv/track-service/handler/container/ServiceContainer.ts`).
- HandlerRegistry and HandlerSetup register handlers in grouped, traceable units (`srv/track-service/handler/registry/HandlerRegistry.ts`, `srv/track-service/handler/setup/HandlerSetup.ts`).
- `srv/track-service/track-service.ts` orchestrates the setup before `super.init()`.

## Decision

We choose Option B. The TrackService first initializes the ServiceContainer and then uses HandlerSetup to attach all handler groups. This keeps the ApplicationService slim, and new components can be integrated through the container without adjusting callers.

## Consequences

- Positive: Clean layer separation, consistent logging via `logger.service*`, easy reuse in tests.
- Positive: New handler groups can be activated via `.with...Handlers()` without changing existing registration.
- Negative: Higher initial effort because all dependencies must be maintained in the container.
- Negative: Developers must become familiar with the fluent setup before they can handle new events.

## References

- `srv/track-service/track-service.ts`
- `srv/track-service/handler/container/ServiceContainer.ts`
- `srv/track-service/handler/setup/HandlerSetup.ts`
- `srv/track-service/handler/registry/HandlerRegistry.ts`
- `srv/track-service/handler/index.ts`
