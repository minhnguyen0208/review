# Day 24 — Interview Prep Day 24

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Testing (Pest advanced)
```php
test('api returns resource shape', function() {
  $res = $this->getJson('/api/posts')->assertOk()->json();
  expect($res)->toHaveKeys(['data','links','meta']);
});
```

#### Feature Flags
```php
if(Feature::enabled('new-header')){ /* render new */ } else { /* old */ }
```

#### Query Builder vs Eloquent
```php
DB::table('posts')->where('published',1)->count(); // faster aggregate than Eloquent
```

#### Design Pattern — Event Sourcing
```txt
// Event store + snapshot explanation
```
```txt
// Event store + snapshot explanation
```

#### MySQL — Batch Upsert
```php
// Model::upsert($rows, ['k'], ['col1','col2'])
```

## Frontend (FE)

#### A11y Modal
```tsx
/* focus trap & ESC close */
```

#### Error Boundary
```tsx
/* componentDidCatch demo */
```

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a full‑stack developer strong in API design, query optimization and pragmatic UI state management. Today I focused on interview prep day 24 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: circuit breaker, consistency, denormalization, idempotency, isolation level, rate limiting, rollbacks, throughput
- **Technical Q&A practice**:  
  1) Walk me through how you designed pagination to avoid duplicates.  
  2) Explain a time you eliminated an N+1 query.
- **Behavioral (STAR)**: Describe a disagreement on design and how you resolved it. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Testing (Pest advanced), Feature Flags, Query Builder vs Eloquent
- React hooks: A11y Modal, Error Boundary
- Pattern: Event Sourcing | MySQL: Batch Upsert
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 24** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Laravel](https://laravel.com/docs)

### Checklist
- [ ] Implement BE + FE
- [ ] Test (BE/FE) pass
- [ ] EXPLAIN/metrics captured
- [ ] 2 English answers recorded


### Reflection
- Which hook or query optimization was most impactful today? Why?