# Day 16 — Interview Prep Day 16

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Rate Limiting
```php
RateLimiter::for('api', fn($r)=> Limit::perMinute(120)->by($r->ip()));
```

#### Validation Rules (Custom)
```php
Validator::extend('safe_slug', fn($attr,$val)=> preg_match('/^[a-z0-9-]+$/',$val));
```

#### Migrations & Seeders
```php
Schema::table('posts', function(Blueprint $t){ $t->softDeletes(); });
Post::factory()->count(50)->create();
```

#### Design Pattern — Unit of Work
```php
// DB::transaction onboarding workflow
```
```php
// DB::transaction onboarding workflow
```

#### MySQL — Covering Index
```sql
-- See expanded example; ensure idx(author_id, created_at desc)
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
- **Elevator pitch (variant)**: *Hi, I'm Minh, a engineer with hands‑on experience scaling REST APIs and front-end performance tuning. Today I focused on interview prep day 16 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: circuit breaker, debounce, idempotency, isolation level, observability, partitioning, retries, throughput
- **Technical Q&A practice**:  
  1) Walk me through how you designed pagination to avoid duplicates.  
  2) How do you ensure idempotency for create endpoints under retries?
- **Behavioral (STAR)**: Describe a disagreement on design and how you resolved it. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Rate Limiting, Validation Rules (Custom), Migrations & Seeders
- React hooks: useCallback, useRef
- Pattern: Unit of Work | MySQL: Covering Index
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 16** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
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
- [Validation](https://laravel.com/docs/validation)

### Checklist
- [ ] Implement BE + FE
- [ ] Test (BE/FE) pass
- [ ] EXPLAIN/metrics captured
- [ ] 2 English answers recorded


### Reflection
- Which hook or query optimization was most impactful today? Why?