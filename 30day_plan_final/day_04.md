# Day 04 — Interview Prep Day 4

_Generated on 2025-10-01 04:13._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Queues+Retry+DLQ
```txt
Queues+Retry+DLQ details here.
```

#### Transactions+Outbox
```txt
Transactions+Outbox details here.
```

#### Design Pattern — Unit of Work
```php
// DB::transaction onboarding workflow
```
```php
// DB::transaction onboarding workflow
```

#### MySQL — Batch Upsert
```php
// Model::upsert($rows, ['k'], ['col1','col2'])
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

### English (Interview)
- **Elevator pitch (variant)**: *Hi, I'm Minh, a full‑stack developer strong in API design, query optimization and pragmatic UI state management. Today I focused on interview prep day 4 and distilled the trade-offs I’d discuss in interviews.*
- **Vocabulary**: denormalization, idempotency, isolation level, memoization, observability, rate limiting, rollbacks, throughput
- **Technical Q&A practice**:  
  1) How do you ensure idempotency for create endpoints under retries?  
  2) Walk me through how you designed pagination to avoid duplicates.
- **Behavioral (STAR)**: Describe a disagreement on design and how you resolved it. (S/T/A/R)
**Mini Dialog (mock)**
- *Interviewer*: What are the trade-offs of using Redis locks for idempotency?
- *You*: We prevent duplicate work under concurrency, but we must choose TTL > processing time and add jitter to retries to avoid a thundering herd. We also log lock contention to tune TTL.



### Interview Focus
- Laravel: Queues+Retry+DLQ, Transactions+Outbox
- React hooks: useState, useEffect, useMemo, useCallback
- Pattern: Unit of Work | MySQL: Batch Upsert
- Pitfalls: SELECT *, missing deps in useEffect, missing rollback for optimistic update, long transactions causing deadlocks.


### Practice Tasks
- Build the feature for **Interview Prep Day 4** end-to-end; show a before/after EXPLAIN or a controlled benchmark.
- Record a 60s tech answer on: *How I handle retries + idempotency without data duplication*. 


### Acceptance Criteria
- BE: schema + status codes correct; at least 1 Feature/Pest test; structured error JSON.
- FE: core hooks used correctly; loading/error/empty states handled; mutation rollback works.
- DB: has EXPLAIN and index/partition note where relevant.


### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Queues](https://laravel.com/docs/queues)
- [React](https://react.dev/learn)

### Checklist
- [ ] Implement BE + FE
- [ ] Test (BE/FE) pass
- [ ] EXPLAIN/metrics captured
- [ ] 2 English answers recorded


### Reflection
- Which hook or query optimization was most impactful today? Why?