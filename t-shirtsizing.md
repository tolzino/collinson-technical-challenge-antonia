
---

T-Shirt Sizing

| Component | Size | Rationale | Key Risks |
|------------|------|------------|------------|
| GraphQL BFF / API Gateway | M | Aggregates services, auth validation, request shaping | Schema drift, versioning complexity |
| Booking Service | L | Transactional logic, idempotency, payment coordination, state management | Double booking, payment failure handling |
| Journey Planning Service | M | Traffic + flight API aggregation, ETA calculations | External API latency, caching correctness |
| Inventory & Partner Content Service | M | Partner availability + content integration | API contract volatility |
| PostgreSQL | M | Transactional persistence, indexing, migrations | Schema evolution |
| Redis | S | Short-lived caching layer | Stale data tolerance |
| Event Bus | S/M | Asynchronous event propagation | Message ordering, retry logic |
| Observability | M | Logging, metrics, tracing integration | Alert fatigue if misconfigured |

---
