# Part 7: Code Examples

This section documents code examples for Rocket.Chat's API functionality and SDKs, organized by language and use case.

## 1. Language-Specific Examples

### TypeScript/JavaScript - DDP Client SDK

The `@rocket.chat/ddp-client` package provides a TypeScript SDK for both browser and Node.js environments.

**Installation:**
```typescript
// From README.md - packages/ddp-client/README.md (lines 1-6)
import { DDPSDK } from '@rocket.chat/ddp-client';
// or
// yarn add @rocket.chat/ddp-client @rocket.chat/emitter
// npm install @rocket.chat/ddp-client @rocket.chat/emitter
```

### TypeScript/JavaScript - REST API Client

The `@rocket.chat/api-client` package provides a typed REST client:

**Import:**
```typescript
// From packages/api-client/src/index.ts (lines 1-16)
import type { Serialized } from '@rocket.chat/core-typings';
import type {
	MatchPathPattern,
	ParamsFor,
	OperationResult,
	PathFor,
	PathWithoutParamsFor,
	PathWithParamsFor,
} from '@rocket.chat/rest-typings';
import type { Credentials } from './Credentials';
import type { Middleware, RestClientInterface } from './RestClientInterface';
import { RestClient } from './index';
```

## 2. Authentication Examples

### REST API Authentication (Login)

```typescript
// From apps/meteor/tests/data/users.helper.ts (lines 80-95)
export const login = (username: string | undefined, password: string, config?: IRequestConfig): Promise<Credentials> =>
	new Promise((resolve) => {
		const requestInstance = config?.request || request;
		void requestInstance
			.post(api('login'))
			.send({
				user: username,
				password,
			})
			.end((_err: unknown, res: Response) => {
				resolve({
					'X-Auth-Token': res.body.data.authToken,
					'X-User-Id': res.body.data.userId,
				});
			});
	});
```

### DDP Client Authentication

**Login with Password:**
```typescript
// From packages/ddp-client/README.md (lines 54-55)
sdk.account.loginWithPassword(username, hashedPassword)
// Password must be hashed as sha-256
```

**Login with Token:**
```typescript
// From packages/ddp-client/README.md (lines 56-57)
sdk.account.loginWithToken('userTokenGoesHere')
```

**Logout:**
```typescript
// From packages/ddp-client/README.md (lines 58-59)
sdk.account.logout()
```

### REST Client Authentication

```typescript
// From packages/api-client/src/index.ts (lines 66-78)
constructor({ baseUrl, credentials, headers = {} }: { baseUrl: string; credentials?: Credentials; headers?: Record<string, string> }) {
	this.baseUrl = `${baseUrl}/api`;
	this.setCredentials(credentials);
	this.headers = headers;
}

setCredentials: RestClientInterface['setCredentials'] = (credentials) => {
	this.credentials = credentials;
};
```

### API Test Authentication Setup

```typescript
// From apps/meteor/tests/data/api-data.ts (lines 68-83)
export function getCredentials(done?: CallbackHandler) {
	void request
		.post(api('login'))
		.send({
			user: adminUsername,
			password: adminPassword,
		})
		.expect('Content-Type', 'application/json')
		.expect(200)
		.expect((res) => {
			credentials['X-Auth-Token'] = res.body.data.authToken;
			credentials['X-User-Id'] = res.body.data.userId;
			instanceId = res.headers['x-instance-id'];
		})
		.end(done);
}
```

## 3. Basic Operations (CRUD)

### Create - Send Message

```typescript
// From apps/meteor/tests/data/chat.helper.ts (lines 36-47)
export const sendMessage = ({
	message,
	requestCredentials,
}: {
	message: { rid: IRoom['_id']; msg: string } & Partial<Omit<IMessage, 'rid' | 'msg'>>;
	requestCredentials?: Credentials;
}) => {
	return request
		.post(api('chat.sendMessage'))
		.set(requestCredentials ?? credentials)
		.send({ message });
};
```

### Create - User

```typescript
// From apps/meteor/tests/data/users.helper.ts (lines 45-78)
export const createUser = <TUser extends IUser>(
	userData: {
		username?: string;
		email?: string;
		roles?: string[];
		active?: boolean;
		joinDefaultChannels?: boolean;
		verified?: string;
		requirePasswordChange?: boolean;
		name?: string;
		password?: string;
	} = {},
	config?: IRequestConfig,
) =>
	new Promise<TestUser<TUser>>((resolve, reject) => {
		const username = userData.username || `user.test.${Date.now()}.${Math.random()}`;
		const email = userData.email || `${username}@rocket.chat`;
		const requestInstance = config?.request || request;
		const credentialsInstance = config?.credentials || credentials;

		void requestInstance
			.post(api('users.create'))
			.set(credentialsInstance)
			.send({ email, name: username, username, password, ...userData })
			.end((err: unknown, res: Response) => {
				if (err) {
					return reject(err);
				}
				resolve(res.body.user);
			});
	});
```

### Read - Get Current User

```typescript
// From apps/meteor/tests/data/users.helper.ts (lines 123-135)
export const getMe = <TUser extends IUser>(overrideCredential = credentials, config?: IRequestConfig) =>
	new Promise<TestUser<TUser>>((resolve) => {
		const requestInstance = config?.request || request;
		const credentialsInstance = config?.credentials || overrideCredential;
		void requestInstance
			.get(api('me'))
			.set(credentialsInstance)
			.expect('Content-Type', 'application/json')
			.expect(200)
			.end((_end: unknown, res: Response) => {
				resolve(res.body);
			});
	});
```

### Read - Get User by Username

```typescript
// From apps/meteor/tests/data/users.helper.ts (lines 109-121)
export const getUserByUsername = <TUser extends IUser>(username: string, config?: IRequestConfig) =>
	new Promise<TestUser<TUser>>((resolve) => {
		const requestInstance = config?.request || request;
		const credentialsInstance = config?.credentials || credentials;

		void requestInstance
			.get(api('users.info'))
			.query({ username })
			.set(credentialsInstance)
			.end((_err: unknown, res: Response) => {
				resolve(res.body.user);
			});
	});
```

### Update - Update Message

```typescript
// From apps/meteor/tests/data/chat.helper.ts (lines 108-123)
export const updateMessage = ({
	msgId,
	requestCredentials,
	updatedMessage,
	roomId,
}: {
	msgId: IMessage['_id'];
	requestCredentials?: Credentials;
	updatedMessage: string;
	roomId?: IRoom['_id'];
}) => {
	return request
		.post(api('chat.update'))
		.set(requestCredentials ?? credentials)
		.send({ msgId, text: updatedMessage, roomId });
};
```

### Update - Set User Status

```typescript
// From apps/meteor/tests/data/users.helper.ts (lines 137-150)
export const setUserActiveStatus = (userId: IUser['_id'], activeStatus = true, config?: IRequestConfig) =>
	new Promise((resolve) => {
		const requestInstance = config?.request || request;
		const credentialsInstance = config?.credentials || credentials;

		void requestInstance
			.post(api('users.setActiveStatus'))
			.set(credentialsInstance)
			.send({
				userId,
				activeStatus,
			})
			.end(resolve);
	});
```

### Delete - Delete Message

```typescript
// From apps/meteor/tests/data/chat.helper.ts (lines 71-83)
export const deleteMessage = ({ roomId, msgId }: { roomId: IRoom['_id']; msgId: IMessage['_id'] }) => {
	if (!roomId) {
		throw new Error('"roomId" is required in "deleteMessage" test helper');
	}
	if (!msgId) {
		throw new Error('"msgId" is required in "deleteMessage" test helper');
	}

	return request.post(api('chat.delete')).set(credentials).send({
		roomId,
		msgId,
	});
};
```

### Delete - Delete User

```typescript
// From apps/meteor/tests/data/users.helper.ts (lines 97-107)
export const deleteUser = async (user: Pick<IUser, '_id'>, extraData = {}, config?: IRequestConfig) => {
	const requestInstance = config?.request || request;
	const credentialsInstance = config?.credentials || credentials;
	return requestInstance
		.post(api('users.delete'))
		.set(credentialsInstance)
		.send({
			userId: user._id,
			...extraData,
		});
};
```

### REST Client Methods

```typescript
// From packages/api-client/src/index.ts (lines 80-196)
// GET request
async get<TPathPattern extends MatchPathPattern<TPath>, TPath extends PathFor<'GET'>>(
	endpoint: TPath,
	params?: ParamsFor<'GET', TPathPattern>,
	options?: Omit<RequestInit, 'method'>,
): Promise<Serialized<OperationResult<'GET', TPathPattern>>>

// POST request
async post<TPathPattern extends MatchPathPattern<TPath>, TPath extends PathFor<'POST'>>(
	endpoint: TPath,
	params?: ParamsFor<'POST', TPathPattern>,
	{ headers, ...options }: Omit<RequestInit, 'method'> = {},
): Promise<Serialized<OperationResult<'POST', TPathPattern>>>

// PUT request
async put<TPathPattern extends MatchPathPattern<TPath>, TPath extends PathFor<'PUT'>>(
	endpoint: TPath,
	params?: ParamsFor<'PUT', TPathPattern>,
	{ headers, ...options }: Omit<RequestInit, 'method'> = {},
): Promise<Serialized<OperationResult<'PUT', TPathPattern>>>

// DELETE request
async delete<TPathPattern extends MatchPathPattern<TPath>, TPath extends PathFor<'DELETE'>>(
	endpoint: TPath,
	_params?: ParamsFor<'DELETE', TPathPattern>,
	options: Omit<RequestInit, 'method'> = {},
): Promise<Serialized<OperationResult<'DELETE', TPathPattern>>>
```

### DDP Stream Subscriptions

```typescript
// From packages/ddp-client/README.md (lines 78-89)
const messages = new Map([]);
const stream = sdk.stream('room-messages', roomId, (args) => {
    setMessages((messages) => {
       messages.set(args._id, args);
       return new Map(messages);
    });
});

// Stop the stream when you're done
stream.stop();
```

## 4. Error Handling

### HTTP Status Code Handling

```typescript
// From apps/meteor/tests/end-to-end/api/channels.ts (line 111)
.expect(400)  // Bad Request

// From apps/meteor/tests/end-to-end/api/abac.ts (line 69)
.expect(401)  // Unauthorized

// From apps/meteor/tests/end-to-end/api/34-engagement-dashboard.ts (line 48)
.expect(403)  // Forbidden

// From apps/meteor/tests/end-to-end/api/call-history.ts (line 524)
.expect(404)  // Not Found
```

### REST Client Error Handling

```typescript
// From packages/api-client/src/index.ts (lines 208-249)
send(endpoint: string, method: string, { headers, ...options }: Omit<RequestInit, 'method'> = {}): Promise<Response> {
	return fetch(`${this.baseUrl}${'/' + endpoint}'.replace(/\/+/, '/')`, {
		...options,
		headers: { ...this.getCredentialsAsHeaders(), ...this.headers, ...headers },
		method,
	}).then(async (response) => {
		if (response.ok) {
			return response;
		}

		if (response.status !== 400) {
			return Promise.reject(response);
		}

		const clone = response.clone();

		const error = await clone.json();

		if ((isTotpRequiredError(error) || isTotpInvalidError(error)) && hasRequiredTwoFactorMethod(error) && this.twoFactorHandler) {
			// Handle 2FA challenge
			const method2fa = 'details' in error ? error.details.method : 'password';
			const code = await this.twoFactorHandler({
				method: method2fa,
				emailOrUsername: error.details.emailOrUsername,
				invalidAttempt: isTotpInvalidError(error),
			});

			return this.send(endpoint, method, {
				...options,
				headers: {
					...this.getCredentialsAsHeaders(),
					...this.headers,
					...headers,
					'x-2fa-code': code,
					'x-2fa-method': method2fa,
				},
			});
		}

		return Promise.reject(response);
	});
}
```

### DDP Client Error Handling

```typescript
// From packages/ddp-client/README.md (lines 70-71)
await sdk.rest.post('/v1/chat.sendMessage', { message: { rid: id, msg } });
// WARNING: if you wrap a rest call in a try catch block, the error will be of type `Response`.
// By calling `error.json()` you get access to the server error response.
```

## 5. Complete Import Statements

### DDP Client

```typescript
// From packages/ddp-client/README.md (line 16)
import { DDPSDK } from '@rocket.chat/ddp-client';
```

### REST API Client

```typescript
// From packages/api-client/src/index.ts (lines 1-16)
import type { Serialized } from '@rocket.chat/core-typings';
import type {
	MatchPathPattern,
	ParamsFor,
	OperationResult,
	PathFor,
	PathWithoutParamsFor,
	PathWithParamsFor,
} from '@rocket.chat/rest-typings';
import { stringify } from 'query-string';
import type { Credentials } from './Credentials';
import type { Middleware, RestClientInterface } from './RestClientInterface';
import { hasRequiredTwoFactorMethod, isTotpInvalidError, isTotpRequiredError } from './errors';
```

### API Test Setup

```typescript
// From apps/meteor/tests/data/api-data.ts (lines 1-10)
import type { Credentials } from '@rocket.chat/api-client';
import type { Path } from '@rocket.chat/rest-typings';
import type { CallbackHandler, Response } from 'supertest';
import supertest from 'supertest';
import { adminUsername, adminPassword } from './user';

// From apps/meteor/tests/data/users.helper.ts (lines 1-8)
import type { Credentials } from '@rocket.chat/api-client';
import type { IUser } from '@rocket.chat/core-typings';
import { UserStatus } from '@rocket.chat/core-typings';
import supertest from 'supertest';
import type { Response } from 'supertest';
import { api, credentials, methodCall, request } from './api-data';
import { password } from './user';
```

## 6. Configuration Examples

### DDP Client Initialization

```typescript
// From packages/ddp-client/README.md (lines 18-27)
// Create SDK instance
const sdk = DDPSDK.create('http://localhost:3000');

// Connect to server
await sdk.connection.connect();

// Or create and connect in one call
const sdk = DDPSDK.createAndConnect('http://localhost:3000');

// Check connection status
sdk.connection.status // 'connected'
```

### REST Client Initialization

```typescript
// From packages/api-client/src/index.ts (lines 53-70)
export class RestClient implements RestClientInterface {
	private readonly baseUrl: string;
	private headers: Record<string, string> = {};
	private credentials: Credentials | undefined;

	constructor({ baseUrl, credentials, headers = {} }: { baseUrl: string; credentials?: Credentials; headers?: Record<string, string> }) {
		this.baseUrl = `${baseUrl}/api`;
		this.setCredentials(credentials);
		this.headers = headers;
	}
}
```

### DDP SDK Modules

```typescript
// From packages/ddp-client/README.md (lines 29-49)
SDK Modules:
- Connection: Responsible for the connection to the server, status and connection states.
- Account: Responsible for account management, login, logout, handle credentials, get user information.
- ClientStream: Responsible for DDP communication, method calls, subscriptions.
- TimeoutControl: Responsible for Reconnection control.
- RestClient: Responsible for REST API communication.
```

### API Test Configuration

```typescript
// From apps/meteor/tests/data/api-data.ts (lines 8-11)
export const apiUrl = process.env.TEST_API_URL || 'http://localhost:3000';
export const request = supertest(apiUrl);
const prefix = '/api/v1/';

export function api<TPath extends PathWithoutPrefix<Path>>(path: TPath) {
	return `${prefix}${path}` as const;
}
```

### LiveChat Widget Configuration

```javascript
// From packages/livechat/README.md (lines 43-55)
const host = window.SERVER_URL
	|| queryString.parse(window.location.search).serverUrl
	|| (process.env.NODE_ENV === 'development' ? 'http://localhost:3000' : null);

// Change to custom server:
const host = window.SERVER_URL
	|| queryString.parse(window.location.search).serverUrl
	|| (process.env.NODE_ENV === 'development' ? 'https://your.rocketserver.com' : null);
```

## Sources

- `packages/ddp-client/README.md` - DDP Client SDK documentation
- `packages/api-client/src/index.ts` - REST API Client implementation
- `apps/meteor/tests/data/users.helper.ts` - User authentication and CRUD helpers
- `apps/meteor/tests/data/chat.helper.ts` - Chat operations (send, update, delete messages)
- `apps/meteor/tests/data/api-data.ts` - API test configuration and setup
- `packages/livechat/README.md` - LiveChat widget configuration
