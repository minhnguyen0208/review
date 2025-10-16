# Day 13 — Interview Prep Day 13

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Form Requests
```php
class StorePostRequest extends FormRequest {
  public function rules(){ return ['title'=>'required|string|max:255']; }
  public function authorize(){ return true; }
}
```

#### Resource Collections
```php
return PostResource::collection(Post::latest()->paginate(20));
```

#### Design Pattern — Factory
```php
// Factory example included above in BE_TOPICS where used in services
```
```php
// Factory example included above in BE_TOPICS where used in services
```

#### MySQL — Partition by Date
```sql
-- Range partition with TO_DAYS(created_at)
```

## Frontend (FE)

#### useState
```tsx
const [count,setCount] = useState(0);
```

#### useEffect
```tsx
useEffect(()=>{ document.title = `Count ${count}`; }, [count]); // cleanup when needed
```

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a backend-leaning full‑stack engineer with solid Laravel + React experience, focused on reliability and throughput. Today I focused on interview prep day 13 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: consistency, debounce, denormalization, isolation level, memoization, observability, rollbacks, throughput
- **Technical Q&A practice**:  
  1) Explain a time you eliminated an N+1 query.  
  2) Explain a time you eliminated an N+1 query.
- **Behavioral (STAR)**: Tell me about a time you had a production incident. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Form Requests, Resource Collections
- React hooks: useState, useEffect
- Pattern: Factory | MySQL: Partition by Date
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 13** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
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
- [Validation](https://laravel.com/docs/validation)

### Checklist
- [ ] Implement BE + FE
- [ ] Test (BE/FE) pass
- [ ] EXPLAIN/metrics captured
- [ ] 2 English answers recorded


### Reflection
- Which hook or query optimization was most impactful today? Why?