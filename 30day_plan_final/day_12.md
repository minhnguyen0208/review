# Day 12 — Interview Prep Day 12

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Policies & Gates
```php
Gate::define('update-post', fn(User $u, Post $p)=> $u->id===$p->author_id);
$this->authorize('update-post', $post);
```

#### Form Requests
```php
class StorePostRequest extends FormRequest {
  public function rules(){ return ['title'=>'required|string|max:255']; }
  public function authorize(){ return true; }
}
```

#### Design Pattern — Event Sourcing
```txt
// Event store + snapshot explanation
```
```txt
// Event store + snapshot explanation
```

#### MySQL — EXPLAIN
```sql
-- EXPLAIN fields: type, rows, filtered, Extra
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

#### useMemo
```tsx
const expensive = useMemo(()=> heavy(data), [data]); // avoid stale memo pitfalls
```

#### useCallback
```tsx
const onClick = useCallback(()=> doThing(id), [id]); // stable ref for deps
```

#### useRef
```tsx
const ref = useRef<HTMLDivElement>(null); // focus, measure, store mutable value
```

#### useReducer
```tsx
function reducer(s, a){ switch(a.type){ case 'inc': return {...s, n:s.n+1}; default:return s; } }
const [state, dispatch] = useReducer(reducer, { n:0 });
```

#### Custom Hook
```tsx
function useDebounce<T>(value:T, delay=300){ const [v,setV] = useState(value); useEffect(()=>{ const id=setTimeout(()=>setV(value),delay); return ()=>clearTimeout(id);},[value,delay]); return v; }
```

#### React Query List
```tsx
/* list with staleTime, error handling */
```

#### React Query Mutation
```tsx
/* optimistic update with rollback */
```

#### Infinite Query
```tsx
/* useInfiniteQuery basic */
```

#### Suspense + Split
```tsx
/* lazy() + <Suspense/> */
```

#### A11y Modal
```tsx
/* focus trap & ESC close */
```

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a engineer with hands‑on experience scaling REST APIs and front-end performance tuning. Today I focused on interview prep day 12 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: circuit breaker, consistency, isolation level, latency, optimistic UI, rate limiting, retries, throughput
- **Technical Q&A practice**:  
  1) Explain a time you eliminated an N+1 query.  
  2) How do you ensure idempotency for create endpoints under retries?
- **Behavioral (STAR)**: Tell me about a time you had a production incident. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Policies & Gates, Form Requests
- React hooks: useState, useEffect, useMemo, useCallback, useRef, useReducer, Custom Hook, React Query List, React Query Mutation, Infinite Query, Suspense + Split, A11y Modal
- Pattern: Event Sourcing | MySQL: EXPLAIN
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 12** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Laravel](https://laravel.com/docs)
- [React](https://react.dev/learn)
- [React Query](https://tanstack.com/query/latest)
- [Validation](https://laravel.com/docs/validation)

### Checklist
- [ ] Implement BE + FE
- [ ] Test (BE/FE) pass
- [ ] EXPLAIN/metrics captured
- [ ] 2 English answers recorded


### Reflection
- Which hook or query optimization was most impactful today? Why?