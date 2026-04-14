# ADR 0010: Mocked Authentication with Test Users for Local Development

## Status

Accepted - Project start (historical)

## Context and Problem Statement

For local development of the time tracking application, we needed a simple authentication system that meets the following requirements:

1. **Fast development workflow**: Developers should be able to sign in without complex OAuth2/XSUAA configuration.
2. **Multi-user testing**: Testing user-specific features (e.g. own TimeEntries, user preferences) requires multiple test users.
3. **Realistic user IDs**: User IDs should use email format to simulate production-like data (e.g. `max.mustermann@test.de` instead of `user1`).
4. **Simple credentials**: Login credentials (username/password) should be easy to remember for developers and testers.
5. **No external dependency**: No need for external identity providers (e.g. SAP Identity Authentication Service) during local development.
6. **Production-ready migration**: The system should be easy to switch to real authentication (e.g. XSUAA, JWT) without code changes in business logic.

Early prototypes had no authentication, which caused the following problems:

- User-specific features (e.g. `UserService.getCurrentUserID()`) could not be tested.
- TimeEntries had no realistic user assignments.
- Multi-user scenarios (e.g. team lead views employee entries) were not testable.

## Decision Factors

- **Simplicity**: Developers should be able to start local development without setup.
- **Multi-user support**: At least 2 test users for different scenarios.
- **Realistic data**: User IDs in email format, German names for readability.
- **CAP-native**: Use CAP's native `auth.kind: mocked` configuration without additional libraries.
- **Production compatibility**: Easy switch to real authentication in production.

## Considered Options

### Option A - No Authentication (Anonymous)

- No login, all requests are anonymous.
- Pros: No setup, simplest solution.
- Cons: User-specific features are not testable, unrealistic production environment.

### Option B - Hard-coded User in Code

- `UserService.getCurrentUserID()` returns a hard-coded user ID (e.g. `'user123'`).
- Pros: Simple, no authentication configuration.
- Cons: Multi-user testing impossible, unrealistic user IDs, requires production code changes.

### Option C - CAP Mocked Authentication with Test Users

- Use CAP's `auth.kind: mocked` in `package.json`.
- Define test users with credentials and roles.
- Login via CAP standard login page (`/login`).
- Pros: CAP-native, multi-user support, realistic user IDs, no code changes for production.
- Cons: Additional configuration in `package.json`.

### Option D - Local OAuth2 Mock Server

- Separate mock server (e.g. Keycloak, mock XSUAA) for local development.
- Pros: Production-like authentication flows.
- Cons: High complexity, additional infrastructure, slower setup.

## Decision

We choose **Option C** - CAP mocked authentication with test users. The configuration is done in `package.json` under `cds.requires.auth`:

### Configuration in `package.json`

```json
{
  "cds": {
    "requires": {
      "auth": "ias",
      "[development]": {
        "auth": {
          "kind": "mocked",
          "users": {
            "max.mustermann@test.de": {
              "password": "max",
              "roles": ["TimeTrackingUser", "TimeTrackingAdmin"],
              "policies": ["cap.TimeTrackingUser", "cap.TimeTrackingAdmin"]
            },
            "erika.musterfrau@test.de": {
              "password": "erika",
              "roles": ["TimeTrackingUser"],
              "policies": ["cap.TimeTrackingUser"]
            },
            "frank.genehmiger@test.de": {
              "password": "frank",
              "roles": ["TimeTrackingUser", "TimeTrackingApprover"],
              "policies": ["cap.TimeTrackingUser", "cap.TimeTrackingApprover"]
            }
          }
        }
      }
    }
  }
}
```

### Test User Details

We define 3 test users with German names and production-like role labels so we can locally simulate the later authorization concept:

#### User 1: Max Mustermann

- **User ID**: `max.mustermann@test.de` (corresponds to the email in the user profile)
- **Password**: `max` (easy to remember)
- **Roles**: `TimeTrackingUser`, `TimeTrackingAdmin` (admin scenario for local tests)
- **Profile**: Referenced in `db/data/io.nimble-Users.csv` with full profile data (name, weeklyHoursDec, etc.)

#### User 2: Erika Musterfrau

- **User ID**: `erika.musterfrau@test.de`
- **Password**: `erika`
- **Role**: `TimeTrackingUser`
- **Profile**: Referenced in `db/data/io.nimble-Users.csv`

#### User 3: Frank Genehmiger

- **User ID**: `frank.genehmiger@test.de`
- **Password**: `frank`
- **Roles**: `TimeTrackingUser`, `TimeTrackingApprover`
- **Profile**: Referenced in `db/data/io.nimble-Users.csv`

### Login Flow

1. Developer starts the app with `npm run watch`.
2. Browser opens `http://localhost:4004/`.
3. CAP displays the standard login page with user selection.
4. Developer selects a user (e.g. `max.mustermann@test.de`) and enters the password (`max`).
5. Session is created, and `req.user.id` returns the user ID.

### Integration with UserService

`UserService.getCurrentUserID()` uses `req.user.id` from the CAP request context:

```typescript
export class UserService {
  static getCurrentUserID(req?: any): string {
    const userId = req?.user?.id || cds.context?.user?.id;
    if (!userId) {
      throw new Error('User not authenticated');
    }
    return userId;
  }
}
```

## Consequences

### Positive

- **Simpler development workflow**: Developers can start the app and sign in with one click, no OAuth2 configuration needed.
- **Multi-user testing**: Two test users allow testing user-specific features (e.g. "Max creates an entry, Erika does not see it").
- **Realistic user IDs**: Email format (`max.mustermann@test.de`) matches production user IDs, which yields more realistic test data.
- **Simple credentials**: Passwords like `max` and `erika` are easy to remember and communicate within the team.
- **CAP-native integration**: No additional libraries or mock server required; uses CAP standard features.
- **Production-ready**: Switching to real authentication (e.g. XSUAA) only requires changing `auth.kind` in `package.json`, no business logic changes.
- **Role support**: Enables future extension with role-based authorization (e.g. `admin`, `team-lead`).

### Negative

- **Not production-suitable**: Mocked authentication is only for development/testing, not for production.
- **Limited user count**: Only 3 test users are defined; more users require manual updates in `package.json`.
- **Simple passwords**: Passwords like `max` are insecure, but acceptable for local development.
- **No password rotation**: Passwords are static in `package.json`; no rotation or expiration mechanism.

### Trade-offs

We accept the limited user count and simple passwords in favor of development speed and simplicity. For production deployment, `auth.kind` will be switched to `xsuaa` or `jwt`, activating real user management.

## Example Code

### Login Page (generated automatically by CAP)

CAP automatically displays a login page under `/login` with user selection:

```
┌─────────────────────────────────┐
│  Login                          │
│                                 │
│  Username: [max.mustermann...]   │
│  Password: [•••]                │
│                                 │
│  [Login]                        │
└─────────────────────────────────┘
```

### UserService Integration

```typescript
// srv/track-service/handler/services/UserService.ts
export class UserService {
  /**
   * Returns the currently authenticated user's ID
   * @param req - Request object (optional if not available in cds.context)
   * @returns User ID (e.g. 'max.mustermann@test.de')
   */
  static getCurrentUserID(req?: any): string {
    // Try to determine the user ID from the request or CDS context
    const userId = req?.user?.id || cds.context?.user?.id;

    if (!userId) {
      logger.error('User not authenticated', null, { context: 'UserService.getCurrentUserID' });
      throw new Error('User not authenticated');
    }

    logger.userOperation('GetCurrentUserID', `User ${userId} authenticated`, { userId });
    return userId;
  }
}
```

### Handler Usage

```typescript
// srv/track-service/handler/handlers/TimeEntryHandlers.ts
class TimeEntryHandlers {
  async onCreate(req: Request) {
    // User ID is automatically determined from the authentication context
    const userId = UserService.getCurrentUserID(req);

    // Automatically assign current user to the new entry
    req.data.user_ID = userId;

    const result = await this.createCommand.execute(req.transaction, req.data);
    return result;
  }
}
```

## Production Migration

### Step 1: Change authentication kind

```json
// package.json (Production)
{
  "cds": {
    "requires": {
      "auth": {
        "kind": "xsuaa" // <- changed from "mocked"
        // XSUAA-specific config here
      }
    }
  }
}
```

### Step 2: No code changes required

`UserService.getCurrentUserID()` and all commands/handlers work unchanged because they only use `req.user.id`, which CAP provides for all authentication types.

### Step 3: Migrate user profiles

Test user data from `db/data/io.nimble-Users.csv` must be replaced with real user profiles sourced from XSUAA or an identity provider.

## References

- `package.json` - mocked authentication configuration under `cds.requires.auth`
- `srv/track-service/handler/services/UserService.ts` - `getCurrentUserID()` implementation
- `db/data/io.nimble-Users.csv` - user profiles for test users
- `.github/copilot-instructions.md` - mock user details in the AI development guide

## Notes for Developers

- **Login credentials**: `max.mustermann@test.de` / `max` and `erika.musterfrau@test.de` / `erika`
- **Add a user**: Add a new user in `package.json` under `cds.requires.auth.users`, and create the user profile in `db/data/io.nimble-Users.csv`.
- **Logout**: Clear the browser session or call `/logout`.
- **Production deployment**: Change `auth.kind` to `xsuaa` or `jwt`; no code changes required.
- **Test roles**: Define new roles in `package.json` (e.g. `"roles": ["TimeTrackingUser", "TimeTrackingAdmin"]`) and verify with `req.user.is('TimeTrackingAdmin')`.
