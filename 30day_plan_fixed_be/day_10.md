# Day 10 — Interview Prep Day 10

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Events & Listeners
```php
event(new UserRegistered($user->id));
class SendWelcome implements ShouldQueue { public function handle(UserRegistered $e){ /* send mail */ } }
```

#### Policies & Gates
```php
Gate::define('update-post', fn(User $u, Post $p)=> $u->id===$p->author_id);
$this->authorize('update-post', $post);
```

#### Design Pattern — Unit of Work
```php
class UserOnboarding {
  public function run(array $data){
    DB::transaction(function() use ($data){
      $user = User::create($data['user']);
      Profile::create(['user_id'=>$user->id] + $data['profile']);
      event(new UserOnboarded($user->id));
    });
  }
}
```

#### MySQL — Idempotency Key
```sql
CREATE TABLE api_idempotency (
  key_hash VARBINARY(32) PRIMARY KEY,
  response MEDIUMBLOB,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```
```php
$key = hash('sha256', $request->header('X-Idempotency-Key').$request->getContent());
$hit = DB::table('api_idempotency')->where('key_hash', $key)->first();
if($hit){ return response()->json(json_decode($hit->response,true), 200); }
$result = doWork($request->all());
DB::table('api_idempotency')->updateOrInsert(['key_hash'=>$key], ['response'=>json_encode($result)]);
return response()->json($result, 201);
```
**Race**: rely on PK + upsert to prevent duplicates.


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

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 10 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: consistency, debounce, denormalization, idempotency, isolation level, partitioning, rate limiting, rollback
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

### Reflection
- What trade-off today will you highlight in an interview?