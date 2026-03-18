# Part 9: Error Handling

## 9.1 HTTP Status Codes

Rocket.Chat uses standardized HTTP status codes to indicate the outcome of API requests. The following status codes are defined in the API type definitions:

| Status Code | Meaning | Description |
|-------------|---------|-------------|
| 200 | OK | Request succeeded |
| 201 | Created | Resource created successfully |
| 400 | Bad Request | Invalid request format or validation failure |
| 401 | Unauthorized | Invalid or missing authentication |
| 403 | Forbidden | Insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 422 | Unprocessable | Validation failed |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Error | Server error |
| 503 | Service Unavailable | Service temporarily unavailable |

**Sources:**
- `apps/meteor/app/api/server/definition.ts:17` - ErrorStatusCodes type definition
- `apps/meteor/app/api/server/definition.ts:11` - SuccessStatusCodes type (200-207)
- `apps/meteor/app/api/server/definition.ts:13` - RedirectStatusCodes type (300-307)
- `apps/meteor/app/api/server/definition.ts:15` - AuthorizationStatusCodes type (400-450)
- `apps/meteor/app/api/server/ApiClass.ts:307` - failure() returns 400
- `apps/meteor/app/api/server/ApiClass.ts:338` - notFound() returns 404
- `apps/meteor/app/api/server/ApiClass.ts:368` - unauthorized() returns 401
- `apps/meteor/app/api/server/ApiClass.ts:378` - forbidden() returns 403
- `apps/meteor/app/api/server/ApiClass.ts:390` - tooManyRequests() returns 429
- `apps/meteor/app/api/server/ApiClass.ts:348` - internalError() returns 500
- `apps/meteor/app/api/server/ApiClass.ts:358` - unavailable() returns 503

---

## 9.2 Error Response Format

Rocket.Chat implements a standardized error response structure across its API. Error responses follow a consistent JSON format.

### Standard Error Response Structure

```json
{
  "success": false,
  "error": "error-message",
  "errorType": "error-type",
  "details": {},
  "message": "Human readable message",
  "stack": "Error stack trace (optional)"
}
```

### Response Type Definitions

**FailureResult (400):**
```typescript
type FailureResult<T, TStack = undefined, TErrorType = undefined, TErrorDetails = undefined> = {
    statusCode: 400;
    body: { success: false } & T & {
        error?: T;
        stack?: TStack;
        errorType?: TErrorType;
        details?: TErrorDetails;
        message?: string;
    };
};
```
Source: `apps/meteor/app/api/server/definition.ts:24-37`

**UnauthorizedResult (401):**
```typescript
type UnauthorizedResult<T> = {
    statusCode: 401;
    body: { success: false; error: T | 'unauthorized' };
};
```
Source: `apps/meteor/app/api/server/definition.ts:44-50`

**ForbiddenResult (403):**
```typescript
type ForbiddenResult<T> = {
    statusCode: 403;
    body: { success: false; error: T | 'forbidden' | 'unauthorized' };
};
```
Source: `apps/meteor/app/api/server/definition.ts:52-59`

**NotFoundResult (404):**
```typescript
type NotFoundResult<T = string> = {
    statusCode: 404;
    body: { success: false; error: T };
};
```
Source: `apps/meteor/app/api/server/definition.ts:85-91`

**TooManyRequestsResult (429):**
```typescript
type TooManyRequestsResult<T> = {
    statusCode: 429;
    body: { success: false; error: T | 'Too many requests' };
};
```
Source: `apps/meteor/app/api/server/definition.ts:61-67`

**InternalError (500):**
```typescript
type InternalError<T, StatusCode extends ErrorStatusCodes = 500, D = 'Internal server error'> = {
    statusCode: StatusCode;
    body: { error: T | D; success: false };
};
```
Source: `apps/meteor/app/api/server/definition.ts:69-75`

### Rate Limit Headers

When rate limiting is enforced, the following headers are included in responses:
- `X-RateLimit-Limit`: Maximum requests allowed in the window
- `X-RateLimit-Remaining`: Requests remaining in current window
- `X-RateLimit-Reset`: Unix timestamp when the rate limit resets

Source: `apps/meteor/app/api/server/ApiClass.ts:425-430`

---

## 9.3 Error Codes

Rocket.Chat uses application-specific error codes for detailed error identification. Error codes are typically prefixed with `error-`.

### Common Error Codes

| Error Code | HTTP Status | Description |
|------------|-------------|-------------|
| `error-user-param-not-provided` | 400 | Required user parameter missing |
| `error-room-param-not-provided` | 400 | Required room parameter missing |
| `error-room-not-found` | 404 | Room does not exist |
| `error-invalid-user` | 400 | Invalid user credentials or ID |
| `error-invalid-param` | 400 | Invalid parameter value |
| `error-param-required` | 400 | Required parameter missing |
| `error-action-not-allowed` | 403 | Action not permitted |
| `error-not-allowed` | 403 | Operation not allowed |
| `error-not-authorized` | 403 | User not authorized |
| `error-unauthorized` | 401 | User not authenticated |
| `error-too-many-requests` | 429 | Rate limit exceeded |
| `error-roomId-param-invalid` | 400 | Invalid room ID format |
| `error-creating-custom-user-status` | 400 | Failed to create custom user status |
| `error-importer-not-defined` | 400 | Importer not found |
| `error-endpoint-disabled` | 403 | Endpoint is disabled |
| `error-email-send-failed` | 500 | Email sending failed |
| `error-invalid-sort-parameter` | 400 | Invalid sort parameter |
| `error-avatar-invalid-url` | 400 | Invalid avatar URL |
| `error-file-too-large` | 400 | Uploaded file exceeds size limit |
| `error-no-file` | 400 | No file uploaded |
| `error-invalid-email-inbox` | 400 | Invalid email inbox configuration |
| `totp-required` | 400 | Two-factor authentication required |
| `totp-user-not-found` | 404 | TOTP user not found |
| `integration-type-must-be-outgoing` | 400 | Integration type must be outgoing webhook |
| `history-data-must-be-defined` | 400 | History data required for replay |

### Client-Side Error Classes

Rocket.Chat provides typed error classes that extend the base `RocketChatError`:

```typescript
export abstract class RocketChatError<TErrorId extends string, TDetails = unknown> extends Error {
    public readonly error: TErrorId;
    public readonly reason?: string;
    public readonly details?: TDetails;
}
```

**Error Class Examples:**
- `InvalidCommandUsage` - Invalid command usage
- `InvalidPreview` - Invalid preview
- `InvalidUrlError` - Invalid URL
- `NotAuthorizedError` - Not authorized
- `NotSubscribedToRoomError` - User not subscribed to room
- `RoomNotFoundError` - Room not found
- `VisitorDoesNotExistError` - Visitor does not exist
- `PinMessagesNotAllowed` - Pinning messages not allowed
- `UiKitTriggerTimeoutError` - UI Kit trigger timeout

Source: `apps/meteor/client/lib/errors/RocketChatError.ts:1-14`

**Sources:**
- `apps/meteor/app/api/server/v1/channels.ts:86` - error-room-not-found
- `apps/meteor/app/api/server/v1/chat.ts:179` - error-param-required
- `apps/meteor/app/api/server/v1/users.ts:198` - error-action-not-allowed
- `apps/meteor/app/api/server/helpers/getUserFromParams.ts:21` - error-user-param-not-provided
- `apps/meteor/app/api/server/helpers/parseJsonQuery.ts:41` - error-invalid-sort-parameter
- `apps/meteor/app/api/server/middlewares/authenticationHono.ts:42` - error-unauthorized
- `apps/meteor/app/api/server/ApiClass.ts:434` - error-too-many-requests
- `apps/meteor/app/2fa/server/code/index.ts:185,210` - totp-required, totp-user-not-found
- `apps/meteor/client/lib/errors/*.ts` - Various error class definitions

---

## 9.4 Retry Logic

Rocket.Chat implements retry strategies primarily for outgoing webhooks and integrations. The retry mechanism supports configurable delays and counts.

### Webhook Retry Configuration

Outgoing webhooks can be configured with the following retry options:

| Setting | Type | Description |
|---------|------|-------------|
| `retryFailedCalls` | boolean | Enable retry on failed calls |
| `retryCount` | number | Maximum number of retry attempts |
| `retryDelay` | string | Retry delay strategy |

### Retry Delay Strategies

Rocket.Chat supports three retry delay strategies:

1. **powers-of-ten**: Exponential backoff with powers of 10
   - Delays: 0.1s, 1s, 10s, 1m40s, 16m40s, 2h46m40s, etc.
   - Formula: `Math.pow(10, tries + 2)`

2. **powers-of-two**: Exponential backoff with powers of 2
   - Delays: 2s, 4s, 8s, 16s, etc.
   - Formula: `Math.pow(2, tries + 1) * 1000`

3. **increments-of-two**: Linear increments
   - Delays: 2s, 4s, 6s, 8s, etc.
   - Formula: `(tries + 1) * 2 * 1000`

### Retry Logic Implementation

```typescript
if (trigger.retryFailedCalls && trigger.retryCount) {
    if (tries < trigger.retryCount && trigger.retryDelay) {
        let waitTime;
        switch (trigger.retryDelay) {
            case 'powers-of-ten':
                waitTime = Math.pow(10, tries + 2);
                break;
            case 'powers-of-two':
                waitTime = Math.pow(2, tries + 1) * 1000;
                break;
            case 'increments-of-two':
                waitTime = (tries + 1) * 2 * 1000;
                break;
        }
        setTimeout(() => {
            void this.executeTriggerUrl(url, trigger, { event, message, room, owner, user }, tries + 1);
        }, waitTime);
    }
}
```

Source: `apps/meteor/app/integrations/server/lib/triggerHandler.ts:722-755`

### Client-Side Retry

React Query is used for data fetching with automatic retry:

```typescript
// Default retry behavior in queryClient
retry: process.env.TEST_MODE === 'true'

// Custom retry for room info
retry: (count, error: { success: boolean; error: string }) => count <= 2 && error.error !== 'not-allowed'
```

Source: `apps/meteor/client/lib/queryClient.ts:7`
Source: `apps/meteor/client/hooks/useRoomInfoEndpoint.ts:45`

---

## 9.5 Timeout Handling

Rocket.Chat implements timeout configurations at multiple levels for HTTP requests.

### Default Timeout Settings

| Context | Default Timeout | Configurable |
|---------|----------------|--------------|
| General HTTP calls | 20 seconds (20000ms) | Yes, via `HTTP_DEFAULT_TIMEOUT` env var |
| Apps HTTP Bridge | 3 minutes (180000ms) | Yes, per-request |
| Livechat HTTP | Configurable | Yes, via settings |

### HTTP Call Timeout

The default HTTP timeout is 20 seconds and can be overridden via environment variable:

```typescript
const envTimeout = parseInt(process.env.HTTP_DEFAULT_TIMEOUT || '', 10);
const defaultTimeout = !isNaN(envTimeout) ? envTimeout : 20000;
```

Source: `apps/meteor/server/lib/http/call.ts:10-13`

Timeout can be specified per-request:

```typescript
if (!('timeout' in options)) {
    options.timeout = defaultTimeout;
}
```

Source: `apps/meteor/server/lib/http/call.ts:79-81`

### Apps Engine HTTP Bridge Timeout

The Apps engine HTTP bridge uses a 3-minute default timeout:

```typescript
// Previously, there was no timeout for HTTP requests.
// We're setting the default timeout now to 3 minutes as it
// seems to be a good balance
const DEFAULT_TIMEOUT = 3 * 60 * 1000;

const timeout = request.timeout || DEFAULT_TIMEOUT;
```

Source: `apps/meteor/app/apps/server/bridges/http.ts:11-14,71`

### Livechat HTTP Timeout

Livechat webhooks have a configurable HTTP timeout:

```typescript
const timeout = settings.get<number>('Livechat_http_timeout');
```

Source: `apps/meteor/app/livechat/server/lib/webhooks.ts:18`

The setting can be configured via:
- `apps/meteor/app/livechat/server/api/v1/integration.ts:38` - Livechat_http_timeout setting
- `apps/meteor/app/livechat/server/api/lib/integrations.ts:8` - Setting reference

### Timeout Configuration Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `HTTP_DEFAULT_TIMEOUT` | 20000ms | General HTTP request timeout |
| `Livechat_http_timeout` | Configurable | Livechat webhook timeout |
| `Livechat_visitor_inactivity_timeout` | Configurable | Visitor inactivity timeout |
| `Livechat_agent_leave_action_timeout` | Configurable | Agent leave action timeout |
| `Livechat_auto_transfer_chat_timeout` | Configurable | Auto transfer chat timeout |

---

## Summary

Rocket.Chat implements a comprehensive error handling system:

1. **HTTP Status Codes**: Standard HTTP status codes (400, 401, 403, 404, 429, 500, 503) with typed TypeScript definitions
2. **Error Response Format**: Consistent JSON structure with `success: false`, error messages, error types, and optional details
3. **Error Codes**: Extensive application-specific error codes prefixed with `error-` for precise error identification
4. **Retry Logic**: Configurable retry strategies for outgoing webhooks with exponential backoff options (powers-of-ten, powers-of-two, increments-of-two)
5. **Timeout Handling**: Multi-level timeout configuration - 20 seconds default for HTTP calls, 3 minutes for Apps engine, with environment variable and per-request overrides
