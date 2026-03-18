# Part 6: Third-Party & Community SDKs

## Overview

This section documents the SDK ecosystem surrounding Rocket.Chat, including official SDKs maintained by Rocket.Chat and notes about community-driven implementations. The main Rocket.Chat repository (this monorepo) contains the server implementation and official client libraries, while community SDKs exist in separate repositories.

---

## 6.1 Community SDKs

### Official SDK Packages (Within Monorepo)

Rocket.Chat maintains several official SDK packages as part of this monorepo:

#### @rocket.chat/api-client
- **Purpose**: REST API client for communicating with Rocket.Chat servers
- **Location**: `packages/api-client/package.json`
- **Version**: 0.2.51
- **Key Dependencies**:
  - `@rocket.chat/core-typings`
  - `@rocket.chat/rest-typings`
  - `query-string`
  - `split-on-first`

**Source**: `packages/api-client/package.json` (lines 1-38)

#### @rocket.chat/ddp-client
- **Purpose**: DDP (Distributed Data Protocol) client for real-time communication
- **Location**: `packages/ddp-client/package.json`
- **Version**: 1.0.4
- **Key Dependencies**:
  - `@rocket.chat/api-client`
  - `@rocket.chat/core-typings`
  - `@rocket.chat/media-signaling`
  - `@rocket.chat/rest-typings`

**Source**: `packages/ddp-client/package.json` (lines 1-42)

#### @rocket.chat/rest-typings
- **Purpose**: TypeScript type definitions for the REST API
- **Location**: `packages/rest-typings/package.json`
- **Version**: 8.3.0-develop

**Source**: `packages/rest-typings/package.json` (lines 1-34)

#### @rocket.chat/federation-sdk
- **Purpose**: Official SDK for Rocket.Chat Federation features
- **Version**: 0.4.1
- **Usage**: Used by `ee/packages/federation-matrix` for Matrix protocol federation

**Source**: 
- `apps/meteor/package.json` (line 106)
- `ee/packages/federation-matrix/package.json` (line 25)
- `ee/packages/federation-matrix/src/FederationMatrix.ts` (line 11)

### Internal SDK Client

Rocket.Chat uses an internal SDK client (`SDKClient`) that combines both REST and DDP functionality:

- **Location**: `apps/meteor/app/utils/client/lib/SDKClient.ts`
- **Components**:
  - `APIClient` - REST API client
  - DDP client for real-time subscriptions and method calls
- **Usage**: Used throughout the client codebase for server communication

**Source**: `apps/meteor/app/utils/client/lib/SDKClient.ts` (lines 1-272)

---

## 6.2 Wrapper Libraries

### Official Wrapper: SDKClient

The primary wrapper around Rocket.Chat APIs is the internal `SDKClient`:

```typescript
// Example usage pattern found throughout codebase
import { sdk } from '../../../utils/client/lib/SDKClient';
```

**Source**: Multiple files including:
- `apps/meteor/app/autotranslate/client/lib/autotranslate.ts` (line 16)
- `apps/meteor/app/slashcommand-asciiarts/client/gimme.ts` (line 3)
- `apps/meteor/app/ui/client/lib/UserAction.ts` (line 8)

### REST API Client Wrapper

The `APIClient` provides a wrapper around the REST API:

**Source**: `apps/meteor/app/utils/client/lib/SDKClient.ts` (line 8)
```typescript
import { APIClient } from './RestApiClient';
```

---

## 6.3 Unofficial Implementations

### No Community SDKs in Monorepo

This repository (Rocket.Chat server monorepo) does **not** contain third-party or community SDK implementations. Community SDKs for other programming languages exist in separate repositories:

- **Python SDKs**: Community-developed Python libraries exist on PyPI
- **Go SDKs**: Community Go packages available
- **Other Languages**: Various community implementations exist externally

**Note**: These external community SDKs are not part of this repository and should be researched separately from the official Rocket.Chat GitHub organization.

---

## 6.4 SDK Quality Assessment

### Official SDKs

| SDK | Maturity | Maintenance Status | Notes |
|-----|----------|-------------------|-------|
| @rocket.chat/api-client | High | Active | Part of main monorepo, well-maintained |
| @rocket.chat/ddp-client | High | Active | Core component, actively developed |
| @rocket.chat/rest-typings | High | Active | TypeScript definitions maintained with API |
| @rocket.chat/federation-sdk | Medium | Active | Version 0.4.1, specific to federation |

### Maintenance Indicators

- **Active Development**: All packages are in the current monorepo with active commits
- **Version Alignment**: SDK versions align with server version 8.3.0-develop
- **Type Safety**: TypeScript typings provided for all official APIs

**Source**: `package.json` (line 3) - Version 8.3.0-develop

---

## 6.5 License Considerations

### Third-Party Licenses

Rocket.Chat maintains a comprehensive third-party licenses document:

- **Location**: `apps/meteor/licenses/THIRD-PARTY-LICENSES.md`
- **Contents**: Lists all third-party components incorporated into Rocket.Chat

**Source**: `apps/meteor/licenses/THIRD-PARTY-LICENSES.md` (lines 1-37)

### Main License

- **Project License**: MIT License
- **Badge**: Shown in README.md

**Source**: `README.md` (line 12)
```markdown
<img src="https://img.shields.io/github/v/release/RocketChat/Rocket.Chat?label=version">
<img alt="Codecov branch" src="https://img.shields.io/codecov/c/github/RocketChat/Rocket.Chat/develop">
<img alt="" src="https://img.shields.io/badge/license-MIT-green">
```

### Official SDK Licenses

All official Rocket.Chat packages use the **MIT License**, as indicated by:
- Package badges in README files showing MIT license
- Repository license file

**Source**: Multiple package README files (e.g., `packages/ddp-client/README.md`, `packages/ui-kit/README.md`)

---

## 6.6 Related Ecosystem: Marketplace

While not SDKs, Rocket.Chat's marketplace extends functionality:

- **Marketplace**: Located at `apps/meteor/client/views/marketplace/`
- **Apps Engine**: Custom apps can be built using the Apps-Engine framework
- **Documentation**: Referenced in `packages/apps-engine/README.md`

**Source**: 
- `.github/history.json` - Multiple marketplace-related commits
- `packages/apps-engine/README.md` (line 58)

---

## Summary

| Checklist Item | Status | Notes |
|---------------|--------|-------|
| Community SDKs | Documented | Official SDKs in monorepo; external community SDKs exist separately |
| Wrapper Libraries | Documented | SDKClient provides wrapper functionality |
| Unofficial Implementations | Noted | Not present in this repository |
| SDK Quality Assessment | Complete | High maturity for official SDKs |
| License Considerations | Complete | MIT license; third-party licenses documented |

---

## Sources

1. `packages/api-client/package.json` - API Client package definition
2. `packages/ddp-client/package.json` - DDP Client package definition
3. `packages/rest-typings/package.json` - REST typings package definition
4. `apps/meteor/package.json` (line 106) - Federation SDK dependency
5. `apps/meteor/app/utils/client/lib/SDKClient.ts` - Internal SDK implementation
6. `packages/ddp-client/README.md` - Package documentation
7. `apps/meteor/licenses/THIRD-PARTY-LICENSES.md` - Third-party licenses
8. `README.md` - Main README with license information
9. `ee/packages/federation-matrix/src/FederationMatrix.ts` (line 11) - Federation SDK usage
