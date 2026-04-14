# ADR 0013: CAP Attachments Plugin for Document Attachments

## Status

Accepted – Iteration 7 (Introduced Attachment Capability)

## Context and Problem Statement

For the TimeEntries object page, users should be able to upload and retrieve receipts, booking documents, or supplementary files. The solution must work both locally (SQLite) and in HANA Cloud, support versioning, and integrate into the Fiori Elements attachments section. At the same time, security, streaming, and metadata aspects must be implemented consistently across all environments.

## Decision Factors

- **Standards compliance:** Use official CAP extensions instead of custom implementations
- **Maintainability:** Minimal custom code for media handling, focus on business logic
- **Deployment flexibility:** Works locally (SQLite) and in productive HANA scenarios
- **Fiori Elements integration:** Attachment facet should work out of the box
- **Cost / infrastructure:** Avoid additional services or licenses unless strictly required

## Considered Options

### Option A – SAP CAP Attachments Plugin (`@cap-js/attachments`) **(chosen)**

- **Pros:**
  - Official CAP extension with auto-generated handlers, metadata storage, and streaming support
  - Works without additional services on both SQLite and HANA
  - Fiori Attachments facet can be connected directly, including toggle via customizing
- **Cons:**
  - Persists binary data in the database (no dedicated content repository)
  - Feature set is limited to standard use cases

### Option B – SAP Document Management Service Plugin (`@cap-js/sdm`)

- **Pros:**
  - Offloads binary data to a dedicated content store with versioning and sharing features
  - Suitable for large volumes and enterprise-wide document management
- **Cons:**
  - Requires a configured SAP Document Management Service (cost, setup, operation)
  - Higher complexity (authentication, destinations, lifecycle management)
  - Overkill for project-internal attachments in the MVP

### Option C – Custom implementation (Custom Media Entity / Storage Service)

- **Pros:**
  - Maximum control over storage location (e.g. S3, Azure Blob) and business rules
- **Cons:**
  - High development and maintenance effort (streaming, security, metadata, cleanup)
  - Risk of security and compliance gaps without tested standard components
  - Additional tests and documentation required

## Decision

We choose **Option A** and integrate the CAP Attachments Plugin. The `TimeEntries` entity was extended with a `Composition of many Attachments` (`db/attachments.cds`). The plugin provides all required handlers and OData endpoints, allowing upload/download directly from Fiori Elements. Visibility is controlled via the customizing field `hideAttachmentFacet`.

## Consequences

**Positive**

- Fast implementation without custom media handlers
- Consistent behavior in dev/prod, streaming supports large files
- Direct Fiori attachments integration, UI toggle via customizing

**Negative**

- Binary data is stored in the database, increasing storage requirements
- Advanced requirements (versioning, sharing) would need future extension via DMS or a custom solution

## References

- `package.json` – dependency `@cap-js/attachments`
- `db/attachments.cds` – composition extension for `TimeEntries`
- `srv/track-service/annotations/ui/timeentries-ui.cds` – attachment facet + hidden toggle
- `docs/ARCHITECTURE.md` – section [8.8 Document Attachments (Attachments Plugin)](#88-dokumentanhänge-attachments-plugin)
- `README.md` – section “Key Features” (document attachments)
