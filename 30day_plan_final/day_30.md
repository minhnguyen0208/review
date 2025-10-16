# Day 30 — Interview Prep Day 30

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Query Builder vs Eloquent
```php
DB::table('posts')->where('published',1)->count(); // faster aggregate than Eloquent
```

#### API Versioning
```php
Route::prefix('v1')->group(fn()=> Route::apiResource('posts', V1\PostController::class));
```

#### API+Validation+Resource
```txt
API+Validation+Resource details here.
```

#### Design Pattern — Event Sourcing
```txt
// Event store + snapshot explanation
```
```txt
// Event store + snapshot explanation
```

#### MySQL — Idempotency Key
```php
// Store SHA-256 key + response, upsert on first success
```

## Frontend (FE)

#### useRef
```tsx
const ref = useRef<HTMLDivElement>(null); // focus, measure, store mutable value
```

#### useReducer
```tsx
function reducer(s, a){ switch(a.type){ case 'inc': return {...s, n:s.n+1}; default:return s; } }
const [state, dispatch] = useReducer(reducer, { n:0 });
```

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a full‑stack developer strong in API design, query optimization and pragmatic UI state management. Today I focused on interview prep day 30 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: circuit breaker, consistency, debounce, idempotency, isolation level, memoization, partitioning, throughput
- **Technical Q&A practice**:  
  1) Explain a time you eliminated an N+1 query.  
  2) Walk me through how you designed pagination to avoid duplicates.
- **Behavioral (STAR)**: Tell me about a time you had a production incident. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Query Builder vs Eloquent, API Versioning, API+Validation+Resource
- React hooks: useRef, useReducer
- Pattern: Event Sourcing | MySQL: Idempotency Key
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 30** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
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