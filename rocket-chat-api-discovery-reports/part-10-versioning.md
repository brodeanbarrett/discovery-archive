# Part 10: Versioning & Breaking Changes

## 10.1 Version Strategy

Rocket.Chat uses a **single API version** approach with incremental breaking changes managed through a feature flag system.

### API Version Structure

Rocket.Chat's REST API operates exclusively on **API v1**. All endpoints are accessible via `/api/v1/` paths.

| Aspect | Details |
|--------|---------|
| Current API Version | v1 |
| Base Path | `/api/v1/` |
| Version Declaration | `API.v1` in `apps/meteor/app/api/server/v1/` |

**Sources:**
- `apps/meteor/app/api/server/v1/` - Directory containing all v1 endpoints (channels.ts, users.ts, rooms.ts, etc.)
- `apps/meteor/app/api/server/api.ts:33` - API creation with version option: `createApi(options: { version?: string })`
- `packages/rest-typings/src/v1/` - TypeScript type definitions for v1 endpoints

### Version Transition Mechanism

Rocket.Chat implements a **feature flag approach** for managing breaking changes. The system allows gradual adoption of breaking changes based on the server version.

```typescript
// apps/meteor/app/api/server/ApiClass.ts:62
export const applyBreakingChanges = shouldBreakInVersion('9.0.0');
```

The `shouldBreakInVersion()` function compares the current server version against a planned breaking change version:

```typescript
// apps/meteor/server/lib/shouldBreakInVersion.ts
export const shouldBreakInVersion = (version: DeprecationLoggerNextPlannedVersion) => semver.gte(Info.version, version);
```

**Sources:**
- `apps/meteor/app/api/server/ApiClass.ts:50-62` - Breaking changes feature flag implementation
- `apps/meteor/server/lib/shouldBreakInVersion.ts` - Version comparison logic

---

## 10.2 Deprecation Policy

Rocket.Chat implements a comprehensive deprecation system with warnings, headers, and version-based removal.

### Deprecation Logger System

The deprecation system is implemented in `deprecationWarningLogger.ts` and provides multiple deprecation types:

```typescript
// apps/meteor/app/lib/server/lib/deprecationWarningLogger.ts:27
export type DeprecationLoggerNextPlannedVersion = '9.0.0';
```

**Deprecation Types:**

1. **Endpoint Deprecation** - Entire API endpoint is deprecated
2. **Parameter Deprecation** - Specific query/body parameter is deprecated
3. **Method Deprecation** - Meteor method is deprecated
4. **Invalid Usage** - Usage pattern that will break in future versions

### Deprecation Headers

When a deprecated endpoint is called, Rocket.Chat adds deprecation information to the response headers:

| Header | Description |
|--------|-------------|
| `x-deprecation-type` | Type of deprecation (endpoint-deprecation, parameter-deprecation, invalid-usage) |
| `x-deprecation-message` | Human-readable deprecation message |
| `x-deprecation-version` | Version where deprecation becomes a breaking change |

**Sources:**
- `apps/meteor/app/lib/server/lib/deprecationWarningLogger.ts:13-19` - Deprecation header writing
- `apps/meteor/app/lib/server/lib/deprecationWarningLogger.ts:29-45` - Endpoint deprecation implementation
- `apps/meteor/app/lib/server/lib/deprecationWarningLogger.ts:46-70` - Parameter deprecation implementation

### Deprecation Configuration in Endpoints

Endpoints can declare deprecations in their options:

```typescript
// apps/meteor/ee/app/api-enterprise/server/canned-responses.ts:112
deprecations: { DELETE: { version: '8.0.0', alternatives: ['/v1/canned-responses/:_id'] } },
```

The deprecation option structure:
```typescript
// apps/meteor/app/api/server/definition.ts:151-154
deprecation?: {
    version: DeprecationLoggerNextPlannedVersion;
    alternatives?: PathPattern[];
};
```

**Sources:**
- `apps/meteor/app/api/server/definition.ts:151-154` - Deprecation option type definition
- `apps/meteor/app/api/server/api.helpers.ts:108-114` - parseDeprecation function
- `apps/meteor/ee/app/api-enterprise/server/canned-responses.ts:112` - Example deprecation usage

### Deprecation Trigger Mechanism

When an endpoint is called with deprecated features:

```typescript
// apps/meteor/app/api/server/ApiClass.ts:818-845
if (options.deprecation && shouldBreakInVersion(options.deprecation.version)) {
    // Break in newer versions
}
if (options.deprecation) {
    parseDeprecation(this, options.deprecation);
}
```

**Sources:**
- `apps/meteor/app/api/server/ApiClass.ts:818` - Deprecation version check
- `apps/meteor/app/api/server/ApiClass.ts:844` - Deprecation parsing

### Environment-Based Error Control

Deprecation errors can be forced in test environments:

```typescript
// apps/meteor/app/lib/server/lib/deprecationWarningLogger.ts:11
const throwErrorsForVersionsUnder = process.env.ROCKET_CHAT_DEPRECATION_THROW_ERRORS_FOR_VERSIONS_UNDER;
```

When `TEST_MODE === 'true'`, deprecations throw errors instead of warnings.

**Sources:**
- `apps/meteor/app/lib/server/lib/deprecationWarningLogger.ts:34-36` - Test mode error throwing
- `apps/meteor/app/lib/server/lib/deprecationWarningLogger.ts:61-63` - Parameter deprecation test mode

---

## 10.3 Breaking Change Indicators

### CHANGELOG Markers

Rocket.Chat uses the `[BREAK]` prefix in CHANGELOG and history to mark breaking changes:

```
[BREAK] Remove deprecated /user.roles endpoint
[BREAK] Remove support to deprecated typing event
[BREAK] Remove deprecated file upload engine Slingshot
```

**Sources:**
- `.github/history.json:17254` - Example: `[BREAK] Remove deprecated /user.roles endpoint`
- `.github/history.json:21936` - Example: `[FIX] Some deprecation issues for media recording`

### Breaking Change Application

The `applyBreakingChanges` flag controls whether breaking changes are active:

```typescript
// apps/meteor/app/api/server/ApiClass.ts:818
if (options.deprecation && shouldBreakInVersion(options.deprecation.version)) {
    // Applies breaking change behavior
}
```

Breaking changes are applied when `shouldBreakInVersion(deprecation.version)` returns true.

**Sources:**
- `apps/meteor/app/api/server/ApiClass.ts:818` - Breaking change application logic
- `apps/meteor/server/lib/shouldBreakInVersion.ts` - Version comparison

### Deprecation Metrics

Deprecation events are tracked via metrics:

```typescript
// apps/meteor/app/lib/server/lib/deprecationWarningLogger.ts:42
metrics.deprecations.inc({ type: 'deprecation', kind: 'endpoint', name: endpoint });
```

**Sources:**
- `apps/meteor/app/lib/server/lib/deprecationWarningLogger.ts:42` - Endpoint deprecation metric
- `apps/meteor/app/lib/server/lib/deprecationWarningLogger.ts:65` - Parameter deprecation metric

---

## 10.4 Migration Guides

### Database Migrations

Rocket.Chat uses a migration system for database schema changes:

- **Migration Location**: `apps/meteor/server/lib/migrations.ts`
- **Migration Files**: Located in `apps/meteor/server/differential-migrations/`
- **Migration Naming**: `v{number}.ts` (e.g., `v267.ts`)

```typescript
// apps/meteor/server/lib/migrations.ts:12
up: (migration: IMigration) => Promise<void> | void;
```

**Sources:**
- `apps/meteor/server/lib/migrations.ts` - Migration system definition
- `.scripts/migration.template` - Migration file template

### Migration Execution

Migrations are run automatically on server startup. The system tracks:
- Current migration version
- Migration lock status
- Migration execution state

```typescript
// apps/meteor/app/metrics/server/lib/metrics.ts:98
migration: new client.Gauge({ name: 'rocketchat_migration', help: 'migration version' }),
```

**Sources:**
- `apps/meteor/app/metrics/server/lib/metrics.ts:98` - Migration metrics
- `apps/meteor/client/views/admin/workspace/DeploymentCard/DeploymentCard.tsx:67` - Migration version display

### Migration Documentation

Migration details are documented in:
- **CHANGELOG.md** - Version-specific migration notes
- **HISTORY.md** - Detailed migration descriptions
- Commit messages with migration context

**Sources:**
- `apps/meteor/CHANGELOG.md` - Changelog with migration entries
- `apps/meteor/HISTORY.md` - History with migration details

---

## 10.5 Support Windows

### MongoDB Support

Rocket.Chat maintains support windows for MongoDB versions:

| Version | Support Status |
|---------|----------------|
| MongoDB 8.0+ | Minimum supported (as of 8.2.0) |
| MongoDB 4.4 | Was supported, now deprecated |
| MongoDB 4.2 | Removed in recent versions |

**Sources:**
- `apps/meteor/CHANGELOG.md:71` - MongoDB 8.0 minimum support
- `apps/meteor/HISTORY.md:1954` - MongoDB 4.2 deprecation notice

### License-Based Version Support

Rocket.Chat supports version-specific features through licenses:

```typescript
// apps/meteor/app/cloud/server/functions/supportedVersionsToken/supportedVersionsToken.ts:139
// Gets the supported versions from the license
```

**Sources:**
- `apps/meteor/app/cloud/server/functions/supportedVersionsToken/supportedVersionsToken.ts` - License version support

### API Version Support

The current API version (v1) is the only actively supported version. There is no v2 API in this codebase:

```bash
# apps/meteor/app/api/server/
ls v1/
# Only v1 exists - no v2 directory
```

**Sources:**
- `apps/meteor/app/api/server/` - API version directories (only v1 exists)

### Node.js Support

Rocket.Chat tracks Node.js version compatibility through the release cycle.

---

## Summary

Rocket.Chat implements a pragmatic versioning strategy:

1. **Single API Version**: All endpoints use `/api/v1/`
2. **Feature Flag Breaking Changes**: Uses `shouldBreakInVersion()` for gradual breaking change adoption
3. **Deprecation System**: Comprehensive logging with headers, metrics, and version-based removal
4. **Migration System**: Automatic database migrations on startup
5. **Support Windows**: Documented support for MongoDB and other dependencies

The current deprecation timeline targets version **9.0.0** for breaking changes to deprecated features.
