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

## Agreed MVP

The first deployment will remain backend-only. It will not include an
administrator dashboard, user accounts, or a database-backed API-key registry.

Authenticated public HTTP requests follow this path by default:

```text
MCP client -> HTTPS endpoint -> bearer-key middleware -> FastMCP server
```

The container will load the shared key at startup from `MCP_API_KEY_CURRENT`.
An optional `MCP_API_KEY_PREVIOUS` will support a short, zero-downtime client
migration window during key rotation. Changing either value requires a service
restart.

Local `stdio` transport will remain available for trusted local use and will
not pass through HTTP authentication middleware.

A separate, explicit opt-in may allow unauthenticated HTTP for private
container-network deployments. That mode must remain disabled by default and
must not publish the MCP port to the host or public network.

## Improvement backlog

| Improvement | Priority | Status | Dependencies | Branch / PR |
| --- | --- | --- | --- | --- |
| Document the backend-only deployment architecture | High | Done | None | [#1](https://github.com/alex-yu-101/TWSEMCPServer/pull/1), [#2](https://github.com/alex-yu-101/TWSEMCPServer/pull/2) |
| Add environment-managed bearer-key configuration | High | Done | None | [#1](https://github.com/alex-yu-101/TWSEMCPServer/pull/1) |
| Add bearer-key middleware for HTTP transport | High | Done | Key configuration | [#1](https://github.com/alex-yu-101/TWSEMCPServer/pull/1) |
| Add authentication and transport tests | High | Done | Authentication middleware | [#1](https://github.com/alex-yu-101/TWSEMCPServer/pull/1) |
| Containerize the FastMCP service | High | Done | Authentication configuration | [#2](https://github.com/alex-yu-101/TWSEMCPServer/pull/2) |
| Add container deployment and key-rotation documentation | High | Done | Containerization | [#2](https://github.com/alex-yu-101/TWSEMCPServer/pull/2) |
| Add opt-in unauthenticated HTTP for private container networks | Medium | In progress | Containerization | `feat/optional-http-auth` |
| Add structured security-event logging | Medium | Proposed | Authentication middleware | — |
| Add configurable rate limiting | Medium | Deferred | Authentication middleware | — |
| Add database-backed, per-client API keys | Low | Deferred | Demonstrated multi-client need | — |
| Add API-key management endpoints | Low | Deferred | Database-backed keys | — |
| Add administrator authentication and dashboard | Low | Deferred | Management endpoints; demonstrated operational need | — |

## Scope notes

### Backend-only deployment architecture

The initial deployment will run the Python FastMCP service without a frontend
or persistence layer. HTTPS should terminate at Cloudflare Tunnel or another
trusted reverse proxy. The origin service must not be exposed directly to the
public internet over plaintext HTTP.

The container deployment should use a non-root user, include a health check,
avoid writing secrets into the image, and document which endpoint—if any—is
intentionally allowed without authentication for health monitoring.

### Startup-managed bearer keys

Require a high-entropy secret of at least 32 random bytes through
`MCP_API_KEY_CURRENT`. Accept `MCP_API_KEY_PREVIOUS` only as an optional
rotation aid. Compare presented keys using a constant-time comparison and
never log either configured or presented values.

Production deployments should inject secrets through a Docker secret or a
protected environment file. Secret files and local `.env` files must remain
outside version control.

### HTTP authentication middleware

Protect every MCP HTTP request before it reaches FastMCP tools, including
streaming and session-related routes. Accept credentials only through the
`Authorization: Bearer <key>` header. Missing, malformed, and incorrect
credentials should receive the same generic `401 Unauthorized` response.

Local `stdio` transport is outside this middleware path and should remain
unaffected.

### Private-network unauthenticated HTTP

Support an explicit configuration option that disables HTTP bearer
authentication only for trusted private-network deployments. Authentication
must remain the default, and missing authentication configuration must continue
to fail closed unless the operator deliberately selects unauthenticated mode.

The documented Docker Compose example must keep the MCP port un-published and
connect clients through a named Docker network. Tests must cover the secure
default, explicit opt-in behavior, invalid configuration, and unchanged stdio
operation. The server should log that unauthenticated HTTP mode is active
without logging secrets.

### Tests and documentation

Test successful authentication plus missing, malformed, and incorrect
credentials. Verify that protected requests never reach MCP handlers after an
authentication failure, secrets are not logged, and `stdio` operation still
works. Document secret generation, container startup, client configuration,
rotation, rollback, and recovery.

### Deferred management platform

Database-backed keys, individual client revocation, expiry, scopes, usage
tracking, management endpoints, and an administrator dashboard are not part of
the MVP. Reconsider them only when multiple independently managed clients make
startup-managed shared keys operationally insufficient.

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
