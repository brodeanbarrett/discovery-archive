# Part 3: Authentication & Authorization

## Overview

Rocket.Chat provides a comprehensive multi-layered authentication and authorization system supporting various authentication methods including password-based login, OAuth 2.0, SAML, LDAP, CAS, and personal access tokens. The authorization system is built on a role-based access control (RBAC) model with granular permissions.

---

## 3.1 Authentication Methods

Rocket.Chat supports the following authentication types:

### 3.1.1 Password-Based Authentication

- **Method**: Username/email + password login via `loginWithPassword`
- **Implementation**: `apps/meteor/client/meteor/login/password.ts`
- **Configuration**:
  - `Accounts_LoginExpiration` - Token expiration in days (default: 90)
  - `Accounts_Password_History_Enabled` - Prevents password reuse
  - `Accounts_AllowedDomainsList` - Domain whitelist for email

**Source**: `apps/meteor/app/authentication/server/startup/index.js` (lines 29-51)

### 3.1.2 OAuth 2.0

Rocket.Chat supports OAuth 2.0 integration with multiple providers:

| Provider | Setting | Callback URL |
|----------|---------|--------------|
| Google | `Accounts_OAuth_Google` | `_oauth/google` |
| GitHub | `Accounts_OAuth_Github` | `_oauth/github` |
| GitHub Enterprise | `Accounts_OAuth_GitHub_Enterprise` | `_oauth/github_enterprise` |
| GitLab | `Accounts_OAuth_Gitlab` | `_oauth/gitlab` |
| Facebook | `Accounts_OAuth_Facebook` | `_oauth/facebook` |
| Twitter | `Accounts_OAuth_Twitter` | `_oauth/twitter` |
| LinkedIn | `Accounts_OAuth_Linkedin` | `_oauth/linkedin` |
| Apple | `Accounts_OAuth_Apple` | `_oauth/apple` |
| Nextcloud | `Accounts_OAuth_Nextcloud` | `_oauth/nextcloud` |
| WordPress | `Accounts_OAuth_Wordpress` | `_oauth/wordpress` |
| Drupal | `Accounts_OAuth_Drupal` | `_oauth/drupal` |
| Meteor | `Accounts_OAuth_Meteor` | `_oauth/meteor` |
| Dolphin | `Accounts_OAuth_Dolphin` | - |
| Custom OAuth | `Accounts_OAuth_Custom-{name}` | `_oauth/custom-{name}` |

**Source**: `apps/meteor/server/settings/oauth.ts` (lines 1-419)

### 3.1.3 SAML 2.0

- **Configuration**: `SAML_Custom_` settings
- **Features**: Role sync, customizable default roles, signing algorithms
- **Implementation**: `apps/meteor/app/meteor-accounts-saml`

**Source**: Multiple files in `apps/meteor/app/meteor-accounts-saml/`

### 3.1.4 LDAP (Lightweight Directory Access Protocol)

- **Purpose**: Enterprise directory authentication and user sync
- **Settings**: `LDAP_Authentication`, `LDAP_Enable`, `LDAP_Sync_*`
- **Implementation**: `apps/meteor/server/lib/ldap/Manager.ts`

**Source**: `apps/meteor/server/lib/ldap/Connection.ts` (line 87)

### 3.1.5 CAS (Central Authentication Service)

- **Implementation**: `apps/meteor/server/lib/cas/loginHandler.ts`
- **Setting**: `CAS_enabled`, `CAS_url`, `CAS_service_url`

### 3.1.6 Personal Access Tokens

- **Endpoint**: `POST /api/v1/users.generatePersonalAccessToken`
- **Permission Required**: `create-personal-access-tokens`
- **Token Format**: Random 43-character secret (via `Random.secret()`)
- **Storage**: Hashed using `Accounts._hashLoginToken()`

**Source**: `apps/meteor/imports/personal-access-tokens/server/api/methods/generateToken.ts` (lines 17-55)

### 3.1.7 Two-Factor Authentication (TOTP)

- **Settings**:
  - `Accounts_TwoFactorAuthentication_Enabled` (default: true)
  - `Accounts_TwoFactorAuthentication_By_TOTP_Enabled`
  - `Accounts_TwoFactorAuthentication_By_Email_Enabled`
  - `Accounts_TwoFactorAuthentication_RememberFor` (default: 1800 seconds)

**Source**: `apps/meteor/server/settings/accounts.ts` (lines 9-91)

### 3.1.8 Iframe Authentication

- **Settings**: `Accounts_iframe_enabled`, `Accounts_iframe_url`, `Accounts_Iframe_api_url`
- **Purpose**: Embedded authentication for third-party integrations

**Source**: `apps/meteor/server/settings/accounts.ts` (lines 172-174)

---

## 3.2 OAuth Flows

### 3.2.1 Authorization Code Flow

Rocket.Chat implements the standard OAuth 2.0 Authorization Code flow:

1. **Authorization Endpoint**: `GET /oauth/authorize`
   - Validates `client_id`, `redirect_uri`
   - Displays consent screen to user

2. **Token Endpoint**: `POST /oauth/token`
   - Exchanges authorization code for access token
   - Supports `authorization_code` and `refresh_token` grant types

**Source**: `apps/meteor/server/oauth2-server/oauth.ts` (lines 82-93, 95-171)

### 3.2.2 OAuth User Info Endpoint

- **Endpoint**: `GET /oauth/userinfo`
- **Authentication**: Bearer token required
- **Response Fields**: `sub`, `name`, `email`, `email_verified`, `preferred_username`, `updated_at`, `picture`

**Source**: `apps/meteor/app/oauth2-server-config/server/oauth/oauth2-server.ts` (lines 48-72)

### 3.2.3 OAuth2 Server Library

- **Package**: `@node-oauth/oauth2-server`
- **Configuration**: `apps/meteor/server/oauth2-server/oauth.ts`

---

## 3.3 Token Endpoints

### 3.3.1 OAuth Token Endpoint

| Property | Value |
|----------|-------|
| Path | `/oauth/token` |
| Method | POST |
| Content-Type | `application/x-www-form-urlencoded` |
| Response | JSON with `access_token`, `token_type`, `expires_in`, `refresh_token` |

**Source**: `apps/meteor/server/oauth2-server/oauth.ts` (lines 82-93)

### 3.3.2 Personal Access Token Generation

| Property | Value |
|----------|-------|
| Path | `/api/v1/users.generatePersonalAccessToken` |
| Method | POST |
| Required Params | `tokenName`, `bypassTwoFactor` |
| Auth Required | Yes |

**Source**: `apps/meteor/app/api/server/v1/users.ts` (lines 954-964)

### 3.3.3 Token User Info

| Property | Value |
|----------|-------|
| Path | `/api/v1/me` |
| Method | GET |
| Auth Required | Yes |
| Returns | Authenticated user data |

**Source**: `apps/meteor/app/api/server/v1/misc.ts` (lines 41-46)

---

## 3.4 Token Refresh & Expiration

### 3.4.1 Login Token Expiration

- **Setting**: `Accounts_LoginExpiration` (default: 90 days)
- **Implementation**: Uses `getLoginExpirationInDays()` helper

**Source**: `apps/meteor/app/authentication/server/startup/index.js` (lines 44-51)

### 3.4.2 JWT Token Expiration

- **Default Expiration**: 1 hour (`exp: now + 1hour`)
- **Algorithm**: HS256
- **Audience**: `RocketChat` (configurable)

**Source**: `apps/meteor/app/utils/server/lib/JWTHelper.ts` (lines 1-56)

### 3.4.3 OAuth Access Token Expiration

- Tokens checked against `expires` field
- Expired tokens return `undefined` user, triggering re-authentication

**Source**: `apps/meteor/app/oauth2-server-config/server/oauth/oauth2-server.ts` (lines 30-35)

### 3.4.4 Two-Factor Authentication Code Expiration

- **Setting**: `Accounts_TwoFactorAuthentication_By_Email_Code_Expiration` (default: 3600 seconds)

**Source**: `apps/meteor/server/settings/accounts.ts` (line 61)

---

## 3.5 Scopes & Permissions

Rocket.Chat implements a comprehensive permission system with over 150 individual permissions.

### 3.5.1 Core Permission Categories

| Category | Description | Example Permissions |
|----------|-------------|---------------------|
| Administration | Server management | `access-permissions`, `view-statistics`, `view-logs` |
| User Management | User CRUD operations | `create-user`, `delete-user`, `edit-other-user-info` |
| Room Management | Channel/group operations | `create-c`, `create-p`, `delete-c`, `archive-room` |
| Message Management | Messaging permissions | `edit-message`, `delete-message`, `pin-message` |
| Integration Management | Webhooks, bots, apps | `manage-incoming-integrations`, `manage-outgoing-integrations` |
| OAuth | OAuth management | `add-oauth-service`, `manage-oauth-apps` |
| Livechat | Omnichannel operations | `view-l-room`, `close-livechat-room`, `transfer-livechat-guest` |
| Team Management | Team operations | `create-team`, `delete-team`, `add-team-member` |

**Source**: `apps/meteor/app/authorization/server/constant/permissions.ts` (lines 1-242)

### 3.5.2 Default Roles

| Role | Description |
|------|-------------|
| `admin` | Full administrative access |
| `moderator` | Room moderation capabilities |
| `owner` | Room ownership |
| `user` | Standard user |
| `guest` | Limited access, cannot post |
| `bot` | Automated service account |
| `app` | App service account |
| `anonymous` | Unauthenticated user |
| `federated-external` | Federation participant |

### 3.5.3 Livechat-Specific Roles

| Role | Description |
|------|-------------|
| `livechat-manager` | Omnichannel management |
| `livechat-agent` | Customer support agent |
| `livechat-monitor` | Livechat monitoring |

**Source**: `apps/meteor/app/authorization/server/constant/permissions.ts` (lines 93-183)

---

## 3.6 Role-Based Access Control (RBAC)

### 3.6.1 Permission Assignment Model

Each permission defines which roles can perform the action:

```typescript
{ _id: 'permission-name', roles: ['role1', 'role2'] }
```

**Source**: `apps/meteor/app/authorization/server/constant/permissions.ts` (line 5)

### 3.6.2 Key RBAC Patterns

#### Room-Level Roles
- **Owner**: Full control over room
- **Moderator**: Message management
- **Leader**: Priority in calls

**Source**: `apps/meteor/app/authorization/server/constant/permissions.ts` (lines 66-69)

#### Global Roles
- **Admin**: Server-wide permissions
- **User**: Standard access

#### Service Roles
- **Bot**: Automated actions (`send-many-messages`, `bypass-time-limit-edit-and-delete`)
- **App**: App integration actions

### 3.6.3 Authorization Functions

| Function | Purpose |
|----------|---------|
| `hasPermission()` | Check if user has specific permission |
| `hasRole()` | Check if user has specific role |
| `canAccessRoom()` | Validate room access |
| `canSendMessage()` | Validate message sending |
| `canDeleteMessage()` | Validate message deletion |

**Source**: `apps/meteor/app/authorization/server/functions/` (multiple files)

### 3.6.4 Permission Middleware

REST API endpoints use permission-based middleware:

```typescript
{ authRequired: true, permissionsRequired: ['permission-name'] }
```

**Source**: `apps/meteor/app/api/server/middlewares/authentication.ts` (lines 47-72)

---

## 3.7 API Key Management

### 3.7.1 Personal Access Tokens (PAT)

| Operation | Endpoint | Permission |
|-----------|----------|------------|
| Generate | `POST /api/v1/users.generatePersonalAccessToken` | `create-personal-access-tokens` |
| Regenerate | `POST /api/v1/users.regeneratePersonalAccessToken` | `create-personal-access-tokens` |
| Remove | `POST /api/v1/users.removePersonalAccessToken` | `create-personal-access-tokens` |
| List | `GET /api/v1/users.getPersonalAccessTokens` | `create-personal-access-tokens` |

**Source**: `apps/meteor/app/api/server/v1/users.ts` (lines 954-1000)

### 3.7.2 PAT Token Structure

- **Name**: User-defined token identifier
- **Value**: 43-character random secret
- **Hashing**: SHA-256 via `Accounts._hashLoginToken()`
- **Storage**: `users.services.resume.loginTokens`
- **Type**: `personalAccessToken`

**Source**: `apps/meteor/imports/personal-access-tokens/server/api/methods/generateToken.ts` (lines 32-53)

### 3.7.3 API Key Authentication in Requests

Rocket.Chat supports multiple authentication methods in API requests:

1. **Header Authentication**:
   - `x-user-id`: User identifier
   - `x-auth-token`: Authentication token

2. **Bearer Token**:
   - `Authorization: Bearer <access_token>`

3. **Query Parameter**:
   - `?access_token=<token>`

**Source**: `apps/meteor/app/api/server/middlewares/authentication.ts` (lines 20-34)

### 3.7.4 Third-Party API Keys

External service API keys (not for authentication):

| Service | Setting | Purpose |
|---------|---------|---------|
| Google Translate | `AutoTranslate_GoogleAPIKey` | Translation |
| Microsoft Translate | `AutoTranslate_Microsoft_API_Key` | Translation |
| DeepL | `AutoTranslate_DeepL_API_Key` | Translation |
| Push Notifications | `GCM_ApiKey` | Mobile notifications |

**Source**: `apps/meteor/app/autotranslate/server/googleTranslate.ts` (line 25)

### 3.7.5 Bypass Rate Limiting

- **Permission**: `api-bypass-rate-limit`
- **Roles**: `admin`, `bot`, `app`
- **Purpose**: Allow high-volume API access for services

**Source**: `apps/meteor/app/authorization/server/constant/permissions.ts` (line 15)

---

## Summary

Rocket.Chat provides a robust, multi-layered authentication and authorization system:

- **7+ authentication methods** including password, OAuth 2.0, SAML, LDAP, CAS, PAT, and TOTP
- **OAuth 2.0 compliant** token endpoints with Authorization Code flow
- **150+ granular permissions** organized by administrative, room, message, and integration categories
- **Comprehensive RBAC** with room-level, global, and service-specific roles
- **Personal Access Tokens** for API authentication with role-based generation permissions
- **Two-factor authentication** support via TOTP and email
- **Flexible token management** with expiration, refresh, and revocation capabilities
