# Day 21 — Interview Prep Day 21

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Horizon
```txt
- Monitor queue throughput, failures, retry patterns; set maxTries/backoff per queue.
```

#### Migrations & Seeders
```php
Schema::table('posts', function(Blueprint $t){ $t->softDeletes(); });
Post::factory()->count(50)->create();
```

#### Casting & Accessors/Mutators
```php
protected $casts = ['data'=>'array'];
public function getTitleAttribute($v){ return trim($v); }
```

#### Design Pattern — Repository
```php
// Repository interface + Eloquent repository + controller usage
```
```php
// Repository interface + Eloquent repository + controller usage
```

#### MySQL — Covering Index
```sql
-- See expanded example; ensure idx(author_id, created_at desc)
```

## Frontend (FE)

#### React Query Mutation
```tsx
/* optimistic update with rollback */
```

#### Infinite Query
```tsx
/* useInfiniteQuery basic */
```

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a engineer with hands‑on experience scaling REST APIs and front-end performance tuning. Today I focused on interview prep day 21 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: circuit breaker, isolation level, memoization, observability, optimistic UI, rate limiting, rollbacks, throughput
- **Technical Q&A practice**:  
  1) Explain a time you eliminated an N+1 query.  
  2) How do you ensure idempotency for create endpoints under retries?
- **Behavioral (STAR)**: Describe a disagreement on design and how you resolved it. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Horizon, Migrations & Seeders, Casting & Accessors/Mutators
- React hooks: React Query Mutation, Infinite Query
- Pattern: Repository | MySQL: Covering Index
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 21** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Eloquent](https://laravel.com/docs/eloquent)
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