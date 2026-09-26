# Rynet Portal Access & Distribution System (V6.0)

[![Architecture: Master-Worker](https://img.shields.io/badge/Architecture-Master--Worker%20Webhooks-blue.svg)](#architectural-overview)
[![Security: Defense-in-Depth](https://img.shields.io/badge/Security-7--Layer%20Zero--Trust-red.svg)](#security-architecture)
[![Runtime: Serverless GAS](https://img.shields.io/badge/Runtime-Google%20Apps%20Script%20%7C%20V8-green.svg)](#system-architecture)
[![License Model: Identity Bound DRM](https://img.shields.io/badge/DRM-Identity--Bound%20Time--Window-orange.svg)](#autonomous-lifecycle--drm-management)

An enterprise-grade, serverless **Digital Resource Distribution and Rights Management Platform** built on a distributed **Master-Worker Webhook Architecture**. The system operates as an autonomous, zero-cost, serverless infrastructure leveraging Google Apps Script (GAS), Google Drive API v2, GitHub Pages, and Blogspot to deliver secure, time-bound, and identity-bound digital asset access.

---

## Table of Contents

- [Executive Summary](#executive-summary)
- [Architectural Overview](#architectural-overview)
- [System Components](#system-components)
- [Communication Protocol (CMW)](#communication-protocol-cmw)
- [Security Architecture](#security-architecture)
- [Autonomous Lifecycle & DRM Management](#autonomous-lifecycle--drm-management)
- [Access Tiers & Licensing](#access-tiers--licensing)
- [Infrastructure & Performance Considerations](#infrastructure--performance-considerations)

---

## Executive Summary

The **Rynet Portal Distribution System** orchestrates end-to-end access lifecycle management for large-scale digital assets. Rather than distributing physical files through risky direct download links or heavy server bandwidth, the platform acts as an **Identity-Bound Digital Rights Management (DRM)** and **Miniature Content Delivery Network (CDN)** router.

### Key Capabilities:
- **Zero-Trust Security Pipeline:** 7-stage access validation pipeline enforcing strict IP, device fingerprint, single-use token, and HMAC-SHA256 signature verification.
- **Autonomous Lifecycle Management:** Self-executing cron engines handle automated permission provisioning, expiration enforcement, and instant access revoking without manual administrative oversight.
- **Distributed Edge Storage (Sharding):** Horizontal expansion via isolated Worker Storage Nodes, keeping storage quotas segregated and master operations lightweight.
- **Atomic Operations & Idempotency:** Guaranteed data integrity across distributed nodes with auto-rollback mechanisms upon storage provisioning errors.

---

## Architectural Overview

The system decouples **central orchestrator logic** from **storage execution**, operating under a **Master-Worker topology**.

```
┌────────────────────────────────────────────────────────────────────────┐
│                    RYNET PORTAL DISTRIBUTION SYSTEM                    │
│                                                                        │
│  [USER / CLIENT]                                                       │
│      │                                                                 │
│      ▼                                                                 │
│  ┌─────────────────┐    Generate Token  ┌───────────────────────────┐  │
│  │ BLOG ENTRY GATE │ ─────────────────► │ Master Central Node (GAS) │  │
│  │ (WCaptcha JS)   │                    │ - Identity & Auth Engine  │  │
│  └─────────────────┘                    │ - State & Policy Registry │  │
│                                         └─────────────┬─────────────┘  │
│  ┌─────────────────┐  Claim Request                   │                │
│  │ PORTAL CLIENT   │ ─────────────────────────────────┤                │
│  │ (GitHub Pages)  │                                  │ Dispatch (CMW) │
│  └─────────────────┘                                  ▼                │
│                                         ┌───────────────────────────┐  │
│  ┌─────────────────┐  Access Query      │ Worker Storage Node (GAS) │  │
│  │ CHECK DASHBOARD │ ─────────────────► │ - Isolated Storage Worker │  │
│  │ (Status Portal) │                    │ - Local Ledger & Execution│  │
│  └─────────────────┘                    └─────────────┬─────────────┘  │
│                                                       │                │
│                                         Grant / Revoke│ Drive API v2   │
│                                                       ▼                │
│                                         ┌───────────────────────────┐  │
│                                         │   Google Drive Storage    │  │
│                                         │   (Encrypted Assets)      │  │
│                                         └───────────────────────────┘  │
└────────────────────────────────────────────────────────────────────────┘
```

### CDN & DRM Architectural Analogy

1. **Origin Server & Router (Master Node):** The central controller handles authorization, licensing, abuse prevention, and node routing without processing file payload transfers.
2. **Edge Storage Nodes (Worker Nodes):** Decentralized storage workers manage actual asset permissions directly at the cloud storage layer.
3. **Identity-Bound Access (Modern DRM):** Access permissions are dynamically attached to the user's validated Google identity rather than releasing static credentials or reusable static links.

---

## System Components

### 1. Token Issuance Gateway (Blogspot Entry)
- **Role:** Primary anti-bot front door and session token generator.
- **Functionality:** Collects anonymized telemetry (client IP, device fingerprint, tab isolation ID) to issue cryptographically signed, single-use **WCaptcha Tokens** with a 1-hour expiration window.

### 2. Main Access Portal (GitHub Pages SPA)
- **Role:** Client-side interface for asset claiming (Free Tier, Premium Tier, and Gift Vouchers).
- **Functionality:** Validates dynamic tokens, monitors submission metrics, enforces human interaction delays (>2500ms), and provides real-time polling on asynchronous processing status.

### 3. User Status & History Dashboard (GitHub Pages SPA)
- **Role:** Self-service diagnostic interface for end-users.
- **Functionality:** Displays active ticket windows, remaining 24-hour quota, active storage access slots, and transaction history under strict query rate limits.

### 4. Master Orchestrator Node (Google Apps Script Backend)
- **Role:** Single source of truth and system authority.
- **Functionality:** Evaluates state policy, manages rate-limiting registers, executes remote configuration state changes, routes requests to target storage nodes, and maintains system audit logs.

### 5. Worker Storage Nodes (Google Apps Script Workers)
- **Role:** Isolated execution agents.
- **Functionality:** Executes dynamic permission provisioning (`GRANT_ACCESS`) and revoking (`REVOKE_ACCESS`) on storage repositories. Runs independent cron engines for local state sync and batch reporting.

---

## Communication Protocol (CMW)

The **Communication Master-Worker (CMW) Protocol** is a resilient, 2-stage asynchronous communication mechanism over HTTP POST webhooks.

```
 Client Request
       │
       ▼
 [STAGE 1: Handshake & Health Probe]
       ├── Master sends PING_STATUS to target Worker Node
       ├── Retries up to 3 times with 30s timeout windows
       └── Validates worker readiness and active operational state
       │
       ▼
 [STAGE 2: Signed Request Execution]
       ├── Master initializes pending transaction record
       ├── Computes dynamic HMAC-SHA256 signature payload
       ├── Transmits execution instruction (GRANT/REVOKE) to Worker
       ├── Worker validates signature & anti-replay timestamp window (60s)
       ├── Worker executes storage permission update via Drive API
       └── Worker returns atomic status response to Master
```

### Protocol Protection Features:
- **Anti-Replay Verification:** Requests with timestamp skew exceeding 60 seconds are rejected immediately.
- **Dynamic HMAC Signature:** Payloads are signed with SHA-256 using shared node secrets.
- **Idempotency Locks:** Prevents duplicate submissions by locking concurrent pending requests for the same identity window.
- **Circuit Breaker:** Stale pending requests older than 5 minutes automatically transition to a failed state to preserve system consistency.

---

## Security Architecture

The platform enforces a **7-Layer Defense-in-Depth Pipeline**. Every request must pass all validation checkpoints sequentially before access is granted.

```
  Incoming Request
         │
         ▼
 ┌─────────────────────────────────────────────────────────────┐
 │ [LAYER 1] Remote Master Control Switch                      │
 │ Checks global operational mode (ACTIVE/MAINTENANCE/DISABLE) │
 └──────────────────────────────┬──────────────────────────────┘
                                │ Pass
                                ▼
 ┌─────────────────────────────────────────────────────────────┐
 │ [LAYER 2] IP Rate Defender                                  │
 │ Enforces daily token generation ceilings per client IP      │
 └──────────────────────────────┬──────────────────────────────┘
                                │ Pass
                                ▼
 ┌─────────────────────────────────────────────────────────────┐
 │ [LAYER 3] Multi-Account & Sybil Escalation Detector         │
 │ Detects multiple identities sharing device/IP fingerprints  │
 └──────────────────────────────┬──────────────────────────────┘
                                │ Pass
                                ▼
 ┌─────────────────────────────────────────────────────────────┐
 │ [LAYER 4] Token Cooldown Enforcer                           │
 │ Enforces mandatory rest windows between token requests      │
 └──────────────────────────────┬──────────────────────────────┘
                                │ Pass
                                ▼
 ┌─────────────────────────────────────────────────────────────┐
 │ [LAYER 5] Threat Intelligence & Blacklist Filter            │
 │ Screens identity against temporary and permanent ban lists  │
 └──────────────────────────────┬──────────────────────────────┘
                                │ Pass
                                ▼
 ┌─────────────────────────────────────────────────────────────┐
 │ [LAYER 6] WCaptcha Single-Use Token Validator               │
 │ Verifies cryptographic token integrity, TTL, and burn state │
 └──────────────────────────────┬──────────────────────────────┘
                                │ Pass
                                ▼
 ┌─────────────────────────────────────────────────────────────┐
 │ [LAYER 7] Dynamic HMAC-SHA256 Worker Handshake              │
 │ Authenticates Master-to-Worker POST payload integrity       │
 └──────────────────────────────┬──────────────────────────────┘
                                │ Pass
                                ▼
                   Access Granted & Provisioned
```

> [!IMPORTANT]
> **Automated Sanction Escalation:**
> - **Token Abuse:** Exceeding issuance thresholds triggers automated 24-hour IP isolation.
> - **Sybil Attack Protection:** Multi-account abuse associated with matching device/network fingerprints results in automated access revocation and permanent blacklisting across all linked identities.

---

## Autonomous Lifecycle & DRM Management

The platform operates on a **Stateful Identity Lifecycle**, replacing static file distributions with controlled access windows.

```
 ┌─────────┐      Claim      ┌────────┐     24h / Duration     ┌─────────┐
 │ PENDING │ ──────────────► │ ACTIVE │ ─────────────────────► │ EXPIRED │
 └─────────┘                 └────────┘                        └────┬────┘
                                  │                                 │
                                  │ Admin / Violation               │ Cron Sweep
                                  ▼                                 ▼
                             ┌──────────┐                      ┌─────────┐
                             │ BANNED / │ ◄─────────────────── │ REVOKED │
                             │ BLOCKED  │                      └─────────┘
                             └──────────┘
```

### Self-Healing & Cron Synchronization
1. **Worker-Side Cron Engine:** Runs periodically to inspect local permission states, execute direct storage permission revocations, and update local ledger states.
2. **Master-Side Cron Engine:** Periodically sweeps central state ledgers, synchronizes license expirations, clears stale token caches, and processes orphaned transactions.
3. **Atomic Rollback:** If storage permission assignment succeeds but local logging encounters an exception, the system triggers an emergency rollback, revoking the permission instantly to prevent untracked access leaks.

---

## Access Tiers & Licensing

The architecture supports a multi-tier entitlement structure to balance resource accessibility and platform fair use:

| Metric / Feature | Free Tier | Premium Tier | Gift Voucher Tier |
|---|---|---|---|
| **Identity Requirement** | Validated Google Account | Valid Premium Key + Linked Account | Valid Gift Code + Ex-Premium Account |
| **Active Access Window** | 24 Hours | Ticket Duration (e.g., 15 Days) | Designated Custom Window (e.g., 24 Hours) |
| **Cooldown Period** | 3 Days post-expiration | None | None |
| **Concurrency Ceiling** | Global active slot cap | Dedicated per-user asset limit | Dedicated per-user asset limit |
| **Daily Claim Quota** | 1 Asset per cycle | Up to 3 Assets per 24 Hours | Bound to designated asset |
| **Upgrade Handling** | Auto-converts Free to Expired | N/A | N/A |

---

## Infrastructure & Performance Considerations

### Why Single-Master with Decentralized Workers?
Evaluations of multi-master configurations demonstrated significant overhead due to inter-script HTTP latencies ($\approx 500\text{ms} - 1.5\text{s}$ per hop) and lock contention on shared database resources.

By maintaining a **single central orchestrator** alongside **decentralized worker storage nodes**:
- **Constant Time Routing $\mathcal{O}(1)$:** Master routes claims based on lightweight catalog mapping without carrying storage processing burdens.
- **Quota Segregation:** Storage API call limits are distributed independently across worker Google accounts, ensuring high availability and system durability.
- **Zero Operating Costs:** Fully serverless deployment requiring zero dedicated physical server footprint or recurring infrastructure overhead.

---

> *Documentation Version: 6.0 | System Architecture Standard*
