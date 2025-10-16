# Day 02 — Interview Prep Day 2

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Eloquent Performance
```txt
Eloquent Performance details here.
```

#### Cache+Redis Lock
```txt
Cache+Redis Lock details here.
```

#### Design Pattern — Strategy
```php
// Strategy example with CheckoutService + binding
```
```php
// Strategy example with CheckoutService + binding
```

#### MySQL — EXPLAIN
```sql
-- EXPLAIN fields: type, rows, filtered, Extra
```

## Frontend (FE)

#### useState
```tsx
const [count,setCount] = useState(0);
```

#### useEffect
```tsx
useEffect(()=>{ document.title = `Count ${count}`; }, [count]); // cleanup when needed
```

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a engineer with hands‑on experience scaling REST APIs and front-end performance tuning. Today I focused on interview prep day 2 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: circuit breaker, consistency, debounce, denormalization, idempotency, rate limiting, rollbacks, throughput
- **Technical Q&A practice**:  
  1) Walk me through how you designed pagination to avoid duplicates.  
  2) Walk me through how you designed pagination to avoid duplicates.
- **Behavioral (STAR)**: Tell me about a time you had a production incident. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Eloquent Performance, Cache+Redis Lock
- React hooks: useState, useEffect
- Pattern: Strategy | MySQL: EXPLAIN
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 2** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
- [Cache](https://laravel.com/docs/cache)
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