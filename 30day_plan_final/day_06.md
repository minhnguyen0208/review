# Day 06 — Interview Prep Day 6

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Auth+Sanctum
```txt
Auth+Sanctum details here.
```

#### Indexing+EXPLAIN
```sql
CREATE INDEX idx_posts_author_created ON posts(author_id, created_at DESC);
EXPLAIN SELECT id,title,created_at FROM posts WHERE author_id=? ORDER BY created_at DESC LIMIT 20;
```

#### Design Pattern — Event Sourcing
```txt
// Event store + snapshot explanation
```
```txt
// Event store + snapshot explanation
```

#### MySQL — Covering Index
```sql
-- See expanded example; ensure idx(author_id, created_at desc)
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

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a engineer with hands‑on experience scaling REST APIs and front-end performance tuning. Today I focused on interview prep day 6 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: consistency, debounce, denormalization, idempotency, isolation level, retries, rollbacks, throughput
- **Technical Q&A practice**:  
  1) Walk me through how you designed pagination to avoid duplicates.  
  2) How do you ensure idempotency for create endpoints under retries?
- **Behavioral (STAR)**: Describe a disagreement on design and how you resolved it. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Auth+Sanctum, Indexing+EXPLAIN
- React hooks: useState, useEffect, useMemo, useCallback, useRef, useReducer
- Pattern: Event Sourcing | MySQL: Covering Index
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 6** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [React](https://react.dev/learn)
- [Sanctum](https://laravel.com/docs/sanctum)

### Checklist
- [ ] Implement BE + FE
- [ ] Test (BE/FE) pass
- [ ] EXPLAIN/metrics captured
- [ ] 2 English answers recorded


### Reflection
- Which hook or query optimization was most impactful today? Why?