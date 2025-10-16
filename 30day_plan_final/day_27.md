# Day 27 — Interview Prep Day 27

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Casting & Accessors/Mutators
```php
protected $casts = ['data'=>'array'];
public function getTitleAttribute($v){ return trim($v); }
```

#### DTOs
```php
class CreatePostDTO{ public function __construct(public string $title, public string $body){} }
```

#### Config Caching
```bash
php artisan config:cache && php artisan route:cache
```

#### Design Pattern — Repository
```php
// Repository interface + Eloquent repository + controller usage
```
```php
// Repository interface + Eloquent repository + controller usage
```

#### MySQL — EXPLAIN
```sql
-- EXPLAIN fields: type, rows, filtered, Extra
```

## Frontend (FE)

#### useEffect
```tsx
useEffect(()=>{ document.title = `Count ${count}`; }, [count]); // cleanup when needed
```

#### useMemo
```tsx
const expensive = useMemo(()=> heavy(data), [data]); // avoid stale memo pitfalls
```

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a backend-leaning full‑stack engineer with solid Laravel + React experience, focused on reliability and throughput. Today I focused on interview prep day 27 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: consistency, debounce, memoization, observability, optimistic UI, partitioning, rollbacks, throughput
- **Technical Q&A practice**:  
  1) Walk me through how you designed pagination to avoid duplicates.  
  2) Walk me through how you designed pagination to avoid duplicates.
- **Behavioral (STAR)**: Describe a disagreement on design and how you resolved it. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Casting & Accessors/Mutators, DTOs, Config Caching
- React hooks: useEffect, useMemo
- Pattern: Repository | MySQL: EXPLAIN
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 27** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
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