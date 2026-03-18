# Part 2: API Endpoints & Operations

This document catalogs the REST API endpoints available in Rocket.Chat, including HTTP methods, parameters, authentication requirements, and operational patterns.

## 2.1 Endpoint Inventory

Rocket.Chat provides a comprehensive REST API under the `/api/v1/` base path. The API is implemented using a custom routing system built on the `@rocket.chat/http-router` package.

### Core API Endpoints (v1)

The following endpoint modules are available in `apps/meteor/app/api/server/v1/`:

| Module | File | Description |
|--------|------|-------------|
| channels | `channels.ts` | Channel/room management operations |
| chat | `chat.ts` | Message posting and reactions |
| commands | `commands.ts` | Slash commands |
| custom-user-status | `custom-user-status.ts` | User status management |
| groups | `groups.ts` | Private group operations |
| im | `im.ts` | Direct message operations |
| mailer | `mailer.ts` | Email operations |
| misc | `misc.ts` | Miscellaneous endpoints (directory, spotlight, etc.) |
| push | `push.ts` | Push notification settings |
| rooms | `rooms.ts` | General room operations |
| stats | `stats.ts` | Statistics and monitoring |
| teams | `teams.ts` | Team management |
| users | `users.ts` | User management |
| assets | `assets.ts` | Asset management |
| autotranslate | `autotranslate.ts` | Auto-translation settings |
| banners | `banners.ts` | Banner management |
| calendar | `calendar.ts` | Calendar integration |
| call-history | `call-history.ts` | Video call history |
| cloud | `cloud.ts` | Cloud workspace operations |
| custom-sounds | `custom-sounds.ts` | Custom sound management |
| e2e | `e2e.ts` | End-to-end encryption |
| email-inbox | `email-inbox.ts` | Email inbox configuration |
| emoji-custom | `emoji-custom.ts` | Custom emoji management |
| imports | `import.ts` | Data import operations |
| instances | `instances.ts` | Instance information |
| integrations | `integrations.ts` | Integration management |
| invites | `invites.ts` | Invitation management |
| ldap | `ldap.ts` | LDAP configuration |
| oauthapps | `oauthapps.ts` | OAuth application management |
| permissions | `permissions.ts` | Permission management |
| presence | `presence.ts` | User presence |
| roles | `roles.ts` | Role management |
| settings | `settings.ts` | Settings management |
| subscriptions | `subscriptions.ts` | Subscription management |
| uploads | `uploads.ts` | File upload operations |
| videoConference | `videoConference.ts` | Video conference management |
| webdav | `webdav.ts` | WebDAV integration |

**Source:** `apps/meteor/app/api/server/v1/` directory structure

### Default API Endpoints

Located in `apps/meteor/app/api/server/default/`:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/info` | GET | Server information (no auth required) |
| `/api/docs/json` | GET | OpenAPI specification |

**Source:** `apps/meteor/app/api/server/default/info.ts:4-12`, `apps/meteor/app/api/server/default/openApi.ts:75-85`

### Livechat API Endpoints

Rocket.Chat provides separate API endpoints for Livechat operations:

#### Livechat API v1 (`apps/meteor/app/livechat/server/api/v1/`)

| Module | Description |
|--------|-------------|
| agent | Agent management |
| config | Livechat configuration |
| contact | Contact management |
| customField | Custom field management |
| integration | Integration settings |
| message | Livechat message operations |
| offlineMessage | Offline message handling |
| pageVisited | Page visit tracking |
| room | Livechat room operations |
| statistics | Livechat statistics |
| transcript | Transcript operations |
| transfer | Chat transfer operations |
| visitor | Visitor management |
| webhooks | Webhook configuration |

#### Livechat REST Imports (`apps/meteor/app/livechat/imports/server/rest/`)

Additional Livechat endpoints for:
- agent, appearance, businessHours, dashboards, departments
- inquiries, integrations, queue, rooms, sms, triggers
- upload, users, visitors

**Source:** `apps/meteor/app/livechat/server/api/v1/`, `apps/meteor/app/livechat/imports/server/rest/`

---

## 2.2 Path Parameters

Path parameters are embedded directly in the URL path. The API uses a consistent pattern for resource identification.

### Common Path Parameter Patterns

| Parameter | Type | Description | Example |
|-----------|------|-------------|---------|
| `roomId` | string | Unique room identifier | `/api/v1/channels.info?roomId=abc123` |
| `roomName` | string | Room name (slug) | `/api/v1/channels.info?roomName=general` |
| `userId` | string | Unique user identifier | `/api/v1/users.info?userId=xyz789` |
| `username` | string | Username | `/api/v1/users.info?username=john` |
| `token` | string | Authentication or visitor token | `/livechat/visitor/:token` |
| `rid` | string | Room ID (short form) | `/livechat/agent.info/:rid/:token` |
| `id` | string | Generic identifier | `/api/v1/banners/:id` |

### Path Parameter Extraction

Path parameters are extracted using the `findChannelByIdOrName` helper function which supports both ID and name-based lookups:

```typescript
// From apps/meteor/app/api/server/v1/channels.ts:61-98
async function findChannelByIdOrName({
    params,
    userId,
    checkedArchived = true,
}: {
    params: { roomId?: string } | { roomName?: string };
    userId?: string;
    checkedArchived?: boolean;
}): Promise<IRoom>
```

**Source:** `apps/meteor/app/api/server/v1/channels.ts:61-98`

---

## 2.3 Query Parameters

Query parameters are documented in type definitions in the `@rocket.chat/rest-typings` package.

### Standard Pagination Parameters

All list endpoints support these pagination parameters:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `count` | number | 50 (configurable) | Number of items to return |
| `offset` | number | 0 | Number of items to skip |
| `sort` | string | - | Sort specification (JSON format) |
| `query` | string | - | Query filter (JSON format, deprecated) |
| `fields` | string | - | Field projection (JSON format, deprecated) |

### Pagination Helper

The pagination is processed by `getPaginationItems`:

```typescript
// From apps/meteor/app/api/server/helpers/getPaginationItems.ts:6-37
export async function getPaginationItems(params: {
    offset?: string | number | null;
    count?: string | number | null;
}): Promise<{
    readonly offset: number;
    readonly count: number;
}>
```

Default values are controlled by settings:
- `API_Default_Count` (default: 50)
- `API_Upper_Count_Limit` (default: 100)
- `API_Allow_Infinite_Count` (allows count=0 for unlimited)

**Source:** `apps/meteor/app/api/server/helpers/getPaginationItems.ts:1-37`

### Example: Channels List Parameters

```typescript
// From packages/rest-typings/src/v1/channels/ChannelsListProps.ts
export type ChannelsListProps = PaginatedRequest<{ _id?: string }>;

const channelsListPropsSchema = {
    type: 'object',
    properties: {
        _id: { type: 'string' },
        query: { type: 'string' },
        count: { type: 'number' },
        offset: { type: 'number' },
        sort: { type: 'string' },
    },
    required: [],
    additionalProperties: false,
};
```

**Source:** `packages/rest-typings/src/v1/channels/ChannelsListProps.ts:1-29`

---

## 2.4 Request Headers

### Authentication Headers

Rocket.Chat uses custom headers for authentication:

| Header | Required | Description |
|--------|----------|-------------|
| `X-User-Id` | Conditional | User identifier |
| `X-Auth-Token` | Conditional | Authentication token |

These are defined in the OpenAPI specification:

```typescript
// From apps/meteor/app/api/server/default/openApi.ts:57-68
components: {
    securitySchemes: {
        userId: {
            type: 'apiKey',
            in: 'header',
            name: 'X-User-Id',
        },
        authToken: {
            type: 'apiKey',
            in: 'header',
            name: 'X-Auth-Token',
        },
    },
}
```

**Source:** `apps/meteor/app/api/server/default/openApi.ts:57-68`

### Standard Headers

The API also handles standard HTTP headers through the Express middleware:
- `Content-Type`: JSON content type
- `User-Agent`: Client identification
- Request IP address (for rate limiting and audit)

**Source:** `apps/meteor/app/api/server/ApiClass.ts:148-180`

### Endpoint Options

Each endpoint can specify authentication requirements via options:

```typescript
// From apps/meteor/app/api/server/v1/channels.ts:100-118
API.v1.addRoute(
    'channels.addAll',
    {
        authRequired: true,
        validateParams: isChannelsAddAllProps,
    },
    {
        async post() { /* ... */ },
    },
);
```

**Source:** `apps/meteor/app/api/server/v1/channels.ts:100-118`

| Option | Description |
|--------|-------------|
| `authRequired` | Boolean, whether authentication is required |
| `twoFactorRequired` | Boolean, requires two-factor authentication |
| `rateLimiterOptions` | Rate limiting configuration |
| `permissionsRequired` | Array of required permissions |
| `validateParams` | (Deprecated) Parameter validation schema |

**Source:** `apps/meteor/app/api/server/definition.ts:104-149`

---

## 2.5 Operation Types

### CRUD Operations

The API follows RESTful patterns with standard HTTP methods:

| Method | Operation Type | Usage |
|--------|----------------|-------|
| `GET` | Read | Retrieve resources, lists, history |
| `POST` | Create | Create resources, execute actions |
| `PUT` | Update | Update existing resources |
| `DELETE` | Delete | Delete resources |

### Custom Actions

Many endpoints perform custom actions beyond standard CRUD:

| Endpoint Pattern | Action |
|------------------|-------|
| `channels.addAll` | Add all users to channel |
| `channels.archive` | Archive a channel |
| `channels.unarchive` | Unarchive a channel |
| `channels.join` | Join a channel |
| `channels.leave` | Leave a channel |
| `channels.kick` | Remove user from channel |
| `users.create` | Create new user |
| `users.update` | Update user information |
| `users.delete` | Delete user |
| `chat.sendMessage` | Send a message |
| `chat.react` | React to a message |

### Subscription Operations

Rocket.Chat supports real-time subscriptions via DDP, but the REST API also provides subscription management:

| Endpoint | Description |
|----------|-------------|
| `subscriptions.get` | Get user subscriptions |
| `subscriptions.update` | Update subscription settings |

**Source:** `apps/meteor/app/api/server/v1/subscriptions.ts`

---

## 2.6 Pagination

### Pagination Strategy

Rocket.Chat uses offset-based pagination with configurable defaults.

### Pagination Flow

1. Client sends `count` and `offset` query parameters
2. Server validates against configured limits
3. Server returns results with pagination metadata

### Implementation

```typescript
// From apps/meteor/app/api/server/helpers/getPaginationItems.ts:25-31
if (count > hardUpperLimit) {
    count = hardUpperLimit;
}

if (count === 0 && !settings.get('API_Allow_Infinite_Count')) {
    count = defaultCount;
}
```

### Settings-Controlled Pagination

| Setting | Default | Description |
|---------|---------|-------------|
| `API_Default_Count` | 50 | Default number of items |
| `API_Upper_Count_Limit` | 100 | Maximum items per request |
| `API_Allow_Infinite_Count` | false | Allow unlimited results |

**Source:** `apps/meteor/app/api/server/helpers/getPaginationItems.ts:10-14`

---

## 2.7 Filtering & Sorting

### Sorting

Sort parameters are passed as JSON:

```
GET /api/v1/channels.list?sort={"name":1}
```

Where `1` is ascending and `-1` is descending.

```typescript
// From apps/meteor/app/api/server/helpers/parseJsonQuery.ts:36-56
let sort;
if (typeof params?.sort === 'string') {
    try {
        sort = JSON.parse(params.sort);
        Object.entries(sort).forEach(([key, value]) => {
            if (value !== 1 && value !== -1) {
                throw new Meteor.Error('error-invalid-sort-parameter', ...);
            }
        });
    } catch (e) {
        // Error handling
    }
}
```

**Source:** `apps/meteor/app/api/server/helpers/parseJsonQuery.ts:36-56`

### Filtering

Query filtering is available but deprecated for security reasons:

```
GET /api/v1/users.list?query={"active":true}
```

The `query` and `fields` parameters require enabling the environment variable:
```
ALLOW_UNSAFE_QUERY_AND_FIELDS_API_PARAMS=true
```

```typescript
// From apps/meteor/app/api/server/helpers/parseJsonQuery.ts:58-84
const isUnsafeQueryParamsAllowed = process.env.ALLOW_UNSAFE_QUERY_AND_FIELDS_API_PARAMS?.toUpperCase() === 'TRUE';

let fields: Record<string, 0 | 1> | undefined;
if (typeof params?.fields === 'string' && isUnsafeQueryParamsAllowed) {
    // Parse and validate fields
}
```

**Source:** `apps/meteor/app/api/server/helpers/parseJsonQuery.ts:58-84`

### Query Operations Whitelist

Allowed MongoDB query operators are restricted:

```typescript
// From apps/meteor/app/api/server/helpers/parseJsonQuery.ts:12-15
const pathAllowConf = {
    '/api/v1/users.list': ['$or', '$regex', '$and'],
    'def': ['$or', '$and', '$regex'],
};
```

**Source:** `apps/meteor/app/api/server/helpers/parseJsonQuery.ts:12-15`

---

## 2.8 API Versioning

### Current Versions

| Version | Base Path | Status |
|---------|-----------|--------|
| v1 | `/api/v1/` | Active |
| default | `/api/` | Active (info only) |
| Livechat | `/livechat/` | Active |

### Deprecation Notes

The following are deprecated:

1. **`addRoute` method** - Use new route registration methods (get, post, put, delete)
   - **Source:** `apps/meteor/app/api/server/ApiClass.ts:735-769`

2. **`validateParams` option** - Use `query` and/or `body` properties instead
   - **Source:** `apps/meteor/app/api/server/definition.ts:147`

3. **`query` and `fields` parameters** - Require `ALLOW_UNSAFE_QUERY_AND_FIELDS_API_PARAMS` environment variable
   - **Source:** `apps/meteor/app/api/server/helpers/parseJsonQuery.ts:20-24`

---

## 2.9 Summary

Rocket.Chat provides a comprehensive REST API with:

- **200+ endpoints** across core, Livechat, and enterprise modules
- **Consistent patterns** for authentication, pagination, and error handling
- **Type-safe** parameter validation using AJV schemas
- **Configurable** rate limiting and pagination limits
- **OpenAPI/Swagger** documentation available at `/api/docs/json`

The API follows RESTful conventions while providing rich functionality for channel management, user administration, messaging, and real-time collaboration features.
