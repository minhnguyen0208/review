# Day 20 — Interview Prep Day 20

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Job Middleware
```php
class SkipIfStale {
  public function handle($job, $next){ if($job->updatedRecently()) return; $next($job); }
}
```

#### Horizon
```txt
- Monitor queue throughput, failures, retry patterns; set maxTries/backoff per queue.
```

#### Observers
```php
Post::observe(PostObserver::class); // created/updated/deleted hooks
```

#### Design Pattern — Strategy
```php
// Strategy example with CheckoutService + binding
```
```php
// Strategy example with CheckoutService + binding
```

#### MySQL — Idempotency Key
```php
// Store SHA-256 key + response, upsert on first success
```

## Frontend (FE)

#### React Query List
```tsx
/* list with staleTime, error handling */
```

#### React Query Mutation
```tsx
/* optimistic update with rollback */
```

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a full‑stack developer strong in API design, query optimization and pragmatic UI state management. Today I focused on interview prep day 20 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: backpressure, debounce, isolation level, latency, memoization, optimistic UI, partitioning, rate limiting
- **Technical Q&A practice**:  
  1) Explain a time you eliminated an N+1 query.  
  2) Explain a time you eliminated an N+1 query.
- **Behavioral (STAR)**: Tell me about a time you had a production incident. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Job Middleware, Horizon, Observers
- React hooks: React Query List, React Query Mutation
- Pattern: Strategy | MySQL: Idempotency Key
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 20** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Laravel](https://laravel.com/docs)
- [Queues](https://laravel.com/docs/queues)
- [React Query](https://tanstack.com/query/latest)

### Checklist
- [ ] Implement BE + FE
- [ ] Test (BE/FE) pass
- [ ] EXPLAIN/metrics captured
- [ ] 2 English answers recorded


### Reflection
- Which hook or query optimization was most impactful today? Why?