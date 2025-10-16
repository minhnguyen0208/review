# Day 30 — Interview Prep Day 30

_Generated on 2025-10-01 04:00._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Auth+Sanctum
```php
$token = $user->createToken('web')->plainTextToken;
Route::middleware('auth:sanctum')->get('/me', fn(Request $r)=> $r->user());
```

#### S3 Multipart
```php
// 1) init multipart -> 2) generate signed part URLs -> 3) complete
```

#### Design Pattern — Event Sourcing
#### Architectural Pattern — Event Sourcing (Concept + Snapshot)
```txt
- Persist events: OrderCreated, ItemAdded, Paid, Shipped...
- Rehydrate aggregate by replaying events; snapshot mỗi N events để nhanh hơn.
- Projections để query nhanh.
```
**Interview angle**: audit trail tự nhiên, rollback theo thời gian; nhưng **dev cost cao**.


#### MySQL — Idempotency Key
#### MySQL — Idempotent API (table + usage)
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
**Interview**: giải thích thứ tự thao tác để tránh race (unique + upsert).


## Frontend (FE)

#### React Query — List
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

#### React Query — Mutation (Optimistic)
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

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 30 with emphasis on reliability and performance.*
- **Tech Qs**:  
  1) *How do you ensure idempotency for mutations?*  
  2) *Explain a query optimization you shipped and its impact.*


### Interview Focus
- Patterns: Event Sourcing
- MySQL: Idempotency Key
- Pitfalls:
- Avoid SELECT *; keep covering index effective.
- Set consistent ordering for pagination.
- Add jitter to retries to avoid herd effect.
- Keep transactions short; avoid long-held locks.

**Case Study D30**: Implement idempotent create-order with DB upsert + Redis lock; return 201 on first call, 200 cached for retries.

### Practice Tasks
- Build the feature end-to-end; write why each design choice fits interview expectations.

### Acceptance Criteria
- BE: đúng schema/status; test Feature; log structured.
- FE: list/mutation OK; handle loading/error/empty; mutation rollback tốt.
- DB: có EXPLAIN + index/partition đề xuất.


### Docs & References
- [AWS S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html)
- [Database](https://dev.mysql.com/doc/)
- [Laravel](https://laravel.com/docs)
- [React Query](https://tanstack.com/query/latest)
- [Validation](https://laravel.com/docs/validation)

### Checklist
- [ ] Code BE+FE end-to-end
- [ ] 1 Feature test (BE) + 1 RTL/Playwright (FE)
- [ ] Ghi chú EXPLAIN & tối ưu
- [ ] 2 câu trả lời kỹ thuật (EN)


### Reflection
- What trade-off today would you defend in an interview? Why?