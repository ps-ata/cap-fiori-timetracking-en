# ADR 0020: Holiday API Integration via BTP Destination + Connectivity

## Status

Accepted – November 2025

## Context and Problem Statement

The holiday API (feiertage-api.de) is needed for automatic annual generation of time entries to correctly recognize German holidays and mark them as non-working days. Locally, access is performed via direct HTTP call using `fetch()`, while production integration should be done via SAP BTP Destination + Connectivity Service.

The central question was: how do we integrate an external REST API both in local development and in the BTP production environment without storing credentials in code, while still providing a simple developer experience?

## Decision Factors

- Security: no hardcoded URLs or credentials in code or repository
- Ops-friendly: URL changes possible without code deployment
- Developer experience: local development works without BTP setup
- Cloud-native: use SAP BTP standard services (Destination, Connectivity)
- Proxy support: corporate firewalls and on-premise integration via Connectivity Service
- Audit trail: logging of all external API calls in BTP

## Considered Options

### Option A – direct fetch everywhere

- direct HTTP call via `fetch()` in all environments
- URL configuration via environment variables (`.env`)
- no BTP services required

### Option B – CAP remote services

- define the holiday API as a CAP remote service with OData metadata
- use `cds.requires` for service binding
- integrate via CAP standard patterns

### Option C – hybrid approach with Destination + Connectivity (chosen)

- local: direct HTTP via `fetch()` with `HOLIDAY_API_BASE_URL` from `.env`
- BTP: destination-based via `@sap-cloud-sdk/connectivity` and `@sap-cloud-sdk/http-client`
- environment detection via `NODE_ENV` and `VCAP_SERVICES`

## Decision

We choose **Option C** – a hybrid approach with two code paths in `HolidayService`:

```typescript
async getHolidays(year: number, stateCode: string): Promise<Map<string, Holiday>> {
  if (this.isProduction()) {
    return await this.fetchViaDestination(year, stateCode);
  } else {
    return await this.fetchDirectly(year, stateCode);
  }
}

private isProduction(): boolean {
  return process.env.NODE_ENV === 'production' || !!process.env.VCAP_SERVICES;
}
```

### Implementation

#### Local (Development)

- direct HTTP call via `fetch()` with configured base URL
- URL from `.env`: `HOLIDAY_API_BASE_URL=https://feiertage-api.de`
- timeout: 5 seconds via `AbortSignal.timeout(5000)`

#### BTP (Production)

- destination `holiday-api` defined in `mta.yaml` with:
  - URL: `https://feiertage-api.de`
  - ProxyType: `Internet`
  - Authentication: `NoAuthentication`
- HTTP calls via `@sap-cloud-sdk/http-client.executeHttpRequest()`
- automatic routing through Connectivity Service

#### Cache strategy

- cache per year + federal state (key: `${year}-${stateCode}`)
- lifetime: service runtime (holidays are static)
- invalidation: on server restart

#### Error handling

- graceful degradation: returns an empty map on API errors
- yearly generation works without holidays as well (marks all weekdays as work entries)
- structured logging via `logger.serviceCall()` and `logger.error()`

## Consequences

### Positive

- **Security**: no credentials in code – destination is managed in the BTP cockpit
- **Ops-friendly**: URL changes without deployment (destination update in cockpit only)
- **Developer experience**: local development with `npm run watch` works without BTP setup
- **Proxy support**: corporate firewalls and on-premise scenarios via Connectivity Service
- **Audit trail**: all API calls are recorded in BTP application logs
- **Testability**: unit tests can cover both code paths via environment variables

### Negative

- **Complexity**: two code paths (direct + destination) increase maintenance effort
- **Testing**: mocking of `executeHttpRequest()` is required for unit tests
- **Dependencies**: additional NPM packages (`@sap-cloud-sdk/connectivity`, `@sap-cloud-sdk/http-client`)
- **Operational cost**: BTP Destination + Connectivity services (lite plan free, but service overhead)

### Neutral

- **Caching**: cache strategy is independent of the fetch method
- **API stability**: holiday API is public and stable (available since 2015)

## Alternatives and why rejected

### Option A rejected

- ❌ no support for corporate proxy/firewall
- ❌ credentials/URLs in code or environment variables (not cloud-native)
- ❌ no audit logs in BTP

### Option B rejected

- ❌ feiertage-api.de is a REST API without OData metadata
- ❌ CAP remote services are primarily designed for SAP OData services (S/4HANA, SuccessFactors)
- ❌ overhead for simple GET requests without a complex entity model

## References

- `srv/track-service/handler/services/HolidayService.ts` – hybrid implementation
- `mta.yaml` – destination configuration under `resources.cap-fiori-timetracking-destination`
- `tests/track-service.test.js` – integration tests for Holiday API (describe block: `TrackService - HolidayService Integration`)
- `.env.example` – `HOLIDAY_API_BASE_URL` environment variable
- [SAP Cloud SDK Connectivity Docs](https://sap.github.io/cloud-sdk/docs/js/features/connectivity/destinations)
- [Feiertage-API documentation](https://feiertage-api.de)

## Extension possibilities

- **Rate limiting**: implement a rate limit tracker for API calls
- **Fallback to local data**: JSON file with German holidays 2020-2030 as offline fallback
- **Multi-country support**: expand to other countries (currently only Germany)
- **Custom holiday management**: admin UI for manual holiday maintenance (company-specific)
