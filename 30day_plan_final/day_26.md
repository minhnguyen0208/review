# Day 26 — Interview Prep Day 26

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Observers
```php
Post::observe(PostObserver::class); // created/updated/deleted hooks
```

#### Casting & Accessors/Mutators
```php
protected $casts = ['data'=>'array'];
public function getTitleAttribute($v){ return trim($v); }
```

#### Throttling & Abuse
```php
// combine RateLimiter + login velocity checks; interview: false positives
```

#### Design Pattern — Strategy
```php
// Strategy example with CheckoutService + binding
```
```php
// Strategy example with CheckoutService + binding
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

#### useEffect
```tsx
useEffect(()=>{ document.title = `Count ${count}`; }, [count]); // cleanup when needed
```

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a full‑stack developer strong in API design, query optimization and pragmatic UI state management. Today I focused on interview prep day 26 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: backpressure, debounce, latency, observability, optimistic UI, partitioning, rate limiting, rollbacks
- **Technical Q&A practice**:  
  1) How do you ensure idempotency for create endpoints under retries?  
  2) Explain a time you eliminated an N+1 query.
- **Behavioral (STAR)**: Describe a disagreement on design and how you resolved it. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Observers, Casting & Accessors/Mutators, Throttling & Abuse
- React hooks: useState, useEffect
- Pattern: Strategy | MySQL: Covering Index
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 26** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
- [Cache](https://laravel.com/docs/cache)
- [Database](https://dev.mysql.com/doc/)
- [Eloquent](https://laravel.com/docs/eloquent)
- [Laravel](https://laravel.com/docs)
- [React](https://react.dev/learn)

### Checklist
- [ ] Implement BE + FE
- [ ] Test (BE/FE) pass
- [ ] EXPLAIN/metrics captured
- [ ] 2 English answers recorded


### Reflection
- Which hook or query optimization was most impactful today? Why?