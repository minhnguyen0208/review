# Day 06 — Interview Prep Day 6

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Auth+Sanctum
```php
$token = $user->createToken('web')->plainTextToken;
Route::middleware('auth:sanctum')->get('/me', fn(Request $r)=> $r->user());
```

#### Indexing+EXPLAIN
```sql
-- Create composite index + EXPLAIN example
CREATE INDEX idx_posts_author_created ON posts(author_id, created_at DESC);
EXPLAIN SELECT id,title,created_at FROM posts WHERE author_id=? ORDER BY created_at DESC LIMIT 20;
```

#### Design Pattern — Event Sourcing
```php
// Event table (simplified)
/*
CREATE TABLE events (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  aggregate_id BIGINT, type VARCHAR(64), payload JSON,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
*/
function appendEvent($aggId, $type, $payload){
  DB::table('events')->insert(['aggregate_id'=>$aggId,'type'=>$type,'payload'=>json_encode($payload)]);
}
```

#### MySQL — Covering Index
```sql
-- Bad: SELECT *
SELECT * FROM posts WHERE author_id = ? ORDER BY created_at DESC LIMIT 20;

-- Good: only needed cols + matching composite index
CREATE INDEX idx_posts_author_created ON posts(author_id, created_at DESC);
SELECT id, title, created_at FROM posts WHERE author_id=? ORDER BY created_at DESC LIMIT 20;
```
**Reasoning**: "Using index" in EXPLAIN => avoid back-to-table lookup → lower IO/latency.


## Frontend (FE)

#### useState
```tsx
const [count,setCount] = useState(0);
```

#### useEffect
```tsx
useEffect(()=>{ document.title = `Count ${count}`; return () => {/* cleanup */}; }, [count]);
```

#### useMemo
```tsx
const expensive = useMemo(()=> heavy(data), [data]); // memo only if heavy & stable
```

#### useCallback
```tsx
const onClick = useCallback(()=> doThing(id), [id]); // stable ref for deps
```

#### useRef
```tsx
const inputRef = useRef<HTMLInputElement>(null); useEffect(()=>{ inputRef.current?.focus(); },[]);
```

#### useReducer
```tsx
function reducer(s, a){ switch(a.type){ case 'inc': return {...s, n:s.n+1}; default:return s; } }
const [state, dispatch] = useReducer(reducer, { n:0 });
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 6 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: denormalization, isolation level, latency, memoization, rate limiting, retries, rollback, throughput
- **Q&A**: 1) *How do you ensure idempotency for create endpoints?*  2) *When do you choose client caching vs server caching?*
- **Behavioral (STAR)**: *Tell me about a production incident you owned end‑to‑end.*


### Practice Tasks
- Build feature end-to-end; capture EXPLAIN & tests.
- Record 60s answer on a topic from today.

### Acceptance Criteria
- BE schema/status correct + tests; FE handles loading/error/empty; DB EXPLAIN + index note.

### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Laravel](https://laravel.com/docs)
- [React](https://react.dev/learn)
- [Sanctum](https://laravel.com/docs/sanctum)

### Reflection
- What trade-off today will you highlight in an interview?