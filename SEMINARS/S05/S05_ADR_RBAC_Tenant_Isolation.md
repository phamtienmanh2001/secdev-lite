# ADR: RBAC + Tenant Isolation (deny-by-default)
Status: Proposed  

## Context
- **Risk ID:** R-05 (Cross-tenant access, Elevation of Privilege) (L = 4, I = 5, Score = 20)
- **DFD Element:** Node: Service (Business Logic)
- **Linked NFR:** NFR-006 (RBAC/AuthZ)
- **Assumptions:** tenant_id is available in JWT; repository layer supports tenant scoping.

## Decision
Implement deny-by-default RBAC inside the service layer with repository-level tenant filtering.  
Authorization middleware will map each action to specific roles (ADMIN, STAFF, USER).  
All access errors follow RFC 7807 format with correlation_id.

## Alternatives
- Gateway-level RBAC: insufficient — cannot prevent internal bypass.  
- Per-tenant database: secure but operationally expensive.  

## Consequences
- (+) Strong logical isolation and auditability.  
- (-) Adds moderate query and policy complexity.

## DoD / Acceptance (Given–When–Then)
- **Given** user from tenant A  
- **When** accessing tenant B data  
- **Then** system returns 403 (RFC 7807) with no ID leakage.

**Check:** e2e tests (`rbac-tenant-isolation`), audit logs, and policy snapshots.

## Rollback / Fallback
Feature-flag policy enforcement to temporarily relax restrictions if blocking errors occur.

## Trace
- [Link to DFD (S04_dfd.md)](../S04/S04_dfd.md)
- S04: R-05 (STRIDE=E, Node: Service)  
- S03: NFR-006 (RBAC/AuthZ)

## Ownership & Dates
Owner: Фам Тиен Мань

Reviewers: Нгуен Дык Хю

Date created: <YYYY-MM-DD> 2025-10-10

Last updated: <YYYY-MM-DD> 2025-10-10