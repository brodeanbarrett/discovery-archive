# Rocket.Chat API & SDK Discovery Audit Report

---

## 1. Executive Summary

Rocket.Chat is an open-source team communications platform written in TypeScript/Node.js providing real-time chat, video conferencing, voice calls, and omnichannel customer engagement. The platform exposes a comprehensive REST API v1, DDP (WebSocket) for real-time communication, and supports webhooks for event-driven integrations.

**Key Findings:**
- **REST API v1** is the primary production API at `/api/v1/`
- **DDP (WebSocket)** via `@rocket.chat/ddp-client` for real-time communication
- **GraphQL removed** in recent versions (PR #15356)
- **No gRPC support** in current codebase
- Multiple official SDKs: JavaScript/TypeScript, Python, Java, Go, React Native
- Comprehensive authentication: OAuth 2.0, SAML, LDAP, CAS, Personal Access Tokens, TOTP
- 200+ REST endpoints across users, rooms, channels, groups, livechat, and more

---

## 2. API Overview

### 2.1 API Types Discovered

| API Type | Present | Notes |
|----------|---------|-------|
| REST | **Yes** | Primary API - `/api/v1/` endpoints |
| GraphQL | **No** | Removed in PR #15356 |
| gRPC | **No** | Not found |
| WebSocket (DDP) | **Yes** | Real-time via `@rocket.chat/ddp-client` |
| Webhook | **Yes** | Both incoming and outgoing supported |

**Sources:**
- `apps/meteor/app/api/server/api.ts` - REST API implementation
- `packages/ddp-client/README.md` - WebSocket client
- HISTORY.md: "Remove GraphQL dependencies left"

### 2.2 Base URLs & Environments

| Environment | Base URL | Source |
|-------------|----------|--------|
| Development | `http://localhost:3000` | Default development |
| REST | `{Site_Url}/api/v1/` | `apps/meteor/server/settings/general.ts` |
| WebSocket | `{Site_Url}/websocket` | DDP client |
| SockJS | `{Site_Url}/sockjs/` | DDP fallback |

**Configuration:** Site URL is set via `Site_Url` setting or `ROOT_URL` environment variable

### 2.3 API Versioning

Rocket.Chat uses **URL path-based versioning**:
- Version: v1 only
- Path Pattern: `/api/v1/<endpoint>`
- Breaking changes managed via feature flag `shouldBreakInVersion('9.0.0')`

**Sources:**
- `apps/meteor/app/api/server/api.ts` (lines 33-75)
- `apps/meteor/server/lib/shouldBreakInVersion.ts`

---

## 3. REST API Documentation

### 3.1 Endpoints

Rocket.Chat provides 200+ REST API endpoints. Key modules in `apps/meteor/app/api/server/v1/`:

| Module | Description |
|--------|-------------|
| `users.ts` | User management |
| `channels.ts` | Channel operations |
| `groups.ts` | Private group operations |
| `rooms.ts` | General room operations |
| `im.ts` | Direct message operations |
| `chat.ts` | Message operations |
| `subscriptions.ts` | Subscription management |
| `settings.ts` | Server settings |
| `integrations.ts` | Integration management |
| `teams.ts` | Team management |
| `push.ts` | Push notifications |
| `videoConference.ts` | Video conferencing |

**Livechat API** (`apps/meteor/app/livechat/server/api/v1/`):
- agent, config, contact, customField, integration, message
- room, statistics, transcript, transfer, visitor, webhooks

### 3.2 Authentication Headers

| Header | Required | Description |
|--------|----------|-------------|
| `X-User-Id` | Conditional | User identifier |
| `X-Auth-Token` | Conditional | Authentication token |

**Source:** `apps/meteor/app/api/server/default/openApi.ts`

### 3.3 Pagination & Query Parameters

| Parameter | Default | Description |
|-----------|---------|-------------|
| `count` | 50 | Items to return |
| `offset` | 0 | Items to skip |
| `sort` | - | JSON sort specification |
| `query` | - | Query filter (deprecated) |

Configuration:
- `API_Default_Count`: 50
- `API_Upper_Count_Limit`: 100
- `API_Allow_Infinite_Count`: true

**Source:** `apps/meteor/app/api/server/helpers/getPaginationItems.ts`

---

## 4. DDP/WebSocket API

### 4.1 Connection Endpoints

| Environment | WebSocket URL |
|-------------|---------------|
| Development | `http://localhost:3000/websocket` |
| Production | `{Site_Url}/websocket` |
| SockJS Fallback | `{Site_Url}/sockjs/` |

### 4.2 DDP Client

**Package:** `@rocket.chat/ddp-client`
**Installation:** `npm install @rocket.chat/ddp-client`

**Features:**
- Real-time message updates
- Room subscriptions
- User presence
- Method calls

**Source:** `packages/ddp-client/README.md`

---

## 5. Authentication & Authorization

### 5.1 Supported Authentication Methods

| Method | Supported | Source |
|--------|-----------|--------|
| Password | Yes | `apps/meteor/app/authentication/server/startup/index.js` |
| OAuth 2.0 | Yes (13 providers) | `apps/meteor/server/settings/oauth.ts` |
| SAML 2.0 | Yes | `apps/meteor/app/meteor-accounts-saml/` |
| LDAP | Yes | `apps/meteor/server/lib/ldap/Connection.ts` |
| CAS | Yes | `apps/meteor/server/lib/cas/loginHandler.ts` |
| Personal Access Tokens | Yes | `apps/meteor/app/api/server/v1/users.ts` |
| TOTP (2FA) | Yes | `apps/meteor/server/settings/accounts.ts` |
| Iframe | Yes | `apps/meteor/server/settings/accounts.ts` |

### 5.2 OAuth Providers

Google, GitHub, GitHub Enterprise, GitLab, Facebook, Twitter, LinkedIn, Apple, Nextcloud, WordPress, Drupal, Meteor, Dolphin, Custom OAuth

### 5.3 Token Endpoints

| Endpoint | Method | Purpose |
|----------|--------|---------|
| `/oauth/token` | POST | OAuth token issuance |
| `/api/v1/users.generatePersonalAccessToken` | POST | PAT generation |
| `/api/v1/me` | GET | Authenticated user info |

### 5.4 Token Expiration

- Login tokens: 90 days (default, via `Accounts_LoginExpiration`)
- JWT: 1 hour default
- TOTP codes: 3600 seconds

### 5.5 Permissions

150+ granular permissions across categories:
- Administration
- User Management
- Room Management
- Message Management
- Integration
- OAuth
- Livechat
- Teams

### 5.6 Default Roles

admin, moderator, owner, user, guest, bot, app, anonymous, federated-external

**Source:** `apps/meteor/app/authorization/server/constant/permissions.ts`

---

## 6. Official SDKs

### 6.1 JavaScript/TypeScript SDKs

| Package | Registry | Version | Status |
|---------|----------|---------|--------|
| `@rocket.chat/api-client` | npm | 0.2.51 | Active |
| `@rocket.chat/ddp-client` | npm | 1.0.4 | Active |
| `@rocket.chat/rest-typings` | npm | 8.3.0-develop | Active |
| `@rocket.chat/apps-engine` | npm | 1.60.0 | Active |
| `@rocket.chat/core-typings` | npm | 8.3.0-develop | Active |
| `@rocket.chat/federation-sdk` | npm | - | Active |

**Installation:**
```bash
npm install @rocket.chat/api-client @rocket.chat/ddp-client
```

### 6.2 Other Language SDKs

| Language | Package | Status |
|----------|---------|--------|
| Python | `rocketchat_API` (PyPI) | Community - Active |
| Java/Android | Rocket.Chat.Java.SDK | Deprecated |
| Go | Rocket.Chat.Go.SDK | Unmaintained |
| React Native | Rocket.Chat.ReactNative | Active (v4.70.1) |
| Kotlin | rocket.chat.kotlin | Community |

### 6.3 SDK Feature Comparison

| Feature | api-client | ddp-client | Python |
|---------|------------|------------|--------|
| REST API | Yes | No | Yes |
| DDP | No | Yes | No |
| Authentication | Yes | Yes | Yes |
| Messaging | Yes | Yes | Yes |
| File Uploads | Yes | Yes | Yes |

---

## 7. Request & Response Formats

### 7.1 Request Formats

- **JSON** (primary)
- Form URL-encoded
- Multipart Form Data

### 7.2 Response Formats

- JSON only

### 7.3 Data Schemas

- AJV JSON Schema validation
- TypeScript types in `@rocket.chat/rest-typings` and `@rocket.chat/core-typings`

### 7.4 Field Naming

- **API params:** camelCase
- **Database:** snake_case (_id, createdAt, updatedAt)

### 7.5 Date/Time Formats

- ISO 8601 via `toISOString()`
- JavaScript Date objects
- Unix timestamps (in some contexts)

### 7.6 Error Response Schema

```json
{
  "success": false,
  "error": "error message",
  "errorType": "error-type",
  "details": [],
  "message": "human readable message",
  "stack": "error stack"
}
```

**Source:** `apps/meteor/app/api/server/definition.ts`

---

## 8. Rate Limiting & Quotas

### 8.1 Rate Limit Headers

```
X-RateLimit-Limit: 10
X-RateLimit-Remaining: 8
X-RateLimit-Reset: 1705312800
```

### 8.2 Limit Values

**REST API:**
- Default: 10 requests per 60000ms
- Configurable via settings

**DDP:**
- IP: 120,000/60s
- User: 1,200/60s
- Connection: 600/60s
- User_By_Method: 20/10s
- Connection_By_Method: 10/10s

### 8.3 Throttling Responses

- HTTP 429 status code
- Error: `error-too-many-requests`
- Includes `timeToReset` and `seconds` in error details

**Sources:**
- `apps/meteor/app/api/server/ApiClass.ts` (lines 425-441)
- `apps/meteor/server/settings/rate.ts`

---

## 9. Error Handling

### 9.1 HTTP Status Codes

200, 201, 400, 401, 403, 404, 422, 429, 500, 503

### 9.2 Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `error-user-param-not-provided` | 400 | Missing required parameter |
| `error-room-not-found` | 404 | Room not found |
| `error-invalid-user` | 400 | Invalid user |
| `error-action-not-allowed` | 403 | Action not permitted |
| `error-too-many-requests` | 429 | Rate limited |

### 9.3 Retry Strategies

Webhook retry strategies:
- Powers-of-ten: 0.1s to 27h+
- Powers-of-two: 2s, 4s, 8s...
- Increments-of-two: 2s, 4s, 6s...

**Source:** `apps/meteor/app/integrations/server/lib/triggerHandler.ts`

### 9.4 Timeout Configuration

- Default: 20s (`HTTP_DEFAULT_TIMEOUT`)
- Apps HTTP bridge: 3 minutes
- Livechat: configurable

---

## 10. Versioning & Breaking Changes

### 10.1 Version Strategy

- Single API version: v1
- Breaking changes via feature flag `shouldBreakInVersion('9.0.0')`
- Deprecation warnings with response headers

### 10.2 Deprecation Headers

```
x-deprecation-type: endpoint
x-deprecation-message: "Use /api/v2/..."
x-deprecation-version: "8.0.0"
```

### 10.3 Breaking Change Indicators

- `[BREAK]` prefix in CHANGELOG
- `applyBreakingChanges` flag
- `shouldBreakInVersion()` function

### 10.4 Database Migrations

- Automatic migration system in `server/lib/migrations.ts`
- Version-based (v{number}.ts files)

**Source:** `apps/meteor/server/lib/migrations.ts`

---

## 11. Developer Experience

### 11.1 Documentation

- Comprehensive at `developer.rocket.chat/apidocs`
- Auto-generated OpenAPI 3.0.3 specs
- API guidelines in `apps/meteor/app/api/README.md`

### 11.2 Interactive Playground

- Swagger UI at `/api-docs`
- OpenAPI JSON at `/api/docs/json`
- Uses `swagger-ui-express` package

### 11.3 Type Definitions

- Full TypeScript support
- `@rocket.chat/core-typings` and `@rocket.chat/rest-typings`
- .d.ts exports in all packages

### 11.4 Changelog

- Extensive HISTORY.md (35,506 lines)
- All API changes documented

### 11.5 Support Channels

- Community server: open.rocket.chat
- GitHub Issues
- Security: HackerOne bug bounty
- Documentation: docs.rocket.chat

---

## 12. Code Examples

### 12.1 REST API Client (TypeScript)

```typescript
import { API } from '@rocket.chat/api-client';

const api = new API({
  baseUrl: 'http://localhost:3000',
  auth: {
    userId: 'user-id',
    token: 'auth-token'
  }
});

// List users
const users = await api.get('users.list');

// Create user
const newUser = await api.post('users.create', {
  name: 'John Doe',
  email: 'john@example.com',
  password: 'password123'
});
```

### 12.2 DDP Client

```typescript
import { DDPClient } from '@rocket.chat/ddp-client';

const ddp = new DDPClient();
await ddp.connect('ws://localhost:3000/websocket');

ddp.loginWithPassword({
  user: { username: 'username' },
  password: 'password'
});

// Subscribe to messages
ddp.subscribe('roomMessages', ['room-id']);
ddp.on('message', (message) => {
  console.log('New message:', message);
});
```

### 12.3 Authentication Example

```typescript
// Login via REST
const loginResponse = await fetch('http://localhost:3000/api/v1/login', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ user: 'username', password: 'password' })
});

const { data: { userId, authToken } } = await loginResponse.json();

// Use token in subsequent requests
const headers = {
  'X-User-Id': userId,
  'X-Auth-Token': authToken
};
```

---

## Summary

Rocket.Chat provides a mature, well-documented REST API with comprehensive authentication options and official SDKs for multiple languages. The platform is actively maintained with regular updates documented in an extensive changelog.

**Key Strengths:**
- 200+ REST endpoints covering all platform features
- Multiple authentication methods (OAuth, SAML, LDAP, PAT)
- Official TypeScript/JavaScript SDKs
- Real-time support via DDP/WebSocket
- OpenAPI documentation with interactive playground

**Limitations:**
- GraphQL removed (no longer available)
- No gRPC support
- Some community SDKs (Python, Java, Go) are unmaintained or deprecated

---

*Report generated from code discovery audit of Rocket.Chat repository*
