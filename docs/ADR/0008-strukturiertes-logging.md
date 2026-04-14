# ADR 0008: Structured Logging with Categorized Methods

## Status

Accepted - Logging Refactoring (Iteration 4)

## Context and Problem Statement

In early development phases, handlers, commands, and services used inconsistent logging approaches:

1. **Direct `console.log` calls**: No structured output, missing log levels, no filtering.
2. **Inconsistent `cds.log` usage**: Different components used different log categories (`track-service`, `handler`, `command`), making centralized configuration impossible.
3. **Missing context information**: Logs often contained only messages without structured context objects (e.g., `userId`, `command`, `error.stack`).
4. **No visual distinctions**: All logs looked the same, making quick identification of command, validation, or repository logs difficult.
5. **Difficult debugging workflows**: Developers had to manually search for relevant log lines since no prefixes or categories existed.

We needed a centralized logging system that provides consistent formatting, structured contexts, and categorized methods to enable fast troubleshooting and observability.

## Decision Factors

- **Consistency**: All components (handlers, commands, repositories, services) should use the same logging API.
- **Structured contexts**: Logs should contain structured context objects (e.g., `{ userId, command, error.stack }`), which can be exported in JSON format.
- **Visual distinctiveness**: Different log categories (command, validation, repository) should be distinguishable through prefixes and colors.
- **CAP-native integration**: The system should use `cds.log` under the hood to leverage CAP features like log level configuration and JSON export.
- **Performance**: Logging should have minimal overhead, especially for debug logs in production (lazy evaluation).
- **Configurability**: Log levels should be configurable via `package.json` or environment variables.

## Considered Options

### Option A - `console.log` with manual prefixes

- Handlers and commands call `console.log('[COMMAND] CreateTimeEntry started')`.
- Advantage: Simple, no additional abstraction.
- Disadvantage: No structured contexts, no log level filtering, no JSON export option, inconsistent formatting.

### Option B - Direct `cds.log` usage with manual categories

- Each component uses `cds.log('track-service')` directly.
- Advantage: CAP-native, log level configuration available.
- Disadvantage: Inconsistent formatting (each developer formats differently), no visual prefixes, difficult filtering by categories.

### Option C - Centralized logger with categorized methods

- A `TrackLogger` class encapsulates `cds.log('track-service')`.
- Categorized methods for different components: `logger.commandStart()`, `logger.validationSuccess()`, `logger.repositoryQuery()`, `logger.serviceCall()`. 
- Automatic prefixes with emojis and ANSI colors (disabled for JSON export).
- Structured contexts as a second parameter.
- Singleton pattern for global access.
- Advantage: Consistent API, visual distinctiveness, structured contexts, CAP integration.
- Disadvantage: Additional abstraction layer, higher initial learning curve.

## Decision

We choose **Option C** - a centralized logger with categorized methods. The implementation is located in `srv/track-service/handler/utils/logger.ts` and exports a singleton instance.

### Logger Structure

The `TrackLogger` provides 10 categories, each with specific methods:

#### 1. Service Lifecycle

- `logger.serviceInit(message, context?)` - Service initialization started
- `logger.serviceReady(message, context?)` - Service successfully initialized
- `logger.serviceRegistered(component, message, context?)` - Component registered

#### 2. Command Execution

- `logger.commandStart(command, context?)` - Command execution started
- `logger.commandEnd(command, context?)` - Command successfully completed
- `logger.commandInfo(command, message, context?)` - Command-specific info
- `logger.commandData(command, message, context?)` - Command data (debug level)

#### 3. Handler Invocation

- `logger.handlerInvoked(handler, event, context?)` - Handler invoked
- `logger.handlerCompleted(handler, event, context?)` - Handler completed

#### 4. Validation

- `logger.validationSuccess(entity, message, context?)` - Validation successful
- `logger.validationWarning(entity, message, context?)` - Validation warning
- `logger.validationInfo(entity, message, context?)` - Validation info

#### 5. Repository Operations

- `logger.repositoryQuery(entity, message, context?)` - Query being executed
- `logger.repositoryResult(entity, message, context?)` - Query result
- `logger.repositorySave(entity, count, context?)` - Save operation
- `logger.repositoryInfo(entity, message, context?)` - Repository info

#### 6. External Services

- `logger.serviceCall(service, message, context?)` - External service call (e.g., HolidayService)
- `logger.serviceCacheHit(service, message, context?)` - Cache hit
- `logger.serviceCacheCleared(service, context?)` - Cache cleared
- `logger.serviceInfo(service, message, context?)` - Service info

#### 7. Strategy Execution

- `logger.strategyExecute(strategy, message, context?)` - Strategy executed
- `logger.strategySkip(strategy, message, context?)` - Element skipped
- `logger.strategyEvent(strategy, message, context?)` - Special case
- `logger.strategyInfo(strategy, message, context?)` - Strategy info

#### 8. Factory

- `logger.factoryCreated(factory, instance, context?)` - Instance created

#### 9. Registry

- `logger.registryStart(message, context?)` - Handler registration started
- `logger.registrySuccess(handler, event, context?)` - Handler registered

#### 10. Generic/Utility

- `logger.info(message, context?)` / `logger.debug()` / `logger.warn()` / `logger.error(message, error?, context?)`
- `logger.userOperation(operation, message, context?)` - User operation
- `logger.calculationResult(type, message, context?)` - Calculation result
- `logger.stats(category, message, context?)` - Statistics/metrics

### Visual Formatting

Each method automatically adds a colored prefix (e.g., `🚀 [COMMAND]`, `✅ [VALIDATION:TimeEntry]`, `📅 [REPO:TimeEntry]`). Colors are added via ANSI codes and disabled for JSON export.

### Configuration

Log levels are configured via `package.json`:

```json
{
  "cds": {
    "log": {
      "levels": {
        "track-service": "info" // info | debug | warn | error
      }
    }
  }
}
```

Alternatively via environment variables:

```bash
# Development: All logs (including debug)
DEBUG=track-service

# Production: Only warnings/errors
CDS_LOG_LEVELS_TRACK_SERVICE=warn

# JSON format for log aggregation (e.g., Elasticsearch)
CDS_LOG_FORMAT=json
```

## Consequences

### Positive

- **Consistent formatting**: All logs follow the same pattern (`Emoji [PREFIX] Message + Context`), enabling quick identification.
- **Structured contexts**: All relevant data (e.g., `userId`, `command`, `error.stack`) is passed as objects and can be exported in JSON.
- **Visual distinctiveness**: Emojis and colors enable quick scanning of log outputs (e.g., `🚀 [COMMAND]` vs. `📅 [REPO]`).
- **CAP integration**: Uses `cds.log` under the hood, allowing log level configuration and JSON export.
- **Performance**: Debug logs can be disabled via log level, saving performance in production.
- **Easy migration**: Existing `console.log` calls can be gradually replaced with `logger.*` methods.

### Negative

- **Additional abstraction**: Developers need to know the correct logger methods (e.g., `commandStart` instead of `info`).
- **Higher learning curve**: New developers need to learn the 10 categories and their methods.
- **Boilerplate**: Each log call requires a method call with structured context, meaning more code than `console.log`.

### Trade-offs

We accept the additional abstraction and learning curve in favor of consistency, observability, and structured logs. The `.github/copilot-instructions.md` documents all logger methods in the AI development guide.

## Example Code

### Command Logging

```typescript
import { logger } from '../utils';

class CreateTimeEntryCommand {
  async execute(tx: Transaction, data: Partial<TimeEntry>) {
    logger.commandStart('CreateTimeEntry', { userId: data.user_ID, workDate: data.workDate });

    try {
      // Business logic
      const result = await this.process(tx, data);

      logger.commandEnd('CreateTimeEntry', { entryId: result.ID });
      return result;
    } catch (error: any) {
      logger.error('CreateTimeEntry failed', error, { userId: data.user_ID });
      throw error;
    }
  }
}
```

### Repository Logging

```typescript
import { logger } from '../utils';

export class TimeEntryRepository {
  async findById(tx: Transaction, id: string): Promise<TimeEntry | null> {
    logger.repositoryQuery('TimeEntry', `Finding by ID: ${id}`, { id });

    const entry = await tx.read(this.entries).where({ ID: id }).one();

    if (!entry) {
      logger.repositoryNotFound('TimeEntry', `Entry not found for ID: ${id}`, { id });
      return null;
    }

    logger.repositoryResult('TimeEntry', `Found entry`, { id, entryType: entry.entryType_code });
    return entry;
  }
}
```

### Service Logging

```typescript
import { logger } from '../utils';

export class HolidayService {
  async getHolidays(year: number, stateCode: string): Promise<Map<string, Holiday>> {
    if (this.cache.has(cacheKey)) {
      logger.serviceCacheHit('Holiday', `Holidays for ${year}/${stateCode}`, { year, stateCode });
      return this.cache.get(cacheKey)!;
    }

    logger.serviceCall('Holiday', `Fetching holidays from API for ${year}/${stateCode}`, { year, stateCode });
    // ... API call

    logger.serviceInfo('Holiday', `Loaded ${holidays.size} holidays`, { year, stateCode, count: holidays.size });
    return holidays;
  }
}
```

### Validation Logging

```typescript
import { logger } from '../utils';

export class TimeEntryValidator {
  async validate(tx: Transaction, data: Partial<TimeEntry>): Promise<void> {
    logger.validationInfo('TimeEntry', 'Starting validation', { userId: data.user_ID });

    if (!data.user_ID) {
      logger.validationWarning('TimeEntry', 'Missing user_ID', { data });
      throw new Error('User ID is required');
    }

    logger.validationSuccess('TimeEntry', 'All fields valid', { userId: data.user_ID });
  }
}
```

### Log Output Example

### Development (with colors)

```
🚀 [INIT] Initializing TrackService...
✅ [INIT] ServiceContainer: initialized
📝 [REGISTRY] Registering handlers...
✅ [REGISTRY] TimeEntryHandlers.onCreate registered
🎉 [READY] TrackService successfully initialized
🚀 [COMMAND] CreateTimeEntry started { userId: 'user123', workDate: '2025-10-14' }
📅 [REPO:TimeEntry] Finding by ID: abc123 { id: 'abc123' }
✅ [VALIDATION:TimeEntry] All fields valid { userId: 'user123' }
✅ [COMMAND] CreateTimeEntry completed { entryId: 'abc123' }
```

### Production (JSON format)

```json
{"level":"info","time":"2025-10-14T10:00:00.000Z","category":"track-service","message":"[INIT] Initializing TrackService..."}
{"level":"info","time":"2025-10-14T10:00:01.000Z","category":"track-service","message":"[COMMAND] CreateTimeEntry started","context":{"userId":"user123","workDate":"2025-10-14"}}
{"level":"info","time":"2025-10-14T10:00:01.500Z","category":"track-service","message":"[COMMAND] CreateTimeEntry completed","context":{"entryId":"abc123"}}
```

## References

- `srv/track-service/handler/utils/logger.ts` - Logger implementation with all methods
- `srv/track-service/handler/commands/time-entry/CreateTimeEntryCommand.ts` - Example for command logging
- `srv/track-service/handler/repositories/TimeEntryRepository.ts` - Example for repository logging
- `srv/track-service/handler/services/HolidayService.ts` - Example for service logging
- `package.json` - Log level configuration under `cds.log.levels`
- `.github/copilot-instructions.md` - Logger conventions in the AI development guide

## Developer Notes

- **Which method to use?** Refer to the component category: Commands → `commandStart/End`, Repositories → `repositoryQuery/Result`, Services → `serviceCall/Info`.
- **Structured contexts**: Always pass relevant data as an object (e.g., `{ userId, workDate, entryId }`).
- **Error logging**: Use `logger.error(message, error, context)` instead of `logger.info()` to capture error stacks.
- **Debug logs**: Use debug methods (e.g., `commandData`, `repositoryQuery`) for detailed logs that can be disabled in production.
- **Migration from `console.log`**: Replace `console.log('[COMMAND] ...')` with `logger.commandInfo('CommandName', '...').