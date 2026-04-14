# ADR 0011: Test Strategy for CAP Services with Jest and REST Client

## Status

Accepted - Test infrastructure setup (Iteration 6)

## Context and Problem Statement

The project required an automated test strategy for CAP services to meet the following requirements:

1. **Automated tests**: Local development and CI/CD pipelines must run tests automatically.
2. **OData/HTTP API testing**: TimeEntries CRUD, Actions (Generation, Recalculate), Functions (balance calculations) must be tested.
3. **Business logic validation**: Commands, validators, factories, and services need unit/integration tests.
4. **Multi-user scenarios**: Tests with different mock users (Max, Erika) for user-specific features.
5. **Fast feedback**: Developers should get immediate feedback on broken functionality.
6. **CI/CD integration**: Tests must run in GitHub Actions and fail the build on failures.
7. **Manual ad-hoc tests**: Developers should be able to execute individual HTTP requests quickly during development.

Without a test strategy, the following problems occurred:

- **Manual regression testing**: Every change had to be tested manually via UI clicks or Postman requests.
- **Missing coverage**: No visibility into which code areas were tested.
- **Slow bug detection**: Bugs were discovered late in production or via user feedback.
- **Difficult refactorings**: Business logic changes required expensive manual tests.

## Decision Factors

- **CAP-native integration**: The test framework should use CAP's `cds.test()` for seamless integration.
- **TypeScript support**: Tests should be written in TypeScript and use type-safe APIs.
- **Test runner flexibility**: Both Jest and Mocha should be supported.
- **Authentication integration**: Mock users from `package.json` should be reusable in tests.
- **HTTP + service API testing**: Both layers (HTTP OData and programmatic service API) should be testable.
- **Easy setup**: Minimal boilerplate, fast onboarding for new developers.
- **IDE integration**: VS Code REST Client for manual tests without Postman.
- **CI/CD ready**: Tests should run in GitHub Actions without extra configuration.

## Considered Options

### Option A - Manual Tests with Postman Only

- Manual HTTP requests in Postman collections.
- Pros: No test framework setup, immediately usable.
- Cons: No automation, no CI/CD, no coverage, difficult team collaboration (Postman collections must be shared).

### Option B - Supertest without CAP Integration

- HTTP tests with Supertest directly against the Express server.
- Pros: Flexible, many community examples.
- Cons: No CAP-native integration, no `cds.test()`, manual mock user handling, more complex setup.

### Option C - CAP Native Testing with cds.test() + Jest

- Use CAP's `cds.test()` for HTTP and service API tests.
- Jest as test runner (alternatively Mocha).
- Chai for assertions (compatible with Jest and Mocha).
- Mock users from `package.json` for authentication.
- Pros: CAP-native, TypeScript support, easy setup, CI/CD-ready, flexible test runner choice.
- Cons: Additional dependencies (Jest, Chai, @types).

### Option D - REST Client for Manual Tests + Jest for Automation (Hybrid)

- **Automated tests**: Jest + `cds.test()` for CI/CD and regression tests.
- **Manual tests**: VS Code REST Client (`.http` files) for ad-hoc tests during development.
- Pros: Best of both worlds - automation + fast manual tests, no Postman dependency.
- Cons: Two test systems to maintain (but with low overlap).

## Decision

We choose **Option D** - a hybrid approach with Jest for automated tests and REST Client for manual tests. This provides an optimal balance between automation and developer experience.

### 1. Automated Tests with Jest + cds.test()

#### Test Structure

```
test/
├── track-service.test.ts    # Integration tests for TrackService
├── commands/                # Unit tests for commands (optional)
├── validators/              # Unit tests for validators (optional)
└── services/                # Unit tests for services (optional)
```

#### Jest configuration (`jest.config.js`)

```javascript
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  testMatch: ['**/test/**/*.test.ts'],
  collectCoverageFrom: ['srv/**/*.ts', '!srv/**/*.d.ts', '!srv/**/index.ts'],
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html'],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70,
    },
  },
  moduleNameMapper: {
    '^#cds-models/(.*)$': '<rootDir>/@cds-models/$1',
  },
  testTimeout: 30000, // CAP Server-Start kann länger dauern
};
```

#### Beispiel-Test (`test/track-service.test.ts`)

```typescript
import cds from '@sap/cds';

describe('TrackService - TimeEntries CRUD', () => {
  const { GET, POST, PATCH, DELETE, expect } = cds.test('../', '--in-memory');
  const maxUser = { auth: { username: 'max.mustermann@test.de', password: 'max' } };

  it('should create a new work time entry', async () => {
    const { data, status } = await POST(
      '/odata/v4/track/TimeEntries',
      {
        user_ID: 'max.mustermann@test.de',
        workDate: '2025-10-14',
        entryType_code: 'W',
        startTime: '08:00:00',
        endTime: '16:30:00',
        breakMin: 30,
      },
      maxUser,
    );

    expect(status).to.equal(201);
    expect(data).to.have.property('ID');
    expect(data.durationHoursNet).to.equal(8.0);
  });
});
```

#### npm Scripts (`package.json`)

```json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

#### Dependencies (werden vom User installiert)

```bash
npm add -D jest ts-jest @types/jest chai@4 chai-as-promised@7 chai-subset
```

### 2. Manuelle Tests mit REST Client

#### REST Client Files

```
test/
└── track-service.http    # HTTP Requests für manuelle Tests
```

#### Beispiel-Request (`test/track-service.http`)

```http
@server = http://localhost:4004
@odata = {{server}}/odata/v4/track
@maxAuth = Authorization: Basic max.mustermann@test.de:max

### Get All TimeEntries
GET {{odata}}/TimeEntries
{{maxAuth}}

### Create Work Time Entry
POST {{odata}}/TimeEntries
Content-Type: application/json
{{maxAuth}}

{
  "user_ID": "max.mustermann@test.de",
  "workDate": "2025-10-14",
  "entryType_code": "W",
  "startTime": "08:00:00",
  "endTime": "16:30:00",
  "breakMin": 30
}
```

#### VS Code Extension

- **Name**: REST Client by Huachao Mao
- **Installation**: VS Code Extensions → "REST Client" suchen
- **Usage**: `Send Request` über jeder `###` Zeile klicken

### 3. CI/CD Integration mit GitHub Actions

#### Workflow (`.github/workflows/test.yml`)

```yaml
name: Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci

      - name: Build TypeScript
        run: npm run build

      - name: Run Tests
        run: npm test

      - name: Upload Coverage
        uses: codecov/codecov-action@v3
        if: success()
        with:
          files: ./coverage/lcov.info
          flags: unittests
          name: cap-fiori-timetracking-coverage
```

## Consequences

### Positive

- **Automated regression tests**: All CRUD operations, actions, and functions are tested on every commit.
- **Fast feedback**: Developers get immediate feedback on broken functionality (local + CI/CD).
- **Code coverage**: Jest generates coverage reports that show which code areas are tested.
- **CAP-native integration**: `cds.test()` uses CAP's internal test utilities for seamless integration.
- **TypeScript type safety**: Tests use typed APIs from `#cds-models/*`, preventing compile-time errors.
- **Mock user reuse**: Existing mock users from `package.json` are reused in tests.
- **Flexible test runner choice**: Jest or Mocha can be used (portable tests with Chai).
- **Manual tests without Postman**: REST Client in VS Code enables ad-hoc tests without external tools.
- **CI/CD integration**: GitHub Actions runs tests automatically and fails the build on failures.
- **Team collaboration**: `.http` files can be shared in the Git repository (vs. Postman collections).

### Negative

- **Additional dependencies**: Jest, Chai, @types increase `node_modules` size (approx. 50 MB).
- **Initial learning curve**: Developers must learn the `cds.test()` API and Chai assertions.
- **Test maintenance**: Tests must be updated when the API changes (trade-off for automation).
- **Longer CI/CD runtime**: Tests increase build time by approx. 30-60 seconds.
- **Two test systems**: REST Client and Jest must be maintained in parallel (but with low overlap - manual vs. automated tests).

### Trade-offs

We accept the additional dependencies and test maintenance in favor of automation, fast feedback, and code coverage. The initial learning curve is offset by sample tests and ADR documentation.

## Test Pyramid for the Project

The project follows a classic test pyramid with a focus on integration tests:

### 1. Integration Tests (70%) - **Primarily with Jest**

- **Purpose**: Test OData HTTP APIs, actions, functions
- **Framework**: Jest + `cds.test()`
- **Location**: `test/track-service.test.ts`
- **Coverage**: TimeEntries CRUD, generation, balance calculations
- **Example**: Create TimeEntry → validation → calculation → persist

### 2. Unit Tests (20%) - **Optional**

- **Purpose**: Test isolated business logic (commands, validators, factories)
- **Framework**: Jest with mocks
- **Location**: `test/commands/*.test.ts`, `test/validators/*.test.ts`
- **Coverage**: time calculations, validation rules
- **Example**: TimeCalculationService.calculateWorkingHours()

### 3. E2E Tests (10%) - **OPA5 (already in place)**

- **Purpose**: Test UI flows
- **Framework**: OPA5 (Fiori Elements)
- **Location**: `app/timetable/webapp/test/integration/`
- **Coverage**: UI navigation, create flow, filter
- **Example**: User opens app → creates entry → saves → sees entry in list

## Developer Workflow

### Local Development

1. **Write tests**: Add new tests in `test/`
2. **Run tests**: `npm test` (all tests) or `npm run test:watch` (watch mode)
3. **Check coverage**: `npm run test:coverage` → open `coverage/index.html`
4. **Manual tests**: Open `test/track-service.http` → click "Send Request"

### CI/CD

1. **Push to GitHub**: Tests run automatically in GitHub Actions
2. **Pull request**: Tests must be green before the PR can be merged
3. **Coverage report**: Codecov shows coverage changes in the PR comment

## Example Tests in the Project

The project includes comprehensive example tests in `test/track-service.test.ts`:

### CRUD-Tests

- ✅ Create Work Time Entry
- ✅ Create Vacation Entry
- ✅ Reject Duplicate Entry (Validation)
- ✅ Reject Invalid Time Range (Validation)
- ✅ Read TimeEntries with Filters
- ✅ Update TimeEntry und Recalculate
- ✅ Delete TimeEntry

### Action-Tests

- ✅ Generate Monthly Entries
- ✅ Generate Yearly Entries
- ✅ Get Default Parameters
- ✅ Recalculate TimeEntry (Bound Action)

### Function-Tests

- ✅ Get Monthly Balance
- ✅ Get Current Balance
- ✅ Get Recent Balances
- ✅ Get Vacation Balance
- ✅ Get Sick Leave Balance

### CodeList-Tests

- ✅ Read Users, Projects, ActivityTypes, EntryTypes

## Manual Test Scenarios (REST Client)

The `test/track-service.http` file contains more than 50 predefined HTTP requests:

### CRUD Operations

- Get All TimeEntries (with $expand, $filter, $orderby)
- Create TimeEntry (Work, Vacation, Sick, Business Trip)
- Update TimeEntry
- Delete TimeEntry

### Generation & Balance

- Generate monthly/yearly entries
- Get balances (monthly, current, recent, vacation, sick)

### OData Query Options

- $select, $filter, $orderby, $top, $count, $expand
- Error scenarios (duplicate, invalid time range)

## References

- `test/track-service.test.ts` - example tests with Jest + cds.test()
- `test/track-service.http` - REST Client requests for manual tests
- `jest.config.js` - Jest configuration
- `.github/workflows/test.yml` - CI/CD pipeline
- `package.json` - npm scripts and dependencies

## Notes for Developers

### Test-Driven Development (TDD)

1. **Red**: Write a test that fails (new feature not implemented yet)
2. **Green**: Implement the feature until the test passes
3. **Refactor**: Improve the code while keeping tests green

### Writing Tests

```typescript
it('should calculate overtime correctly', async () => {
  const { data } = await POST(
    '/odata/v4/track/TimeEntries',
    {
      user_ID: 'max.mustermann@test.de',
      workDate: '2025-10-14',
      entryType_code: 'W',
      startTime: '08:00:00',
      endTime: '18:00:00', // 10h - 2h overtime
      breakMin: 60,
    },
    maxUser,
  );

  expect(data.overtimeHours).to.equal(2.0);
  expect(data.undertimeHours).to.equal(0);
});
```

### Using the REST Client

1. Install the VS Code extension "REST Client"
2. Open `test/track-service.http`
3. Start the CAP server: `npm run watch`
4. Click "Send Request" above any `###` section
5. Adjust variables as needed (e.g. `@server`, `@maxAuth`)

### CI/CD debuggen

- Lokaler Test-Run: `npm test` (sollte exakt gleich wie CI/CD laufen)
- Coverage lokal: `npm run test:coverage`
- GitHub Actions Logs: `Actions` Tab → fehlgeschlagener Workflow → Logs anzeigen

### Neue Dependencies

```bash
# Dependencies aktualisieren (User macht dies selbst)
npm install -D jest ts-jest @types/jest chai@4 chai-as-promised@7 chai-subset
```
