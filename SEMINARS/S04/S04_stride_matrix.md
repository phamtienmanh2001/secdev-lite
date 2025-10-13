# S04 - STRIDE per element (матрица)


| Element | Data/Boundary | Threat (S/T/R/I/D/E) | Description | NFR link (ID) | Mitigation idea (ADR later) |
|:-----------------------------|:----------------|:-----------------------|:------------------------------------------------------------------|:-----------------|:------------------------------|
| Edge: Internet → API | JWT/public | S | Повтор/подмена токена, reuse истёкшего/украденного JWT. | NFR-001 | JWT TTL + Refresh + Rotation |
| Edge: Internet → API | Requests | T | Грязный ввод: extra поля, oversize payload, несоответствие схеме. | NFR-002, NFR-004 | Input Validation & Size Limits; reject extra fields |
| Edge: Internet → API | Traffic | D | Злоупотребление частотой запросов, brute-force/регистрация. | NFR-003 | Rate Limiting (≤5 req/min/IP) + 429 + Retry-After  |
| Node: API/Controller | Errors | R | Неединый формат ошибок, отсутствует correlation_id. | NFR-004 | API Errors: RFC7807 + correlation_id |
| Node: Service | Logs | I | PII в логах/ошибках, чрезмерные детали. | NFR-005 | PII masking in logs + retention ≤30d |
| Node: Service | RBAC/Tenant | E | Обход ролей/межтенантная утечка. | NFR-006 | RBAC + Tenant Isolation (deny-by-default, per-request tenant scoping) |
| Edge: Service → DB | SQL/ORM | T | Инъекции/неканоничные значения при записи. | NFR-007 | DB Safety: parameterized queries + canonicalization |
| Node: Admin paths | Audit | R | Нет неизменяемого журнала критических операций. | NFR-008 | Audit Log for Admin Actions |
| Edge: Service → External API | HTTP/gRPC | D | Отсутствуют timeout/retry/circuit-breaker. | NFR-009 | Timeouts ≤2s + Retry≤3 with jitter + Circuit Breaker (50%/1m) |
