# Day 22 — Interview Prep Day 22

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Migrations & Seeders
```php
Schema::table('posts', function(Blueprint $t){ $t->softDeletes(); });
Post::factory()->count(50)->create();
```

#### Factories & Relationships
```php
Post::factory()->for(User::factory())->has(Comment::factory()->count(3))->create();
```

#### DTOs
```php
class CreatePostDTO{ public function __construct(public string $title, public string $body){} }
```

#### Design Pattern — Unit of Work
```php
// DB::transaction onboarding workflow
```
```php
// DB::transaction onboarding workflow
```

#### MySQL — EXPLAIN
```sql
-- EXPLAIN fields: type, rows, filtered, Extra
```

## Frontend (FE)

#### Infinite Query
```tsx
/* useInfiniteQuery basic */
```

#### Suspense + Split
```tsx
/* lazy() + <Suspense/> */
```

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a full‑stack developer strong in API design, query optimization and pragmatic UI state management. Today I focused on interview prep day 22 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: backpressure, circuit breaker, consistency, idempotency, observability, partitioning, retries, throughput
- **Technical Q&A practice**:  
  1) Explain a time you eliminated an N+1 query.  
  2) Explain a time you eliminated an N+1 query.
- **Behavioral (STAR)**: Tell me about a time you had a production incident. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Migrations & Seeders, Factories & Relationships, DTOs
- React hooks: Infinite Query, Suspense + Split
- Pattern: Unit of Work | MySQL: EXPLAIN
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 22** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Laravel](https://laravel.com/docs)
- [React Query](https://tanstack.com/query/latest)

### Checklist
- [ ] Implement BE + FE
- [ ] Test (BE/FE) pass
- [ ] EXPLAIN/metrics captured
- [ ] 2 English answers recorded


### Reflection
- Which hook or query optimization was most impactful today? Why?