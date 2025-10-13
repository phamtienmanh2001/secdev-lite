# S05 – Option Matrix for R-01 (JWT TTL + Refresh + Rotation)

| Option | Description | Security impact | Blast radius | Complexity | Time | Dependencies | Benefit | Cost | Net | Note |
|--------|--------------|----------------:|-------------:|------------:|------:|--------------:|---------:|------:|------:|------|
| O1 | Access token TTL 15 m + Refresh 7 d + rotation and revoke list | 5 | 4 | 3 | 2 | 2 | 9 | 7 | +2 | Balanced security and maintainability |
| O2 | Long-lived JWT (24 h) + IP/device binding | 3 | 3 | 2 | 2 | 3 | 6 | 7 | -1 | Hard to revoke stolen tokens |
| O3 | Stateful sessions + CSRF defense | 4 | 4 | 4 | 3 | 3 | 8 | 10 | -2 | Secure but increases infrastructure load |

**Decision:** O1 – standard practice for modern REST APIs.
