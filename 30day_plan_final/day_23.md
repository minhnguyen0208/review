# Day 23 — Interview Prep Day 23

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Factories & Relationships
```php
Post::factory()->for(User::factory())->has(Comment::factory()->count(3))->create();
```

#### Testing (Pest advanced)
```php
test('api returns resource shape', function() {
  $res = $this->getJson('/api/posts')->assertOk()->json();
  expect($res)->toHaveKeys(['data','links','meta']);
});
```

#### Soft Deletes
```php
use SoftDeletes; Post::withTrashed()->find($id); Post::onlyTrashed()->restore();
```

#### Design Pattern — CQRS
```php
// Command handler + projector outline
```
```php
// Command handler + projector outline
```

#### MySQL — Partition by Date
```sql
-- Range partition with TO_DAYS(created_at)
```

## Frontend (FE)

#### Suspense + Split
```tsx
/* lazy() + <Suspense/> */
```

#### A11y Modal
```tsx
/* focus trap & ESC close */
```

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a full‑stack developer strong in API design, query optimization and pragmatic UI state management. Today I focused on interview prep day 23 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: debounce, denormalization, isolation level, latency, observability, optimistic UI, rate limiting, retries
- **Technical Q&A practice**:  
  1) Walk me through how you designed pagination to avoid duplicates.  
  2) How do you ensure idempotency for create endpoints under retries?
- **Behavioral (STAR)**: Describe a disagreement on design and how you resolved it. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Factories & Relationships, Testing (Pest advanced), Soft Deletes
- React hooks: Suspense + Split, A11y Modal
- Pattern: CQRS | MySQL: Partition by Date
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 23** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Eloquent](https://laravel.com/docs/eloquent)
- [Laravel](https://laravel.com/docs)

### Checklist
- [ ] Implement BE + FE
- [ ] Test (BE/FE) pass
- [ ] EXPLAIN/metrics captured
- [ ] 2 English answers recorded


### Reflection
- Which hook or query optimization was most impactful today? Why?