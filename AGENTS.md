# AI Agent Guide

## Purpose and Scope

- Supports developers with tasks in a SAP CAP / Fiori Time-Tracking project.
- Provides context-sensitive knowledge about architecture, guidelines, and workflows from existing documentation.
- Targets autonomous or semi-autonomous agents that take on code, documentation, or analysis tasks.

## Project Essentials

- **Domain**: Time tracking with focus on project recording, balance calculation, holiday logic, and bulk generation of work days.
- **Architecture**: Five-layer Clean Architecture (Presentation, Application, Business Logic, Data, Infrastructure) with Command, Service, Repository, and Validator patterns; consistent Dependency Injection via `ServiceContainer`.
- **Technology Stack**: SAP Cloud Application Programming Model (Node.js), 100% TypeScript, UI5 ≥ 1.120, SQLite (Dev) / SAP HANA (Prod), Jest for tests, ESLint/Prettier.
- **Key Artifacts**: `README.md` (Executive Overview), `docs/ARCHITECTURE.md` (arc42 detail), `GETTING_STARTED.md`, ADRs under `docs/ADR/`.

## Agent Responsibilities

- Consult business and architecture context from `README.md`, `docs/ARCHITECTURE.md`, and ADRs before proposing changes.
- Place changes according to layer separation (e.g., Commands in Business Logic Layer, Handlers in Application Layer, Repositories in Data Layer).
- Maintain type safety (no `any`, prefer Interfaces/DTOs, respect CAP CDS definitions).
- Keep documentation in sync (document changes to APIs, behavior, or architecture in relevant Markdown files).
- Consider tests: respect existing Jest suites and add new tests for new business logic.

## MCP Servers and Tools

- **sap-docs**: Aggregated SAP documentation via HTTP server (ABAP, CAP, UI5, OpenUI5, SAP Community).
- **cap-js/mcp-server**: Access to CAP documentation, API references, and best practices for CAP Node.js/TypeScript.
- **@sap-ux/fiori-mcp-server**: Information on Fiori Elements, UI5 templates, annotations, and SAP Fiori Guidelines.
- **@ui5/mcp-server**: Deeper UI5 API references (Controls, MVC, i18n, Routing).

Additional local tools:

- `rg` for code and text search (faster than `grep`).
- `npm run watch` for CAP server & UI5 Development Preview.
- `npm test` for Jest suites; `npm run lint` if available.
- REST client files under `tests` for manual OData calls.

## Relevant Knowledge Sources

- **Architecture**: `docs/ARCHITECTURE.md` (building block insights, runtime flows, quality goals).
- **Service Behavior**: `srv/track-service/` (Handlers, Commands, Services).
- **Data Model**: `db/data-model.cds`, plus CSV master data in `db/data/`.
- **UI Layers**: `app/timetable` (Fiori Elements) and `app/timetracking` (Custom UI5 with TypeScript).
- **Integrations**: Holiday API, Attachments Plugin (`@cap-js/attachments`), Logging via Infrastructure Layer.

## Workflows for Agents

1. **Understand Requirements**: Identify requirements, affected layers, and quality goals.
2. **Research Context**: Check CDS models, existing Commands/Services/Handlers (e.g., using `rg`, project structure, ADRs).
3. **Create Plan**: Formulate a rough plan with steps for code, tests, and documentation before implementing changes.
4. **Implement**: Respect layer conventions, update DI container, maintain barrel exports, ensure type safety.
5. **Validate**: Run relevant tests (`npm test`) and optionally `npm run lint`; adjust auto-previews for UI changes.
6. **Document**: Update release notes, README, architecture additions, or ADRs if behavior or structure changes.
7. **Prepare Review**: Provide summary of changes, test results, and open points.

## Coding & Architecture Guidelines

- **TypeScript only**: Do not add JavaScript files; strictly type CAP handlers.
- **Separation of Concerns**: Encapsulate business logic in Commands/Services, not in Handlers.
- **Dependency Injection**: Register new components in `ServiceContainer`, realize unit tests via interfaces and mocks.
- **Validation & Error Handling**: Use existing validator pipeline, return errors as CAP `ServiceError` with codes/details.
- **Internationalization**: Check UI changes in `_i18n`; place new texts in `i18n.properties`.
- **Logging**: Use provided Logger utilities for structured logging, no `console.log`.
- **Performance**: Avoid long-running operations, consider caching utilities (see section 8.6 of architecture docs).

## Environment & Constraints

- Local runtime: Node.js ≥ 18, `npm install` for setup; never commit `.env` files.
- Database: SQLite locally (`db.sqlite`), migrations via `cds deploy`. In Prod, SAP HANA is planned – SQL statements must be compatible.
- Network access is restricted; use external dependencies only via permitted interfaces (e.g., holiday API).

## Domain-Specific Hints

- **Core Entities**: `TimeEntries`, `Projects`, `Users`, `ActivityTypes`, `EntryTypes`, `Customizing`.
- **Bulk Generation**: Strategies (`Monthly`, `Yearly`) combine API queries for holidays with entry generation.
- **Balance Logic**: Commands transform data into monthly and total balances; UI shows criticality via annotations.
- **Attachments**: `@cap-js/attachments` handles upload/download; verify consistency between backend and UI on changes.

## Quality Checklist Before Completion

- Relevant tests pass (or there is justification plus an open task).
- Code complies with layering, typing, and style guidelines.
- Documentation/ADR updated if architecture or process decisions are affected.
- Reviewers receive clear description of changes, tests, and open questions.

## Escalation & Collaboration

- Clarify unclear requirements or architecture questions with the Maintainer Team (Development Team) when in doubt.
- For larger decisions (e.g., new integration, pattern change), prepare a new ADR entry.
- Add open risks or technical debt to section 11 of `docs/ARCHITECTURE.md`.

## Rules for creation or modification of SAP Fiori elements apps

- When asked to create an SAP Fiori elements app check whether the user input can be interpreted as an application organized into one or more pages containing table data or forms, these can be translated into a SAP Fiori elements application, else ask the user for suitable input.
- The application typically starts with a List Report page showing the data of the base entity of the application in a table. Details of a specific table row are shown in the ObjectPage. This first Object Page is therefore based on the base entity of the application.
- An Object Page can contain one or more table sections based on to-many associations of its entity type. The details of a table section row can be shown in an another Object Page based on the associations target entity.
- The data model must be suitable for usage in a SAP Fiori elements frontend application. So there must be one main entity and one or more navigation properties to related entities.
- Each property of an entity must have a proper datatype.
- For all entities in the data model provide primary keys of type UUID.
- When creating sample data in CSV files, all primary keys and foreign keys MUST be in UUID format (e.g., `550e8400-e29b-41d4-a716-446655440001`).
- When generating or modifying the SAP Fiori elements application on top of the CAP service use the Fiori MCP server if available.
- When attempting to modify the SAP Fiori elements application like adding columns you must not use the screen personalization but instead modify the code of the project, before this first check whether an MCP server provides a suitable function.
- When previewing the SAP Fiori elements application use the most specific `npm run watch-*` script for the app in the `package.json`.

## Guidelines for UI5

Use the `get_guidelines` tool of the UI5 MCP server to retrieve the latest coding standards and best practices for UI5 development.
Other UI5 MCP Tools:

- create_ui5_app: Scaffolds a new UI5 application based on a set of templates.
- get_api_reference: Fetches and formats UI5 API documentation.
- get_guidelines: Provides access to UI5 development best practices.
- get_project_info: Extracts metadata and configuration from a UI5 project.
- get_version_info: Retrieves version information for the UI5 framework.
- run_ui5_linter: Integrates with @ui5/linter to analyze and report issues in UI5 code.

## Guidelines for CAP

- You MUST search for CDS definitions, like entities, fields and services (which include HTTP endpoints) with cds-mcp, only if it fails you MAY read \*.cds files in the project.
- You MUST search for CAP docs with cds-mcp EVERY TIME you create, modify CDS models or when using APIs or the `cds` CLI from CAP. Do NOT propose, suggest or make any changes without first checking it.
