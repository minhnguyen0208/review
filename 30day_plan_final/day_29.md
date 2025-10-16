# Day 29 — Interview Prep Day 29

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Soft Deletes
```php
use SoftDeletes; Post::withTrashed()->find($id); Post::onlyTrashed()->restore();
```

#### Query Builder vs Eloquent
```php
DB::table('posts')->where('published',1)->count(); // faster aggregate than Eloquent
```

#### S3 Multipart
```txt
S3 Multipart details here.
```

#### Design Pattern — CQRS
```php
// Command handler + projector outline
```
```php
// Command handler + projector outline
```

#### MySQL — Batch Upsert
```php
// Model::upsert($rows, ['k'], ['col1','col2'])
```

## Frontend (FE)

#### useCallback
```tsx
const onClick = useCallback(()=> doThing(id), [id]); // stable ref for deps
```

#### useRef
```tsx
const ref = useRef<HTMLDivElement>(null); // focus, measure, store mutable value
```

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a engineer with hands‑on experience scaling REST APIs and front-end performance tuning. Today I focused on interview prep day 29 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: circuit breaker, consistency, debounce, latency, memoization, observability, partitioning, throughput
- **Technical Q&A practice**:  
  1) How do you ensure idempotency for create endpoints under retries?  
  2) Walk me through how you designed pagination to avoid duplicates.
- **Behavioral (STAR)**: Tell me about a time you had a production incident. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Soft Deletes, Query Builder vs Eloquent, S3 Multipart
- React hooks: useCallback, useRef
- Pattern: CQRS | MySQL: Batch Upsert
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 29** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
- [AWS S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html)
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