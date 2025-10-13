# ADR: JWT TTL + Refresh + Rotation
Status: Proposed  


## Context
- **Risk ID:** R-01 (Token spoofing/reuse) (L = 4, I = 4, Score = 16)
- **DFD Element:** Edge: Internet → API
- **Linked NFR:** NFR-001 (AuthN)
- **Assumptions:** tokens signed with RS256; kid header supports key rotation.

## Decision
Adopt **short-lived access tokens** (TTL = 15 min) and **rotating refresh tokens** (TTL = 7 days) with revoke list storage.  
Use **RFC 7807** error format and rate-limit login/refresh endpoints.

## Alternatives
- **Long-lived JWT:** easier UX but no immediate revocation.  
- **Server sessions:** secure but requires central session store and CSRF protection.

## Consequences
- (+) Limits exposure window and enables revocation.  
- (-) Requires revoke store and token rotation logic.

## DoD / Acceptance (Given–When–Then)
- **Given** expired or stolen token  
- **When** sending request to API  
- **Then** response is 401/403 RFC 7807; no sensitive info disclosed.

**Check:** Integration tests (`auth-refresh-rotation`), logs, revoke list snapshots.

## Rollback / Fallback
Increase token TTL temporarily or disable rotation flag in configuration.

## Trace
- [Link to DFD (S04_dfd.md)](../S04/S04_dfd.md)
- S04: R-01 (STRIDE=S, Edge: Internet → API)  
- S03: NFR-001 (AuthN)

## Ownership & Dates
Owner: Нгуен Дык Хю

Reviewers: Фам Тиен Мань

Date created: <YYYY-MM-DD> 2025-10-11

Last updated: <YYYY-MM-DD> 2025-10-11