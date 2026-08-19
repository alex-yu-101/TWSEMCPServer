# Improvement Roadmap

This document tracks proposed improvements that are not yet part of the
upstream project. It is a lightweight working backlog for planning changes
before they become implementation branches, pull requests, or upstream issues.

Nothing listed here should be treated as an accepted upstream commitment.
Large architectural changes should be discussed with the upstream maintainer
before significant implementation work begins.

## Status and priority

Statuses:

- **Proposed** — recorded for evaluation; scope is not final.
- **Approved** — direction and scope have been agreed upon.
- **In progress** — implementation has started on a dedicated branch.
- **Blocked** — waiting for a decision, dependency, or external input.
- **Done** — implemented and verified.
- **Deferred** — intentionally postponed.

Priorities:

- **High** — required for a secure, usable public deployment.
- **Medium** — important operational capability after the core is working.
- **Low** — useful enhancement that is not currently essential.

## Proposed architecture and administration work

| Improvement | Priority | Status | Dependencies | Branch / PR |
| --- | --- | --- | --- | --- |
| Define the deployment and admin architecture | High | Proposed | None | — |
| Design API-key storage and lifecycle | High | Proposed | Architecture decision | — |
| Add API-key authentication middleware for HTTP transport | High | Proposed | API-key storage | — |
| Add separate administrator authentication | High | Proposed | Architecture decision | — |
| Add API-key management endpoints | High | Proposed | API-key storage; administrator authentication | — |
| Build an administrator dashboard | High | Proposed | Administrator authentication; management endpoints | — |
| Add request and audit logging | Medium | Proposed | Architecture decision; persistence | — |
| Add per-key rate limiting | Medium | Proposed | Authentication middleware; API-key storage | — |
| Add security, integration, and end-to-end tests | High | Proposed | Implemented authentication components | — |
| Add public-deployment and administration documentation | Medium | Proposed | Finalized deployment design | — |

## Scope notes

### Deployment and admin architecture

Decide and document:

- the supported public deployment topology;
- whether the admin application runs in the same process as the MCP server;
- the persistence technology and migration strategy;
- the boundary between public MCP endpoints and admin endpoints;
- how local `stdio` operation differs from protected HTTP operation; and
- the configuration and secret-management model.

### API-key storage and lifecycle

Define the key format and support secure creation, hashing, lookup, naming,
expiration, rotation, and revocation. Raw API keys should only be displayed at
creation time and should not be stored in plaintext.

### HTTP authentication middleware

Protect public HTTP MCP requests at a central gateway before they reach MCP
tools. The middleware should reject missing, invalid, expired, or revoked keys
consistently. Local `stdio` transport is outside this middleware path and needs
a separately documented trust model.

### Administrator authentication

Keep administrator login separate from client API-key authentication. Define
secure session handling, credential storage, logout, and protection against
common web attacks before building the dashboard.

### API-key management endpoints

Provide administrator-only operations to list key metadata, create keys,
revoke keys, rotate keys, and inspect relevant usage information without
exposing stored secrets.

### Administrator dashboard

Build a minimal interface on top of the management endpoints. Initial scope
should focus on authentication and API-key lifecycle operations rather than a
general analytics platform.

### Logging and rate limiting

Record security-relevant events without logging credentials or raw API keys.
Rate limits should be configurable and enforced per key, with predictable error
responses and enough metadata for administrators to investigate problems.

## Working process

1. Refine a proposed item until its goal and acceptance criteria are clear.
2. Discuss substantial architecture or behavior changes with upstream.
3. Mark the item **Approved** before implementation.
4. Create a dedicated feature or fix branch; do not develop directly on
   `main`.
5. Add the branch or pull-request link to the table and mark the item
   **In progress**.
6. Keep each pull request focused on one reviewable improvement.
7. Run the relevant tests and update documentation before marking it **Done**.

Create a GitHub issue when an item needs upstream discussion, multi-person
coordination, or a dedicated conversation history. Until then, this document is
the source of truth for the improvement backlog.
