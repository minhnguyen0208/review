# Day 23 — Repository + Specification

_Generated from your plan on 2025-10-01 06:24._

### Plan Notes
_(none)_

## Backend (BE)

```php
interface PostRepository { public function paginate(array $filters,int $perPage=20): \Illuminate\Contracts\Pagination\LengthAwarePaginator; }
class EloquentPostRepository implements PostRepository { public function paginate(array $filters,int $perPage=20){ return Post::query()->when($filters['q']??null, fn($q,$v)=>$q->whereFullText('title',$v))->paginate($perPage);} }
```

## Frontend (FE)

```tsx
const [count,setCount] = useState(0);
```

```tsx
useEffect(()=>{ document.title = `Count ${count}`; return ()=>{/* cleanup */}; }, [count]);
```

```tsx
const expensive = useMemo(()=> heavy(data), [data]);
```

```tsx
const onClick = useCallback(()=> doThing(id), [id]);
```

### English (Interview)
- **Pitch (30–45s)**: *Hi, I'm Minh, a Laravel + React engineer. Today I focused on repository + specification.*
- **Tech Qs**: 1) *Walk through API → DB → UI.* 2) *How to avoid N+1 and ensure idempotency?*
- **Vocab**: latency, throughput, idempotency, pagination, memoization, partitioning, rollback, retry


### Practice Tasks
- Build end-to-end; add 1 BE + 1 FE test; capture EXPLAIN & index/partition note.

### Acceptance Criteria
- Correct schema/status + normalized error; FE handles loading/error/empty; show EXPLAIN.

### Docs & References
- (add relevant docs)

### Reflection
- Which trade-off today will you defend in interview and why?