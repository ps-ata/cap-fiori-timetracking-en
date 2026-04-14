# ADR 0006: Modularization of CDS Annotations by Concerns

## Status

Accepted - Annotations Refactoring (Iteration 2)

## Context and Problem Statement

At the beginning of the project, UI annotations were placed directly in the service definition or in a monolithic annotations file. As requirements for UI5 configurations grew (Field Controls, Capabilities, Authorization), the annotations file became increasingly complex and difficult to maintain. Multiple developers working on the same annotation file led to frequent merge conflicts.

## Decision Factors

- Separation of Concerns: UI layout, Field Controls, Capabilities, and Authorization should be independently maintainable.
- Team Collaboration: Multiple developers should be able to work in parallel on different annotation areas without causing merge conflicts.
- Reusability: Common patterns (e.g., `@readonly`, `@mandatory`) should be defined centrally and applicable across all entities.
- Clarity: Entity-specific UI layouts should be in separate files for quick navigation.
- Tooling Compatibility: The structure must be compatible with Fiori Elements, SAP Business Application Studio, and VS Code Extension.

## Considered Options

### Option A - Monolithic Annotations File

- All annotations in one file `srv/track-service/annotations.cds`.
- Alphabetical or entity-based sorting.
- Advantage: Only one file to maintain.
- Disadvantage: Becomes quickly unclear (>1000 lines), difficult to navigate, high likelihood of merge conflicts.

### Option B - Separation by Entity (flat)

- One file per entity: `users-annotations.cds`, `projects-annotations.cds`, `timeentries-annotations.cds`.
- All annotations for an entity in one file.
- Advantage: Entity-focused structure.
- Disadvantage: Duplication of common patterns, difficult reuse of Field Controls and Capabilities.

### Option C - Two-Level Structure: common/ and ui/

- `common/` - Shared concerns across all entities: `labels.cds`, `field-controls.cds`, `capabilities.cds`, `value-helps.cds`, `authorization.cds`, `actions.cds`.
- `ui/` - Entity-specific UI layouts: `users-ui.cds`, `projects-ui.cds`, `activities-ui.cds`, `timeentries-ui.cds`, `balance-ui.cds`.
- Master import in `srv/track-service/annotations.cds` includes all files.
- Advantage: Clear separation, high reusability, team collaboration friendly.
- Disadvantage: Higher number of files, initial learning curve for new developers.

## Decision

We choose **Option C** - the two-level structure with `common/` and `ui/`. The structure is organized as follows:

```
srv/track-service/annotations/
├── annotations.cds           # Master Import (imports all files)
├── common/                   # Shared Concerns
│   ├── labels.cds            # @Common.Text, @Common.TextArrangement, Titles
│   ├── field-controls.cds    # @readonly, @mandatory, @UI.Hidden
│   ├── capabilities.cds      # @Capabilities (InsertRestrictions, UpdateRestrictions)
│   ├── value-helps.cds       # @Common.ValueList (Dropdowns, F4-Helps)
│   ├── authorization.cds     # @restrict (Security, Roles)
│   └── actions.cds           # @Common.SideEffects, @Core.OperationAvailable
└── ui/                       # Entity-specific UI Layouts
    ├── users-ui.cds          # Users UI (LineItem, SelectionFields, HeaderInfo)
    ├── projects-ui.cds       # Projects UI Layout
    ├── activities-ui.cds     # ActivityTypes UI Layout
    ├── timeentries-ui.cds    # TimeEntries UI Layout (Main entity)
    └── balance-ui.cds        # MonthlyBalances UI Layout
```

The master file `annotations.cds` imports all files with `using from './annotations/common/...'` and `using from './annotations/ui/...'`, providing a central entry point for all annotations and ensuring consistency.

## Consequences

### Positive

- **Separation of Concerns**: Field Controls (`@readonly`, `@mandatory`) are defined centrally in `common/field-controls.cds` and applied across all entities. Changes to Field Controls affect all entities uniformly without code duplication.
- **Team Collaboration**: UI experts work on `ui/` files, security experts on `authorization.cds`, without blocking each other.
- **Reusability**: Common patterns like `@Common.Text` for associations are defined once and applied everywhere.
- **Quick Navigation**: Entity-specific UI layouts are in separate files, e.g., `timeentries-ui.cds` for all TimeEntries UI configurations.
- **IDE Support**: VS Code and SAP Business Application Studio can navigate the structure well and provide precise auto-completion suggestions.

### Negative

- **Higher Number of Files**: 11 files instead of 1 (6 in `common/`, 5 in `ui/`). New developers must first understand the structure.
- **Initial Learning Curve**: Developers must know where to maintain which annotation type (e.g., `@readonly` in `field-controls.cds`, UI layout in `ui/*.cds`).
- **Master Import Required**: The `annotations.cds` must be updated for new files (alternatively, wildcard import, but worse for explicit control).

### Trade-offs

We accept the higher number of files in favor of maintainability and team collaboration. The initial learning curve is mitigated by clear documentation in the master file and in `.github/copilot-instructions.md`.

## References

- `srv/track-service/annotations.cds` - Master import with structure documentation
- `srv/track-service/annotations/common/field-controls.cds` - Example of common patterns
- `srv/track-service/annotations/ui/timeentries-ui.cds` - Example of entity-specific UI layouts
- `.github/copilot-instructions.md` - Annotations structure in AI Development Guide

## Developer Notes

- **Add Field Controls**: Annotate in `common/field-controls.cds`
- **Adjust UI Layout**: Modify in `ui/<entity>-ui.cds`
- **Define Value Helps**: Define in `common/value-helps.cds`
- **New Annotation File**: Register in master import `annotations.cds` with `using from './annotations/...'`