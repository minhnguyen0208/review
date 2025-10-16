# Day 15 — Interview Prep Day 15

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Error Handling & Exceptions
```php
// app/Exceptions/Handler.php => render(): normalize 4xx/5xx JSON; add context IDs.
```

#### Rate Limiting
```php
RateLimiter::for('api', fn($r)=> Limit::perMinute(120)->by($r->ip()));
```

#### Design Pattern — Repository
```php
// Repository interface + Eloquent repository + controller usage
```
```php
// Repository interface + Eloquent repository + controller usage
```

#### MySQL — Idempotency Key
```php
// Store SHA-256 key + response, upsert on first success
```

## Frontend (FE)

#### useMemo
```tsx
const expensive = useMemo(()=> heavy(data), [data]); // avoid stale memo pitfalls
```

#### useCallback
```tsx
const onClick = useCallback(()=> doThing(id), [id]); // stable ref for deps
```

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a engineer with hands‑on experience scaling REST APIs and front-end performance tuning. Today I focused on interview prep day 15 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: circuit breaker, debounce, denormalization, idempotency, isolation level, memoization, partitioning, retries
- **Technical Q&A practice**:  
  1) How do you ensure idempotency for create endpoints under retries?  
  2) Explain a time you eliminated an N+1 query.
- **Behavioral (STAR)**: Tell me about a time you had a production incident. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Error Handling & Exceptions, Rate Limiting
- React hooks: useMemo, useCallback
- Pattern: Repository | MySQL: Idempotency Key
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 15** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
- [Cache](https://laravel.com/docs/cache)
- [Database](https://dev.mysql.com/doc/)
- [Laravel](https://laravel.com/docs)
- [React](https://react.dev/learn)

### Checklist
- [ ] Implement BE + FE
- [ ] Test (BE/FE) pass
- [ ] EXPLAIN/metrics captured
- [ ] 2 English answers recorded


### Reflection
- Which hook or query optimization was most impactful today? Why?