# S05 – Option Matrix for R-05 (RBAC + Tenant Isolation)

| Option | Description | Security impact | Blast radius | Complexity | Time | Dependencies | Benefit | Cost | Net | Note |
|--------|--------------|--------------------:|-----------------:|---------------:|---------------------:|-----------------:|---------:|------:|------:|------|
| O1 | Deny-by-default RBAC in service layer + tenant filter in repository | 5 | 5 | 3 | 2 | 2 | 10 | 7 | +3 | Best trade-off between security and complexity |
| O2 | RBAC checks only at API gateway | 3 | 3 | 2 | 2 | 2 | 6 | 6 | 0 | Easier but can be bypassed inside microservices |
| O3 | Separate database/schema per tenant | 5 | 5 | 5 | 4 | 4 | 10 | 13 | -3 | Very secure but high operational cost |

**Decision:** Choose O1 – strong isolation with minimal complexity.
