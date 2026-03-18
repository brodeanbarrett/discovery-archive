# Part 5: Official SDKs & Client Libraries

## 5.1 Language Coverage

Rocket.Chat provides official SDKs and client libraries for the following programming languages:

| Language | SDK Name | Status | Registry |
|----------|----------|--------|----------|
| **JavaScript/TypeScript** | `@rocket.chat/api-client`, `@rocket.chat/ddp-client` | Active | npm |
| **Python** | `rocket-python` (archived), `rocketchat_API` (community) | Mixed | PyPI |
| **Java/Android** | `Rocket.Chat.Java.SDK` | Deprecated | Maven/JitPack |
| **Go** | `Rocket.Chat.Go.SDK` | Unmaintained | Go modules |
| **Kotlin** | `Rocket.Chat.Kotlin.SDK` | Community | GitHub |
| **Rust** | Community crates | Community | crates.io |
| **Mobile (React Native)** | `Rocket.Chat.ReactNative` | Active | GitHub |

## 5.2 Package Locations

### JavaScript/TypeScript SDKs (npm - @rocket.chat organization)

The Rocket.Chat organization on npm publishes 52 packages. Key SDK packages include:

| Package Name | Version | Description |
|--------------|---------|-------------|
| `@rocket.chat/api-client` | 0.2.52 | REST API client for browser and Node.js |
| `@rocket.chat/ddp-client` | 0.3.49 | DDP (Distributed Data Protocol) real-time client |
| `@rocket.chat/apps-engine` | 1.57.2 | Apps development framework |
| `@rocket.chat/apps-cli` | 1.13.0 | CLI tool for Rocket.Chat Apps |
| `@rocket.chat/apps-compiler` | 0.5.1 | Apps compiler |
| `@rocket.chat/desktop-api` | 1.1.0 | Desktop application API |
| `@rocket.chat/emitter` | 0.32.0 | Event emitter utility |
| `@rocket.chat/federation-sdk` | 0.4.2 | Matrix Federation SDK |
| `@rocket.chat/livechat` | - | Livechat widget SDK (Preact) |

Source: https://www.npmjs.com/org/rocket.chat

### Python SDKs (PyPI)

| Package Name | Version | Maintainer | Status |
|--------------|---------|------------|--------|
| `rocket-python` | - | Rocket.Chat (archived) | Deprecated - recommends community fork |
| `rocketchat_API` | 1.3.4 | Jorge Alberto Díaz Orozco | Active (community) |

Source: https://pypi.org/project/rocket-python/, https://pypi.org/project/rocketchat_API/

### Java/Android

| Package | Registry | Status |
|---------|----------|--------|
| `Rocket.Chat.Java.SDK` | GitHub/Maven/JitPack | Deprecated |
| `rocketchat-modern-client` | Maven Central (v1.3.2) | Deprecated |
| `rocket-chat-rest-client` | GitHub/JitPack | Community |

Source: https://github.com/RocketChat/Rocket.Chat.Java.SDK

### Livechat SDK (npm)

| Package Name | Version | Description |
|-------------|---------|-------------|
| `rocketchat-livechat-sdk` | 1.0.4 | Node.js wrapper for Omnichannel Livechat REST API |

Source: https://registry.npmjs.org/rocketchat-livechat-sdk

## 5.3 Installation Instructions

### JavaScript/TypeScript SDKs

```bash
# REST API Client
npm install @rocket.chat/api-client

# DDP (Real-time) Client
npm install @rocket.chat/ddp-client @rocket.chat/emitter

# Apps Engine
npm install @rocket.chat/apps-engine

# Livechat SDK
npm install rocketchat-livechat-sdk

# Livechat Widget
npm install @rocket.chat/livechat
```

Source: `packages/api-client/package.json:2-3`, `packages/ddp-client/package.json:2-3`

### Python SDK

```bash
# Official (deprecated)
pip install rocket-python

# Community-maintained (recommended)
pip install rocketchat_API
```

Source: https://pypi.org/project/rocket-python/, https://pypi.org/project/rocketchat_API/

### Java/Android

```xml
<!-- Maven via JitPack -->
<repository>
    <id>jitpack.io</id>
    <url>https://jitpack.io</url>
</repository>

<dependency>
    <groupId>com.github.RocketChat</groupId>
    <artifactId>Rocket.Chat.Android</artifactId>
    <version>latest.version</version>
</dependency>
```

Source: https://jitpack.io/p/RocketChat/Rocket.Chat.Android

## 5.4 Repository Links

| SDK | Repository URL |
|-----|----------------|
| Rocket.Chat (Monorepo) | https://github.com/RocketChat/Rocket.Chat |
| API Client | https://github.com/RocketChat/Rocket.Chat/tree/develop/packages/api-client |
| DDP Client | https://github.com/RocketChat/Rocket.Chat/tree/develop/packages/ddp-client |
| Apps Engine | https://github.com/RocketChat/Rocket.Chat/tree/develop/packages/apps-engine |
| React Native Mobile | https://github.com/RocketChat/Rocket.Chat.ReactNative |
| Python SDK (archived) | https://github.com/RocketChat/Rocket.Chat.py.SDK |
| Python SDK (community) | https://github.com/jadolg/rocketchat_API |
| Java SDK (deprecated) | https://github.com/RocketChat/Rocket.Chat.Java.SDK |
| Go SDK (unmaintained) | https://github.com/RocketChat/Rocket.Chat.Go.SDK |
| Kotlin SDK | https://github.com/RocketChat/Rocket.Chat.Kotlin.SDK |

## 5.5 Version Management

### JavaScript/TypeScript SDKs

The SDKs follow semantic versioning and are published from the Rocket.Chat monorepo:

- **Monorepo version**: 8.3.0-develop (as seen in `package.json:3`)
- **api-client**: 0.2.52 (published 2 days ago)
- **ddp-client**: 0.3.49 (published 2 days ago)
- **apps-engine**: 1.57.2 (published 2 days ago)

Source: `packages/api-client/package.json:3`, `packages/ddp-client/package.json:3`

### Python SDKs

- **rocket-python**: Archived - no longer actively versioned
- **rocketchat_API**: Active development, latest stable versions on PyPI

Source: https://pypi.org/project/rocketchat_API/

### Mobile SDK

- **React Native**: Version 4.70.1 (latest release March 9, 2026)

Source: https://github.com/RocketChat/Rocket.Chat.ReactNative/releases

## 5.6 Feature Matrix

| Feature | api-client | ddp-client | Python | Java | React Native |
|---------|------------|------------|--------|------|--------------|
| **REST API** | Full | Via api-client | Full | Partial | Via native |
| **Real-time (DDP)** | - | Full | Full | Partial | Via native |
| **User Management** | Yes | Yes | Yes | Yes | Yes |
| **Channel/Room Operations** | Yes | Yes | Yes | Yes | Yes |
| **Messaging** | Yes | Yes | Yes | Yes | Yes |
| **File Uploads** | Yes | Yes | Yes | - | Yes |
| **Livechat** | - | Via livechat | - | - | Yes |
| **User Auth** | Token/ Credentials | Token/ Credentials | Token/ Credentials | Token/ Credentials | Token |
| **TypeScript Types** | Yes | Yes | - | - | Yes |
| **React Native Mobile** | - | - | - | - | Full |

Source: `packages/api-client/src/index.ts:53-299`, `packages/ddp-client/src/index.ts`

### API Client Capabilities

The `@rocket.chat/api-client` provides:
- Generic HTTP methods: `get()`, `post()`, `put()`, `delete()`
- File upload with progress events
- Two-factor authentication (TOTP) handling
- Credentials management (`X-User-Id`, `X-Auth-Token` headers)
- Middleware support for request/response interception
- FormData support for file uploads

Source: `packages/api-client/src/index.ts:53-299`

### DDP Client Capabilities

The `@rocket.chat/ddp-client` provides:
- WebSocket-based real-time communication
- Method calls and subscriptions
- Client stream handling
- Livechat client implementation
- Legacy SDK compatibility

Source: `packages/ddp-client/src/index.ts`

## 5.7 Maintenance Status

| SDK | Status | Last Updated | Notes |
|-----|--------|--------------|-------|
| `@rocket.chat/api-client` | **Active** | 2 days ago | Part of monorepo, actively maintained |
| `@rocket.chat/ddp-client` | **Active** | 2 days ago | Part of monorepo, actively maintained |
| `@rocket.chat/apps-engine` | **Active** | 2 days ago | Actively maintained |
| `@rocket.chat/livechat` | **Active** | Recent | Preact-based widget SDK |
| `rocket-python` | **Deprecated** | Archived | Recommends `rocketchat_API` |
| `rocketchat_API` (Python) | **Active** | Recent | Community-maintained, actively developed |
| `Rocket.Chat.Java.SDK` | **Deprecated** | 4 months ago | Read-only, not maintained |
| `Rocket.Chat.Go.SDK` | **Unmaintained** | 8+ months ago | No longer actively developed |
| `Rocket.Chat.ReactNative` | **Active** | March 2026 | Mobile client actively maintained |

Source: npm package statistics, GitHub repositories

---

## Summary

Rocket.Chat provides official SDK coverage primarily for **JavaScript/TypeScript** through its monorepo packages. The REST API client (`@rocket.chat/api-client`) and DDP client (`@rocket.chat/ddp-client`) are actively maintained and published to npm. 

For other languages, Rocket.Chat's official support is limited:
- **Python**: Official SDK archived, community `rocketchat_API` recommended
- **Java/Android**: Deprecated, no official replacement
- **Go**: Unmaintained community SDK
- **Kotlin**: Community SDK available

The Rocket.Chat Apps Engine (`@rocket.chat/apps-engine`) provides a comprehensive framework for building extensions and integrations. Mobile development is supported through the React Native SDK (`Rocket.Chat.ReactNative`).

Developers seeking SDK support beyond JavaScript/TypeScript should consider the community-maintained libraries or direct REST API integration.
