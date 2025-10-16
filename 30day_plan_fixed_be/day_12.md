# Day 12 — Interview Prep Day 12

_Generated on 2025-10-01 04:22._

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

#### MySQL — EXPLAIN
```sql
EXPLAIN SELECT id,title,created_at FROM posts
WHERE author_id = 42
ORDER BY created_at DESC
LIMIT 20;
```
**Read fields**  
- `type`: prefer `ref`/`range`; avoid `ALL`  
- `rows`: est. rows to examine (lower is better)  
- `filtered`: % rows passing filter (higher is better)  
- `Extra`: "Using index" (covering), "Using filesort" (needs better index)


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

#### Custom Hook
```tsx
function useDebounce<T>(value:T, delay=300){ const [v,setV] = useState(value); useEffect(()=>{ const id=setTimeout(()=>setV(value),delay); return ()=>clearTimeout(id);},[value,delay]); return v; }
```

#### React Query List
```tsx
import { useQuery } from '@tanstack/react-query';
async function fetchPosts(){ const r=await fetch('/api/posts'); if(!r.ok) throw new Error('Network'); return r.json(); }
export function Posts(){
  const { data, isLoading, error } = useQuery({ queryKey:['posts'], queryFn: fetchPosts, staleTime: 30_000 });
  if(isLoading) return <p>Loading…</p>;
  if(error) return <p>Failed to load.</p>;
  return <ul>{data.data.map((p:any)=> <li key={p.id}>{p.title}</li>)}</ul>;
}
```

#### React Query Mutation
```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';
function useUpdatePost(){
  const qc = useQueryClient();
  return useMutation({
    mutationFn: (p:any)=> fetch('/api/posts/'+p.id,{method:'PUT',headers:{'Content-Type':'application/json'},body:JSON.stringify(p)}).then(r=>r.json()),
    onMutate: async (p)=>{ await qc.cancelQueries({queryKey:['posts']}); const prev=qc.getQueryData<any>(['posts']); qc.setQueryData(['posts'], (old:any)=> ({...old, data: old.data.map((x:any)=> x.id===p.id?{...x,...p}:x)})); return {prev}; },
    onError: (_e,_p,ctx)=> ctx?.prev && qc.setQueryData(['posts'], ctx.prev),
    onSettled: ()=> qc.invalidateQueries({queryKey:['posts']})
  });
}
```

#### Infinite Query
```tsx
import { useInfiniteQuery } from '@tanstack/react-query';
const fetchPage = async ({pageParam=1})=> (await fetch(`/api/posts?page=${pageParam}`)).json();
export function Feed(){
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({
    queryKey:['posts'], queryFn: fetchPage, getNextPageParam:(last)=> last.next_page ?? false
  });
  // render pages + load more
}
```

#### Suspense + Split
```tsx
import { lazy, Suspense } from 'react';
const Reports = lazy(()=> import('./Reports'));
export default function App(){ return <Suspense fallback={<div>Loading…</div>}><Reports/></Suspense>; }
```

#### A11y Modal
```tsx
// focus first tabbable; ESC to close; restore focus on unmount
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 12 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: consistency, debounce, denormalization, isolation level, latency, memoization, partitioning, rollback
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
- [React Query](https://tanstack.com/query/latest)
- [Validation](https://laravel.com/docs/validation)

### Reflection
- What trade-off today will you highlight in an interview?