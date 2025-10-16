# Day 10 — Interview Prep Day 10

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Providers & Config
```php
// AppServiceProvider::boot() => Schema::defaultStringLength(191);
// config('app.env'); php artisan config:cache (interview: when to use)
```

#### Events & Listeners
```php
// event(new UserRegistered($user->id));
class SendWelcome implements ShouldQueue { public function handle(UserRegistered $e){ /* ... */ } }
```

#### Design Pattern — Unit of Work
```php
// DB::transaction onboarding workflow
```
```php
// DB::transaction onboarding workflow
```

#### MySQL — Idempotency Key
```php
// Store SHA-256 key + response, upsert on first success
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

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a backend-leaning full‑stack engineer with solid Laravel + React experience, focused on reliability and throughput. Today I focused on interview prep day 10 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: backpressure, circuit breaker, idempotency, observability, rate limiting, retries, rollbacks, throughput
- **Technical Q&A practice**:  
  1) How do you ensure idempotency for create endpoints under retries?  
  2) How do you ensure idempotency for create endpoints under retries?
- **Behavioral (STAR)**: Describe a disagreement on design and how you resolved it. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Providers & Config, Events & Listeners
- React hooks: useState, useEffect, useMemo, useCallback, useRef, useReducer, Custom Hook, React Query List, React Query Mutation, Infinite Query
- Pattern: Unit of Work | MySQL: Idempotency Key
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 10** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
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

### Checklist
- [ ] Implement BE + FE
- [ ] Test (BE/FE) pass
- [ ] EXPLAIN/metrics captured
- [ ] 2 English answers recorded


### Reflection
- Which hook or query optimization was most impactful today? Why?