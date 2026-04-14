# ADR 0014: Swagger UI Preview for TrackService

## Status

Accepted – Iteration 10 (Developer Experience Improvements)

## Context and Problem Statement

The TrackService OData APIs are used by both the Fiori frontend and integration partners. Until now, there was no easily accessible, always-current API documentation. Test data could only be traced via REST Client files or manual requests, which increased onboarding time for new team members and was documented as technical debt (TD-3).

## Decision Factors

- **Freshness:** API description must be generated automatically from the CDS models (no manual maintenance).
- **Developer productivity:** New team members should be able to explore endpoints quickly.
- **Maintainability:** The solution should integrate into the CAP server without additional custom code.
- **Scope control:** Provide only in development so unauthenticated API docs are not exposed in production.

## Considered Options

### Option A – `cds-swagger-ui-express` plugin **(chosen)**

- **Pros:**
  - Seamless integration into the CAP server via the `serving` hook.
  - Generates OpenAPI specifications on the fly via `@cap-js/openapi`.
  - Provides an interactive Swagger UI including diagrams (`/$api-docs/...`).
  - Zero-config: default settings are sufficient for local development.
- **Cons:**
  - Runs only during development; production deployment would need separate protection.

### Option B – manual OpenAPI generation via `cds compile`

- **Pros:**
  - Produces a static OpenAPI JSON that can be versioned.
  - No additional runtime dependencies.
- **Cons:**
  - No UI; requires extra tools (Postman, Stoplight, etc.).
  - Requires scripts/automation to keep specs current.
  - No fast access for non-developers.

### Option C – maintain Postman/REST Client collections

- **Pros:**
  - Low barrier for targeted test cases.
- **Cons:**
  - Not a full API directory; high manual maintenance effort.
  - Not self-documenting, no automatic model synchronization.

## Decision

We integrate Option A (`cds-swagger-ui-express`) as a dev dependency. On startup via `npm run watch`, the plugin registers automatically and exposes Swagger UI at `http://localhost:4004/$api-docs/odata/v4/track/`. The underlying OpenAPI specification is also available as JSON at `/$api-docs/odata/v4/track/openapi.json`. This resolves the documented technical debt TD-3 and improves onboarding for new developers as well as collaboration with integration partners.

## Consequences

**Positive**

- Interactive API documentation with no additional maintenance overhead.
- Consistency between the CDS model and the exposed specification.
- Faster alignment with UI and integration teams thanks to Swagger UI.

**Negative**

- No production deployment yet; secure Swagger hosting would be needed for external consumers.
- Additional dev dependency that must be installed locally.

## References

- `package.json` – dev dependency `cds-swagger-ui-express`
- `README.md` – section “Highlights” & Quick Start (Swagger UI note)
- `GETTING_STARTED.md` – chapter “Swagger UI & OpenAPI (development only)”
- `docs/ARCHITECTURE.md` – section [8.9 OpenAPI & Swagger UI](../ARCHITECTURE.md#89-openapi--swagger-ui) & updated technical debt TD-3
