# ADR 0005: Dual UI5 Strategy for Time Tracking

## Status

Accepted - Extension for dashboard requirements

## Context and Problem Statement

The project should offer both a fast Fiori Elements list for business users and a visually rich dashboard for self-service. A single app type did not cover both use cases: the List Report app provides quick CRUD processes, while planning functions (calendar, KPIs) require custom controls.

## Decision Factors

- Reuse the same TrackService (`/odata/v4/track/`) for both clients.
- Share OData annotations while allowing different presentations.
- Ability to integrate custom UI5 controls (SinglePlanningCalendar).
- Separate build configuration so generator updates can be applied safely.

## Considered Options

### Option A - Single Fiori Elements app with extensions

- Use extension points and side effects for special cases.
- Complex custom controls would need to be embedded in FE.

### Option B - Two specialized UI5 apps on shared service basis

- `app/timetable` as Fiori Elements List Report with annotations (`app/timetable/annotations.cds`).
- `app/timetracking` as custom TypeScript app with own controllers (`app/timetracking/webapp/controller/*.ts`).
- Shared service definition via `app/services.cds` and the same CAP service.

## Decision

We implement Option B. Both apps live as workspaces, share the backend OData, and use specialized UI functionality. Annotations are maintained per app, keeping FE-specific settings separate from the custom frontend. Deployment can deliver both bundles in parallel.

## Consequences

- Positive: Business users get a ready-to-use FE list, developers can design UI5 features freely in the dashboard.
- Positive: Annotations in `app/timetable` and `srv/track-service/annotations/ui` can be optimized without regard to custom views.
- Negative: Two build pipelines (ui5.yaml) must be maintained and tested.
- Negative: Shared assets (i18n) must be consciously synchronized.

## References

- `app/timetable/`
- `app/timetracking/`
- `app/services.cds`
- `srv/track-service/annotations/ui/timeentries-ui.cds`
- `srv/track-service/track-service.ts`
