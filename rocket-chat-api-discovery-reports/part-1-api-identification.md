# Part 1: API Identification & Types

## 1. Repository Overview

Rocket.Chat is an open-source team communications platform written in TypeScript/Node.js. It provides real-time chat, video conferencing, voice calls, and omnichannel customer engagement capabilities.

**Primary API Offerings:**
- **REST API** (v1) - Primary production API for programmatic access
- **DDP (Distributed Data Protocol)** - Meteor-based WebSocket protocol for real-time communication
- **Webhooks** - For event-driven integrations (incoming and outgoing)

**Repository Location:** `./rocket-chat-api-discovery/Rocket.Chat/`

**Key Source:**
- README.md (line 18-20)
- apps/meteor/app/api/README.md (API development guidelines)

---

## 2. API Type Classification

| API Type | Present | Notes |
|----------|---------|-------|
| REST | **Yes** | Primary API - `/api/v1/` endpoints |
| GraphQL | **No** | Removed in PR #15356 (dependencies removed) |
| gRPC | **No** | Not found |
| WebSocket (DDP) | **Yes** | Real-time communication via `@rocket.chat/ddp-client` |
| Webhook | **Yes** | Both incoming and outgoing webhooks supported |

**Sources:**
- HISTORY.md (line 26293): "Remove GraphQL dependencies left"
- apps/meteor/app/api/server/v1/*.ts - REST endpoints
- packages/ddp-client/README.md - WebSocket/DDP client documentation

---

## 3. Primary API Discovery

### REST API (Production)

The primary production API is the REST API v1:

- **Base Path:** `/api/v1/`
- **Implementation:** `apps/meteor/app/api/server/api.ts` (line 72-75)
- **Version:** v1 (defined as `version: 'v1'`)

**Key Endpoint Files (in `apps/meteor/app/api/server/v1/`):**
- `users.ts` - User management
- `rooms.ts` - Room/channel operations
- `channels.ts` - Channel-specific operations
- `groups.ts` - Private group operations
- `im.ts` - Direct message operations
- `chat.ts` - Message operations
- `subscriptions.ts` - Subscription management
- `settings.ts` - Server settings
- `integrations.ts` - Integration management
- `push.ts` - Push notifications
- `videoConference.ts` - Video conferencing
- `teams.ts` - Team management

**Authentication:**
- Uses `X-User-Id` and `X-Auth-Token` headers
- Defined in `packages/api-client/src/index.ts` (lines 198-205)

**Example Endpoint:**
```typescript
// apps/meteor/app/api/server/v1/users.ts
API.v1.get('users.list', { /* endpoint definition */ });
```

**Source:**
- apps/meteor/app/api/server/api.ts (line 44, 72-75)
- apps/meteor/app/api/server/v1/*.ts

---

## 4. Internal APIs

### DDP (Method Calls)

Rocket.Chat uses DDP (Meteor's Distributed Data Protocol) for internal client-server communication:

- **Purpose:** Real-time updates, method calls, and subscriptions
- **Implementation:** Via `@rocket.chat/ddp-client` package
- **Endpoint:** `/sockjs/` and `/websocket/` paths

**Note:** Recent changes (PR #32590) moved method calls to HTTP endpoints:
- `/api/v1/method.call` - Authenticated method calls
- `/api/v1/method.callAnon` - Anonymous method calls

**Sources:**
- ee/apps/ddp-streamer/ - DDP streamer service
- apps/meteor/CHANGELOG.md (line 743): "Removes the setting API_Use_REST_For_DDP_Calls"
- apps/meteor/app/api/server/middlewares/metrics.ts (line 34): method.call handling
- packages/ddp-client/README.md

### Microservice Communication

For microservice deployments, internal communication uses HTTP REST endpoints.

**Source:**
- apps/meteor/CHANGELOG.md (line 743)

---

## 5. API Versioning Strategy

Rocket.Chat uses **URL path-based versioning**:

- **Version:** v1
- **Path Pattern:** `/api/v1/<endpoint>`
- **No Header-based Versioning:** Not found

**Version Definition:**
```typescript
// apps/meteor/app/api/server/api.ts (lines 33-40)
const createApi = function _createApi(options: { version?: string; useDefaultAuth?: true } = {}): APIClass {
    return new APIClass({
        apiPath: '',
        useDefaultAuth: false,
        prettyJson: process.env.NODE_ENV === 'development',
        ...options,
    });
};

// v1 API instance (lines 72-75)
v1: createApi({
    version: 'v1',
    useDefaultAuth: true,
}),
```

**Breaking Changes Management:**
- Feature flag approach using `shouldBreakInVersion('9.0.0')`
- Defined in `apps/meteor/app/api/server/ApiClass.ts` (line 62)

**Source:**
- apps/meteor/app/api/server/api.ts (lines 33-75)
- apps/meteor/app/api/server/ApiClass.ts (line 62)

---

## 6. Base URL/Endpoint Documentation

### Default Base URLs

| Environment | Base URL |
|-------------|----------|
| Development | `http://localhost:3000` |
| Production | Configurable via `Site_Url` setting |

**Configuration:**
- **Setting:** `Site_Url` (stored in database)
- **Default:** Set via `ROOT_URL` environment variable at startup
- **Location:** `apps/meteor/server/settings/general.ts` (lines 58-68)

```typescript
// Site_Url setting definition
await this.add(
    'Site_Url',
    typeof (global as any).__meteor_runtime_config__ !== 'undefined'
        ? (global as any).__meteor_runtime_config__.ROOT_URL
        : null,
    { type: 'string', public: true },
);
```

### REST API Endpoints

| Environment | Base REST URL |
|-------------|---------------|
| Development | `http://localhost:3000/api/v1/` |
| Production | `{Site_Url}/api/v1/` |

**Example Full URLs:**
- `http://localhost:3000/api/v1/users.list`
- `http://localhost:3000/api/v1/rooms.create`

### WebSocket Endpoints

| Environment | WebSocket URL |
|-------------|---------------|
| Development | `http://localhost:3000/websocket` |
| Production | `{Site_Url}/websocket` |

**SockJS Fallback:**
- `{Site_Url}/sockjs/`

### Configuration Settings

**API-Related Settings** (in `apps/meteor/server/settings/general.ts` and `rate.ts`):
- `API_Upper_Count_Limit` - Default: 100
- `API_Default_Count` - Default: 50
- `API_Allow_Infinite_Count` - Default: true
- `API_Enable_CORS` - Default: false
- `API_CORS_Origin` - Default: `*`
- `API_Enable_Rate_Limiter` - Default: true
- `API_Enable_Rate_Limiter_Limit_Calls_Default` - Default: 10
- `API_Enable_Rate_Limiter_Limit_Time_Default` - Default: 60000ms

**Sources:**
- apps/meteor/server/settings/general.ts (lines 5-36, 58-68)
- apps/meteor/server/settings/rate.ts (lines 57-71)
- packages/ddp-client/README.md (lines 19, 27)

---

## Summary

Rocket.Chat provides a comprehensive REST API (v1) as its primary production API, with DDP/WebSocket for real-time client communication. The API is well-structured with:
- URL-based versioning (`/api/v1/`)
- TypeScript-typed endpoints via `@rocket.chat/rest-typings`
- Rate limiting and CORS support
- Authentication via custom headers

GraphQL was previously available but has been removed. No gRPC support exists in the current codebase.
