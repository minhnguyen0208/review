# Day 28 — Interview Prep Day 28

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### DTOs
```php
class CreatePostDTO{ public function __construct(public string $title, public string $body){} }
```

#### Soft Deletes
```php
use SoftDeletes; Post::withTrashed()->find($id); Post::onlyTrashed()->restore();
```

#### Exception to API Errors
```php
throw ValidationException::withMessages(['email'=>'Already taken']);
```

#### Design Pattern — Unit of Work
```php
// DB::transaction onboarding workflow
```
```php
// DB::transaction onboarding workflow
```

#### MySQL — Partition by Date
```sql
-- Range partition with TO_DAYS(created_at)
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
- **Elevator pitch (variant)**: *Hi, I'm Minh, a backend-leaning full‑stack engineer with solid Laravel + React experience, focused on reliability and throughput. Today I focused on interview prep day 28 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: circuit breaker, consistency, idempotency, observability, optimistic UI, retries, rollbacks, throughput
- **Technical Q&A practice**:  
  1) How do you ensure idempotency for create endpoints under retries?  
  2) How do you ensure idempotency for create endpoints under retries?
- **Behavioral (STAR)**: Tell me about a time you had a production incident. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: DTOs, Soft Deletes, Exception to API Errors
- React hooks: useMemo, useCallback
- Pattern: Unit of Work | MySQL: Partition by Date
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 28** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
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