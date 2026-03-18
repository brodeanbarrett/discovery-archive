# Part 8: Rate Limiting & Quotas

This section documents Rocket.Chat's rate limiting and quota mechanisms for API access.

## 1. Rate Limit Headers

Rocket.Chat includes standard rate limit headers in REST API responses:

| Header | Description |
|--------|-------------|
| `X-RateLimit-Limit` | Maximum number of requests allowed in the time window |
| `X-RateLimit-Remaining` | Number of requests remaining in the current time window |
| `X-RateLimit-Reset` | Unix timestamp when the rate limit resets |

**Source:** `apps/meteor/app/api/server/ApiClass.ts` (lines 425-430)
```typescript
response.headers.set(
	'X-RateLimit-Limit',
	String(rateLimiterDictionary[objectForRateLimitMatch.route].options.numRequestsAllowed ?? ''),
);
response.headers.set('X-RateLimit-Remaining', String(attemptResult.numInvocationsLeft));
response.headers.set('X-RateLimit-Reset', String(new Date().getTime() + attemptResult.timeToReset));
```

## 2. Limit Values

### REST API Default Limits

| Setting | Default Value | Description |
|---------|---------------|-------------|
| `API_Enable_Rate_Limiter` | `true` | Master switch for rate limiting |
| `API_Enable_Rate_Limiter_Dev` | `true` | Enable rate limiting in development mode |
| `API_Enable_Rate_Limiter_Limit_Calls_Default` | `10` | Default requests per window |
| `API_Enable_Rate_Limiter_Limit_Time_Default` | `60000` | Default time window in milliseconds (1 minute) |

**Source:** `apps/meteor/server/settings/rate.ts` (lines 57-70)
```typescript
await this.section('API_Rate_Limiter', async function () {
	await this.add('API_Enable_Rate_Limiter', true, { type: 'boolean' });
	await this.add('API_Enable_Rate_Limiter_Dev', true, {
		type: 'boolean',
		enableQuery: { _id: 'API_Enable_Rate_Limiter', value: true },
	});
	await this.add('API_Enable_Rate_Limiter_Limit_Calls_Default', 10, {
		type: 'int',
		enableQuery: { _id: 'API_Enable_Rate_Limiter', value: true },
	});
	await this.add('API_Enable_Rate_Limiter_Limit_Time_Default', 60000, {
		type: 'int',
		enableQuery: { _id: 'API_Enable_Rate_Limiter', value: true },
	});
});
```

### DDP Rate Limiting

The DDP (Distributed Data Protocol) has multiple rate limit configurations:

| Setting | Default Value | Time Window |
|---------|---------------|-------------|
| `DDP_Rate_Limit_IP_Requests_Allowed` | 120,000 | 60,000ms |
| `DDP_Rate_Limit_User_Requests_Allowed` | 1,200 | 60,000ms |
| `DDP_Rate_Limit_Connection_Requests_Allowed` | 600 | 60,000ms |
| `DDP_Rate_Limit_User_By_Method_Requests_Allowed` | 20 | 10,000ms |
| `DDP_Rate_Limit_Connection_By_Method_Requests_Allowed` | 10 | 10,000ms |

**Source:** `apps/meteor/server/settings/rate.ts` (lines 5-55)

### Per-Endpoint Custom Limits

Individual endpoints can override default rate limits using `rateLimiterOptions`:

**Source:** `apps/meteor/app/api/server/definition.ts` (lines 115-120)
```typescript
rateLimiterOptions?:
	| {
			numRequestsAllowed?: number;
			intervalTimeInMS?: number;
	  }
	| boolean;
```

**Examples of custom limits:**
- `apps/meteor/app/api/server/v1/calendar.ts` - 3 requests per 1000ms
- `apps/meteor/app/api/server/v1/mailer.ts` - 1 request per 60000ms
- `apps/meteor/app/api/server/v1/cloud.ts` - 2 requests per 60000ms

## 3. Burst Handling

### UI Kit Burst Allowance

The UI Kit server uses `express-rate-limit` with a multiplied limit for burst handling:

**Source:** `apps/meteor/ee/server/apps/communication/uikit.ts` (lines 48-58)
```typescript
// use specific rate limit of 600 (which is 60 times the default limits) requests per minute (around 10/second)
const apiLimiter = rateLimit({
	windowMs: settings.get('API_Enable_Rate_Limiter_Limit_Time_Default'),
	max: (settings.get('API_Enable_Rate_Limiter_Limit_Calls_Default') as number) * 60,
	skip: () =>
		settings.get('API_Enable_Rate_Limiter') !== true ||
		(process.env.NODE_ENV === 'development' && settings.get('API_Enable_Rate_Limiter_Dev') !== true),
});
```

### DDP Slow Down Mechanism

The DDP rate limiter includes a "slow down" feature that adds delays when limits are exceeded:

**Source:** `apps/meteor/app/lib/server/startup/rateLimiter.js` (lines 13, 137-140)
```javascript
const slowDownRate = parseInt(process.env.RATE_LIMITER_SLOWDOWN_RATE);
// ...
if (slowDownRate > 0 && reply.numInvocationsExceeded) {
	await sleep(slowDownRate * reply.numInvocationsExceeded);
}
```

## 4. Quota Management

Rocket.Chat does **not** implement daily or monthly quota limits in its API layer. Rate limiting is purely time-window based.

### License-Based Feature Limits

While not API rate limiting, Rocket.Chat's license system enforces workspace-level quotas:

- `monthlyActiveContacts` - Monthly active contacts limit for Livechat

**Source:** `apps/meteor/ee/app/license/server/startup.ts` (line 115)
```typescript
License.setLicenseLimitCounter('monthlyActiveContacts', () => LivechatContacts.countContactsOnPeriod(moment.utc().format('YYYY-MM')));
```

## 5. Throttling Responses

### 429 Too Many Requests Response

When rate limits are exceeded, the API returns a 429 status code:

**Source:** `apps/meteor/app/api/server/ApiClass.ts` (lines 389-397, 432-441)
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
// ...
if (!attemptResult.allowed) {
	throw new Meteor.Error(
		'error-too-many-requests',
		`Error, too many requests. Please slow down. You must wait ${timeToResetAttempsInSeconds} seconds before trying this endpoint again.`,
		{
			timeToReset: attemptResult.timeToReset,
			seconds: timeToResetAttempsInSeconds,
		},
	);
}
```

### Error Response Format

```json
{
	"success": false,
	"error": "Error, too many requests. Please slow down. You must wait X seconds before trying this endpoint again."
}
```

The error includes:
- `timeToReset` - milliseconds until the rate limit resets
- `seconds` - human-readable seconds until retry

### Rate Limit Bypass Permission

Users with the `api-bypass-rate-limit` permission are exempt from rate limiting:

**Source:** `apps/meteor/app/api/server/ApiClass.ts` (lines 403-409)
```typescript
protected async shouldVerifyRateLimit(route: string, userId?: string): Promise<boolean> {
	return (
		rateLimiterDictionary.hasOwnProperty(route) &&
		settings.get<boolean>('API_Enable_Rate_Limiter') === true &&
		(process.env.NODE_ENV !== 'development' || settings.get<boolean>('API_Enable_Rate_Limiter_Dev') === true) &&
		!(userId && (await hasPermissionAsync(userId, 'api-bypass-rate-limit')))
	);
}
```

### Metrics

Rate limit violations are tracked via metrics:

**Source:** `apps/meteor/app/metrics/server/lib/metrics.ts` (line 87)
```typescript
ddpRateLimitExceeded: new client.Counter({
	name: 'rocketchat_ddp_rate_limit_exceeded_total',
	help: 'Total number of times a DDP rate limit was exceeded',
}),
```

**Source:** `apps/meteor/app/lib/server/startup/rateLimiter.js` (lines 126-136)
```typescript
if (reply.allowed === false) {
	rateLimiterLog({ msg, reply, input });
	metrics.ddpRateLimitExceeded.inc({
		limit_name: name,
		user_id: input.userId,
		client_address: input.clientAddress,
		type: input.type,
		name: input.name,
		connection_id: input.connectionId,
	});
	// ...
}
```

## Summary

Rocket.Chat implements a comprehensive rate limiting system:

1. **Headers**: Standard `X-RateLimit-*` headers inform clients of their limits
2. **Limits**: Default 10 requests/minute for REST API; configurable per-endpoint
3. **Burst**: UI Kit allows bursts up to 60x the default; DDP has separate IP/user/method limits
4. **Quotas**: No daily/monthly quotas; purely time-window based
5. **Throttling**: Returns HTTP 429 with informative error messages and retry guidance
