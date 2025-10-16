# Day 18 — Interview Prep Day 18

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Notifications
```php
$user->notify(new ResetPasswordNotification($token));
```

#### Broadcasting
```php
broadcast(new OrderCreated($order))->toOthers(); // Pusher/socket
```

#### Testing (Pest advanced)
```php
test('api returns resource shape', function() {
  $res = $this->getJson('/api/posts')->assertOk()->json();
  expect($res)->toHaveKeys(['data','links','meta']);
});
```

#### Design Pattern — Event Sourcing
```txt
// Event store + snapshot explanation
```
```txt
// Event store + snapshot explanation
```

#### MySQL — Partition by Date
```sql
-- Range partition with TO_DAYS(created_at)
```

## Frontend (FE)

#### useReducer
```tsx
function reducer(s, a){ switch(a.type){ case 'inc': return {...s, n:s.n+1}; default:return s; } }
const [state, dispatch] = useReducer(reducer, { n:0 });
```

#### Custom Hook
```tsx
function useDebounce<T>(value:T, delay=300){ const [v,setV] = useState(value); useEffect(()=>{ const id=setTimeout(()=>setV(value),delay); return ()=>clearTimeout(id);},[value,delay]); return v; }
```

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a backend-leaning full‑stack engineer with solid Laravel + React experience, focused on reliability and throughput. Today I focused on interview prep day 18 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: backpressure, consistency, debounce, denormalization, latency, optimistic UI, retries, rollbacks
- **Technical Q&A practice**:  
  1) How do you ensure idempotency for create endpoints under retries?  
  2) Explain a time you eliminated an N+1 query.
- **Behavioral (STAR)**: Tell me about a time you had a production incident. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Notifications, Broadcasting, Testing (Pest advanced)
- React hooks: useReducer, Custom Hook
- Pattern: Event Sourcing | MySQL: Partition by Date
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 18** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Laravel](https://laravel.com/docs)
- [React](https://react.dev/learn)

### Checklist
- [ ] Implement BE + FE
- [ ] Test (BE/FE) pass
- [ ] EXPLAIN/metrics captured
- [ ] 2 English answers recorded


### Reflection
- Which hook or query optimization was most impactful today? Why?