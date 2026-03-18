# Part 4: Request & Response Formats

This section documents the request and response formats used by the Rocket.Chat API.

## 4.1 Request Formats

The Rocket.Chat REST API primarily supports the following request body formats:

### 4.1.1 JSON (Primary Format)

JSON is the primary and most widely used format for request bodies.

**Source**: `apps/meteor/app/api/server/ApiClass.ts` (line 597)
```typescript
'application/json': 'schema' in schema ? { schema: schema.schema } : schema,
```

**Source**: `apps/meteor/app/api/server/v1/banners.ts` (line 59)
```typescript
 *            application/json:
```

**Source**: `apps/meteor/app/api/server/v1/e2e.ts` (line 485)
```typescript
 *          application/json:
```

### 4.1.2 Form Data (application/x-www-form-urlencoded)

Used for specific endpoints, particularly in OAuth and cloud integrations.

**Source**: `apps/meteor/app/cloud/server/functions/finishOAuthAuthorization.ts` (line 27)
```typescript
headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
```

**Source**: `apps/meteor/app/cloud/server/functions/getWorkspaceAccessTokenWithScope.ts` (line 59)
```typescript
headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
```

**Source**: `apps/meteor/app/custom-oauth/server/custom_oauth_server.js` (line 118)
```typescript
'Content-Type': 'application/x-www-form-urlencoded',
```

### 4.1.3 Multipart Form Data

Used for file uploads (avatars, file attachments, emoji uploads, etc.)

**Source**: `apps/meteor/app/api/server/lib/getUploadFormData.spec.ts` (line 40)
```typescript
entries: () => [['content-type', `multipart/form-data; boundary=${boundary}`]],
```

**Source**: `apps/meteor/app/api/server/v1/uploads.ts` - file upload handling via `FileUpload` module

**Source**: `apps/meteor/app/emoji-custom/server/lib/uploadEmojiCustom.ts` - emoji file uploads

### 4.1.4 XML

Not a primary format for REST API. XML is only used for:
- SVG responses for icons (line 298 in `apps/meteor/app/api/server/v1/misc.ts`)
- Potentially in legacy integrations

**Source**: `apps/meteor/app/api/server/v1/misc.ts` (lines 298-300)
```typescript
headers: { 'Content-Type': 'image/svg+xml;charset=utf-8' },
```

### 4.1.5 Protobuf

Not supported in the current REST API implementation.

---

## 4.2 Response Formats

### 4.2.1 JSON (Primary Format)

All REST API endpoints return JSON by default.

**Source**: `apps/meteor/app/api/server/ApiClass.ts` (lines 271-282)
```typescript
public success<T>(result: T = {} as T): SuccessResult<T> {
    if (isObject(result)) {
        (result as Record<string, any>).success = true;
    }

    const finalResult = {
        statusCode: 200,
        body: result,
    } as SuccessResult<T>;

    return finalResult;
}
```

### 4.2.2 Success Response Structure

Successful responses wrap the result in an object with `success: true`:

**Source**: `apps/meteor/app/api/server/definition.ts` (lines 19-22)
```typescript
export type SuccessResult<T, TStatusCode extends SuccessStatusCodes = 200> = {
    statusCode: TStatusCode;
    body: T extends Record<string, unknown> ? { success: true } & T : T;
};
```

Example response:
```json
{
    "success": true,
    "data": { ... }
}
```

### 4.2.3 Content-Type Headers

The API sets appropriate Content-Type headers for responses:

**Source**: `apps/meteor/app/api/server/ApiClass.ts` (line 597)
```typescript
'application/json': 'schema' in schema ? { schema: schema.schema } : schema,
```

---

## 4.3 Data Schemas

### 4.3.1 JSON Schema (AJV)

Rocket.Chat uses [AJV](https://ajv.js.org/) (Another JSON Schema Validator) for request validation.

**Source**: `packages/rest-typings/src/v1/Ajv.ts`

Schema definition example from `packages/rest-typings/src/v1/chat.ts` (lines 11-78):
```typescript
const chatSendMessageSchema = {
    type: 'object',
    properties: {
        message: {
            type: 'object',
            properties: {
                _id: { type: 'string', nullable: true },
                rid: { type: 'string' },
                tmid: { type: 'string', nullable: true },
                msg: { type: 'string', nullable: true },
                // ... more properties
            },
        },
    },
    required: ['message'],
    additionalProperties: false,
};
```

### 4.3.2 TypeScript Types

The `packages/rest-typings` package provides comprehensive TypeScript type definitions for all API endpoints.

**Source**: `packages/rest-typings/src/index.ts`

Example from `packages/rest-typings/src/v1/chat.ts` (lines 889-894):
```typescript
export type ChatEndpoints = {
    '/v1/chat.sendMessage': {
        POST: (params: ChatSendMessage) => {
            message: IMessage;
        };
    };
};
```

### 4.3.3 Core Typings

The `packages/core-typings` package provides interfaces for domain objects:

- `IMessage` - Message interface with timestamps
- `IRoom` - Room/channel interface
- `IUser` - User interface
- `ISubscription` - Subscription interface

**Source**: `packages/core-typings/src/IMessage/IMessage.ts`
**Source**: `packages/core-typings/src/IRoom.ts`

---

## 4.4 Field Naming Convention

### 4.4.1 Mixed Convention

Rocket.Chat uses a mix of naming conventions:

#### camelCase (Primary in REST API params/responses)
Used in API parameter names and response fields:

**Source**: Multiple endpoint definitions in `packages/rest-typings/src/v1/channels/ChannelsInviteProps.ts`
```typescript
export type ChannelsInviteProps = { roomId: string; userId?: string; username?: string; user?: string }
```

**Source**: `packages/rest-typings/src/v1/groups/GroupsCreateProps.ts`
```typescript
export type GroupsCreateProps = {
    name: string;
    members?: string[];
    // ...
}
```

#### snake_case (Database/MongoDB)
Used in database models and some legacy API fields:

**Source**: `packages/core-typings/src/IMessage/IMessage.ts` (lines 167-172)
```typescript
pinnedAt?: Date;
pinnedBy?: Pick<IUser, '_id' | 'username'>;
tlm?: Date;  // timestamp of last message
```

**Source**: `apps/meteor/app/api/server/ApiClass.ts` (lines 117-126)
```typescript
createdAt: number;
_updatedAt: number;
lastLogin: number;
```

#### _id Convention

The `_id` field uses underscore prefix consistently:

**Source**: `packages/rest-typings/src/v1/channels/ChannelsListProps.ts`
```typescript
export type ChannelsListProps = PaginatedRequest<{ _id?: string }>;
```

---

## 4.5 Date/Time Formats

### 4.5.1 ISO 8601

The primary datetime serialization format is ISO 8601 via `toISOString()`.

**Source**: `apps/meteor/app/api/server/v1/im.ts` (line 284)
```typescript
lm = room?.lm ? new Date(room.lm).toISOString() : new Date(room._updatedAt).toISOString();
```

**Source**: `apps/meteor/app/api/server/v1/im.ts` (line 289)
```typescript
unreadsFrom = new Date(subscription.ls).toISOString();
```

### 4.5.2 JavaScript Date Objects

Internal representation uses JavaScript `Date` objects.

**Source**: `packages/core-typings/src/IMessage/IMessage.ts` (lines 167-172)
```typescript
pinnedAt?: Date;
tlm?: Date;
```

**Source**: `packages/core-typings/src/IRoom.ts` (line 65)
```typescript
ts?: Date;
```

### 4.5.3 Unix Timestamps

Some endpoints accept Unix timestamps (as strings or numbers).

**Source**: `apps/meteor/app/api/server/v1/chat.ts` (line 164)
```typescript
ts: Date.now().toString(),
```

### 4.5.4 Date Parsing

The API uses `Date.parse()` for date validation:

**Source**: `apps/meteor/app/api/server/v1/chat.ts` (line 186)
```typescript
if (lastUpdate && isNaN(Date.parse(lastUpdate))) {
```

**Source**: `apps/meteor/app/api/server/v1/chat.ts` (line 845)
```typescript
if (isNaN(Date.parse(updatedSince))) {
```

---

## 4.6 Error Response Schema

### 4.6.1 Standard Error Response Structure

All error responses follow a consistent structure with `success: false`.

**Source**: `apps/meteor/app/api/server/definition.ts` (lines 24-37)
```typescript
export type FailureResult<T, TStack = undefined, TErrorType = undefined, TErrorDetails = undefined> = {
    statusCode: 400;
    body: T extends object
        ? { success: false } & T
        : {
            success: false;
            error?: T;
            stack?: TStack;
            errorType?: TErrorType;
            details?: TErrorDetails;
            message?: string;
        } & (undefined extends TErrorType ? object : { errorType: TErrorType }) &
            (undefined extends TErrorDetails ? object : { details: TErrorDetails extends string ? unknown : TErrorDetails });
};
```

### 4.6.2 HTTP Status Codes

Rocket.Chat uses standard HTTP status codes:

| Status Code | Meaning | Source |
|-------------|---------|--------|
| 200 | Success | `apps/meteor/app/api/server/ApiClass.ts` (line 277) |
| 400 | Bad Request / Failure | `apps/meteor/app/api/server/ApiClass.ts` (line 307) |
| 401 | Unauthorized | `apps/meteor/app/api/server/ApiClass.ts` (line 368) |
| 403 | Forbidden | `apps/meteor/app/api/server/ApiClass.ts` (line 377) |
| 404 | Not Found | `apps/meteor/app/api/server/ApiClass.ts` (line 337) |
| 429 | Too Many Requests | `apps/meteor/app/api/server/ApiClass.ts` (line 389) |
| 500 | Internal Server Error | `apps/meteor/app/api/server/ApiClass.ts` (line 347) |
| 503 | Service Unavailable | `apps/meteor/app/api/server/ApiClass.ts` (line 357) |

### 4.6.3 Error Response Examples

#### Failure Result (400)
**Source**: `apps/meteor/app/api/server/ApiClass.ts` (lines 300-334)
```typescript
public failure<T, TErrorType extends string, TStack extends string, TErrorDetails>(
    result?: T,
    errorType?: TErrorType,
    stack?: TStack,
    error?: { details: TErrorDetails },
): FailureResult<T> {
    const response: {
        statusCode: 400;
        body: any & { message?: string; errorType?: string; stack?: string; success?: boolean; details?: Record<string, any> | string };
    } = { statusCode: 400, body: result };
    // ...
}
```

#### Unauthorized (401)
**Source**: `apps/meteor/app/api/server/ApiClass.ts` (lines 366-374)
```typescript
public unauthorized<T>(msg?: T): UnauthorizedResult<T> {
    return {
        statusCode: 401,
        body: {
            success: false,
            error: msg || 'unauthorized',
        },
    };
}
```

#### Forbidden (403)
**Source**: `apps/meteor/app/api/server/ApiClass.ts` (lines 376-387)
```typescript
public forbidden<T>(msg?: T): ForbiddenResult<T> {
    return {
        statusCode: 403,
        body: {
            success: false,
            error: msg || (applyBreakingChanges ? 'forbidden' : 'unauthorized'),
        },
    };
}
```

#### Not Found (404)
**Source**: `apps/meteor/app/api/server/ApiClass.ts` (lines 336-344)
```typescript
public notFound(msg?: string): NotFoundResult {
    return {
        statusCode: 404,
        body: {
            success: false,
            error: msg || 'Resource not found',
        },
    };
}
```

#### Internal Error (500)
**Source**: `apps/meteor/app/api/server/ApiClass.ts` (lines 346-354)
```typescript
public internalError<T>(msg?: T): InternalError<T> {
    return {
        statusCode: 500,
        body: {
            success: false,
            error: msg || 'Internal server error',
        },
    };
}
```

#### Too Many Requests (429)
**Source**: `apps/meteor/app/api/server/ApiClass.ts` (lines 389-397)
```typescript
public tooManyRequests<T>(msg?: T): TooManyRequestsResult<T> {
    return {
        statusCode: 429,
        body: {
            success: false,
            error: msg || 'Too many requests',
        },
    };
}
```

### 4.6.4 Error Response Format Examples

```json
// 400 Bad Request
{
    "success": false,
    "error": "error-message",
    "errorType": "error-type",
    "stack": "stack-trace-if-available",
    "details": {}
}

// 401 Unauthorized
{
    "success": false,
    "error": "unauthorized"
}

// 403 Forbidden
{
    "success": false,
    "error": "forbidden"
}

// 404 Not Found
{
    "success": false,
    "error": "Resource not found"
}

// 500 Internal Error
{
    "success": false,
    "error": "Internal server error"
}

// 429 Too Many Requests
{
    "success": false,
    "error": "Too many requests"
}
```

---

## Summary

| Aspect | Format |
|--------|--------|
| **Request Body** | JSON (primary), Form URL-encoded, Multipart Form Data |
| **Response Format** | JSON |
| **Schema Validation** | AJV (JSON Schema) + TypeScript |
| **Field Naming** | camelCase (API), snake_case (database) |
| **Date/Time** | ISO 8601 (strings), JavaScript Date objects |
| **Error Response** | Consistent structure with `success: false` + error details |

---

*This report was generated as part of the Rocket.Chat API Discovery Audit.*
