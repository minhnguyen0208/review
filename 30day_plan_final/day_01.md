# Day 01 — Interview Prep Day 1

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### API+Validation+Resource
```txt
API+Validation+Resource details here.
```

#### Eloquent Performance
```txt
Eloquent Performance details here.
```

#### Design Pattern — Factory
```php
// Factory example included above in BE_TOPICS where used in services
```
```php
// Factory example included above in BE_TOPICS where used in services
```

#### MySQL — Covering Index
```sql
-- See expanded example; ensure idx(author_id, created_at desc)
```

## Frontend (FE)

#### useState
```tsx
const [count,setCount] = useState(0);
```

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a full‑stack developer strong in API design, query optimization and pragmatic UI state management. Today I focused on interview prep day 1 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: circuit breaker, isolation level, latency, memoization, rate limiting, retries, rollbacks, throughput
- **Technical Q&A practice**:  
  1) Explain a time you eliminated an N+1 query.  
  2) Explain a time you eliminated an N+1 query.
- **Behavioral (STAR)**: Describe a disagreement on design and how you resolved it. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: API+Validation+Resource, Eloquent Performance
- React hooks: useState
- Pattern: Factory | MySQL: Covering Index
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 1** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Eloquent](https://laravel.com/docs/eloquent)
- [React](https://react.dev/learn)

### Checklist
- [ ] Implement BE + FE
- [ ] Test (BE/FE) pass
- [ ] EXPLAIN/metrics captured
- [ ] 2 English answers recorded


### Reflection
- Which hook or query optimization was most impactful today? Why?