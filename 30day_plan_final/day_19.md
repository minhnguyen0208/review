# Day 19 — Interview Prep Day 19

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Broadcasting
```php
broadcast(new OrderCreated($order))->toOthers(); // Pusher/socket
```

#### Job Middleware
```php
class SkipIfStale {
  public function handle($job, $next){ if($job->updatedRecently()) return; $next($job); }
}
```

#### Feature Flags
```php
if(Feature::enabled('new-header')){ /* render new */ } else { /* old */ }
```

#### Design Pattern — Factory
```php
// Factory example included above in BE_TOPICS where used in services
```
```php
// Factory example included above in BE_TOPICS where used in services
```

#### MySQL — Batch Upsert
```php
// Model::upsert($rows, ['k'], ['col1','col2'])
```

## Frontend (FE)

#### Custom Hook
```tsx
function useDebounce<T>(value:T, delay=300){ const [v,setV] = useState(value); useEffect(()=>{ const id=setTimeout(()=>setV(value),delay); return ()=>clearTimeout(id);},[value,delay]); return v; }
```

#### React Query List
```tsx
/* list with staleTime, error handling */
```

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a backend-leaning full‑stack engineer with solid Laravel + React experience, focused on reliability and throughput. Today I focused on interview prep day 19 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: consistency, debounce, isolation level, latency, memoization, retries, rollbacks, throughput
- **Technical Q&A practice**:  
  1) Explain a time you eliminated an N+1 query.  
  2) Walk me through how you designed pagination to avoid duplicates.
- **Behavioral (STAR)**: Tell me about a time you had a production incident. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Broadcasting, Job Middleware, Feature Flags
- React hooks: Custom Hook, React Query List
- Pattern: Factory | MySQL: Batch Upsert
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 19** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Laravel](https://laravel.com/docs)
- [Queues](https://laravel.com/docs/queues)
- [React](https://react.dev/learn)
- [React Query](https://tanstack.com/query/latest)

### Checklist
- [ ] Implement BE + FE
- [ ] Test (BE/FE) pass
- [ ] EXPLAIN/metrics captured
- [ ] 2 English answers recorded


### Reflection
- Which hook or query optimization was most impactful today? Why?