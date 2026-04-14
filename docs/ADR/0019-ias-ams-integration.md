# ADR 0019: Productive Authentication with IAS & Authorization Management Service

## Status

Accepted – implementation completed in the repository, BTP configuration pending in the subaccount.

## Context and Problem Statement

- Time tracking should run on SAP BTP Cloud Foundry and be delivered via SAP Build Work Zone Standard Edition.
- So far we used `auth.kind = mocked` locally (ADR-0010) and had no production-ready identity and role model.
- Requirements: SSO via IAS, role-based approvals for time entries, prepared attributes for future policy rules (project, federal state).
- Stakeholders: end users (time tracking), team leads (approval), admins (master data), security & compliance.

## Decision Factors

- **Security & compliance:** enterprise-wide identities, centralized role transport, auditability.
- **Work Zone integration:** AFS requires IAS-/XSUAA-compatible JWTs; Fiori feature toggles read shell roles.
- **Attribute-based policies:** AMS should eventually drive project/location filters.
- **Developer experience:** local mock users must remain quick to use.

## Considered Options

### Option A – pure XSUAA setup

- XSUAA provides OAuth2 tokens and role templates (`xs-security.json`).
- Pros: proven, low additional complexity, direct CF integration.
- Cons: no attribute policy layer, additional coupling for SAML/corporate IdP, later migration to AMS required.

### Option B – IAS + AMS (chosen)

- IAS handles AuthN, AMS policies provide attributes; XSUAA remains a fallback via `xsuaa-cross-consumption`.
- Pros: unified identity provider landscape, prepared attributes (`db/ams-attributes.cds`), automatic DCL deployments (`ams/dcl/basePolicies.dcl`).
- Cons: two new CF services, additional deployment steps, service keys required for policy deployment.

## Decision

We choose **Option B**. The implementation includes:

- `package.json`: default `cds.requires.auth = "ias"`, fallback `xsuaa = true`, mock users with roles `TimeTrackingUser/Approver/Admin`.
- `xs-security.json`: scopes & role templates for the three production roles.
- `db/ams-attributes.cds`: mapping of user, project, and status information (including `ProjectNumber`) to AMS attributes.
- `ams/dcl/basePolicies.dcl`: base AMS policies deployed by `cap-fiori-timetracking-ams-policies-deployer`.
- `mta.yaml`: bindings for `cap-fiori-timetracking-ias` and `cap-fiori-timetracking-ams`, including certificate credentials.

## Consequences

- **Positive:** unified AuthN/AuthZ stack for Work Zone; attributes available to business logic and AMS policies; no code swap between dev & prod.
- **Negative:** additional services to provision (cost-relevant), deployment fails if AMS or IAS bindings are missing; local tests still require mock role maintenance.
- **Follow-ups:** refine AMS policies per business area, create role collections in the subaccount, automate end-to-end tests with real JWTs.

## References

- `package.json`, `xs-security.json`
- `mta.yaml` (modules & resources for IAS/AMS)
- `db/ams-attributes.cds`, `ams/dcl/basePolicies.dcl`
- `docs/ARCHITECTURE.md` section 7.4
