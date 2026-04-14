# ADR 0018: MTA Deployment for SAP BTP Cloud Foundry

## Status

Accepted – `mta.yaml` is available, deployment pipeline pending

## Context and Problem Statement

- The application should be operated in production on SAP Business Technology Platform (BTP) in the Cloud Foundry environment.
- In addition to the CAP service, other service instances (HANA HDI, Object Store, Malware Scanning, Application Logging) must be bound consistently.
- The Fiori frontends should be delivered via the SAP Application Frontend Service (Managed App Router + Static Hosting) without a dedicated app router module.
- A simple `cf push` with a manifest reaches its limits (no DB deployer build, no ordered service dependencies, no multi-module support).
- For CI/CD, reproducible packaging is needed that clearly separates build and deploy steps.

## Decision Factors

- **Cloud-native principles** – infrastructure dependencies should be described declaratively (Infrastructure-as-Code).
- **Multi-module support** – the CAP service (`gen/srv`) and HDI deployer (`gen/db`) must be built and bound in the correct order.
- **Service bindings** – attachments, malware scanner, logging, and HANA require identical names between build and deploy.
- **CI/CD capability** – the artifact (`.mtar`) should be easy to transport in pipelines (promotion to test/prod).
- **Standards compliance** – alignment with SAP best practices for BTP deployments (Multi-Target Applications).

## Considered Options

### Option A – `cf push` with `manifest.yaml`

- **Pros**: simple setup, fast local iteration.
- **Cons**: no build hooks, no multi-module packages, services must be created manually in advance, order risks for HDI deployment.

### Option B – `cf deploy` with `mta.yaml`

- **Pros**: declarative build/deploy flow, clear separation of modules and resources, service bindings included, supports CI/CD by default (MTAR artifact).
- **Cons**: requires an additional build step (`mbt build`), initially higher complexity.

### Option C – Kyma / Helm charts

- **Pros**: Kubernetes-native, expands scaling options.
- **Cons**: high migration effort, no immediate need, additional operational complexity.

## Decision

- We adopt **Option B – Multi-Target Application (MTA)** as the standard deployment strategy.
- `mta.yaml` describes three modules (`cap-fiori-timetracking-srv`, `cap-fiori-timetracking-db-deployer`, `cap-fiori-timetracking-app-deployer`) and resources (HANA HDI, Object Store, Malware Scanning, Application Logging, Application Frontend Service).
- Build hook `before-all` runs `npm ci` and `cds build --production`; `mbt build` generates a transportable `.mtar`.
- Deployments are performed via `cf deploy gen/mta.mtar`, keeping build and deploy clearly separated.

## Consequences

- **Positive**: cloud-native, reproducible deployments; service dependencies declared declaratively; HDI deployment automated; UI delivery through Managed App Router without self-hosting.
- **Negative/risks**: `mbt` tooling is an additional requirement in CI/CD; MTAR versioning must match the app version.
- **Follow-ups**:
  - integrate the MTA build into the release pipeline (link to ADR-0017).
  - keep service names in `mta.yaml` and `package.json → cds.requires` synchronized.
  - evaluate an optional Kyma deployment once requirements make it necessary.

## References

- `mta.yaml`
- `README.md` section “☁️ Cloud Deployment (SAP BTP)”
- `GETTING_STARTED.md` section “☁️ Deploy on SAP BTP (Cloud Foundry)”
- `docs/ARCHITECTURE.md` chapters 7.3 (deployment scenarios) and 9 (ADR)
