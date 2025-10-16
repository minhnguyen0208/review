# Day 15 — Interview Prep Day 15

_Generated on 2025-10-01 04:00._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### S3 Multipart
```php
// 1) init multipart -> 2) generate signed part URLs -> 3) complete
```

#### Indexing+EXPLAIN
#### MySQL — Covering Index (Stop SELECT *)
```sql
-- Bad
SELECT * FROM posts WHERE author_id = ? ORDER BY created_at DESC LIMIT 20;

-- Good: only needed cols + composite index
CREATE INDEX idx_posts_author_created ON posts(author_id, created_at DESC);

SELECT id, title, created_at
FROM posts
WHERE author_id = ?
ORDER BY created_at DESC
LIMIT 20;
```
**Why**: "Using index" => tránh back-to-table lookup → giảm IO, latency.


#### MySQL — Đọc EXPLAIN (example)
```sql
EXPLAIN SELECT id,title,created_at FROM posts
WHERE author_id = 42
ORDER BY created_at DESC
LIMIT 20;
```
**Key fields**  
- `type`: `ref/range` tốt; `ALL` xấu (full scan)  
- `rows`: ước lượng số row đọc  
- `filtered`: % row qua filter  
- `Extra`: "Using index" (covering), "Using filesort" (cần index theo ORDER BY)

**Fix "Using filesort"**: tạo index `(author_id, created_at DESC)`.


#### Design Pattern — Repository
#### Design Pattern — Repository (Search & Filter)
```php
interface PostRepository {
  public function paginate(array $filters, int $perPage=20): \Illuminate\Contracts\Pagination\LengthAwarePaginator;
}

class EloquentPostRepository implements PostRepository {
  public function paginate(array $filters, int $perPage=20){
    return Post::query()
      ->when($filters['q']??null, fn($q,$v)=>$q->whereFullText('title', $v))
      ->when($filters['author_id']??null, fn($q,$v)=>$q->where('author_id',$v))
      ->when($filters['since']??null, fn($q,$v)=>$q->where('created_at','>=',$v))
      ->orderByDesc('created_at')
      ->paginate($perPage);
  }
}

// Controller
class PostController extends Controller {
  public function __construct(private PostRepository $repo){}
  public function index(Request $r){ return PostResource::collection($this->repo->paginate($r->all())); }
}
```
**When**: cần cô lập ORM, dễ test; **Pitfall**: abstraction dư thừa nếu CRUD đơn giản.


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

#### RHF + Zod
```tsx
import { useForm } from 'react-hook-form';
import { z } from 'zod'; import { zodResolver } from '@hookform/resolvers/zod';
const schema = z.object({ email:z.string().email(), password:z.string().min(8) });
type Form = z.infer<typeof schema>;
export default function Login(){
  const { register, handleSubmit, formState:{errors} } = useForm<Form>({ resolver: zodResolver(schema) });
  const onSubmit = (d:Form)=> console.log(d);
  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-3 max-w-sm">
      <input {...register('email')} className="border p-2 w-full" placeholder="Email"/>
      <p className="text-sm text-red-600">{errors.email?.message}</p>
      <input type="password" {...register('password')} className="border p-2 w-full" placeholder="Password"/>
      <p className="text-sm text-red-600">{errors.password?.message}</p>
      <button className="border rounded px-4 py-2">Login</button>
    </form>
  );
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
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 15 with emphasis on reliability and performance.*
- **Tech Qs**:  
  1) *How do you ensure idempotency for mutations?*  
  2) *Explain a query optimization you shipped and its impact.*


### Interview Focus
- Patterns: Repository
- MySQL: Idempotency Key
- Pitfalls:
- Avoid SELECT *; keep covering index effective.
- Set consistent ordering for pagination.
- Add jitter to retries to avoid herd effect.
- Keep transactions short; avoid long-held locks.

**Case Study D15**: Implement idempotent create-order with DB upsert + Redis lock; return 201 on first call, 200 cached for retries.

### Practice Tasks
- Build the feature end-to-end; write why each design choice fits interview expectations.

### Acceptance Criteria
- BE: đúng schema/status; test Feature; log structured.
- FE: list/mutation OK; handle loading/error/empty; mutation rollback tốt.
- DB: có EXPLAIN + index/partition đề xuất.


### Docs & References
- [AWS S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html)
- [Database](https://dev.mysql.com/doc/)
- [React](https://react.dev/learn)
- [React Query](https://tanstack.com/query/latest)

### Checklist
- [ ] Code BE+FE end-to-end
- [ ] 1 Feature test (BE) + 1 RTL/Playwright (FE)
- [ ] Ghi chú EXPLAIN & tối ưu
- [ ] 2 câu trả lời kỹ thuật (EN)


### Reflection
- What trade-off today would you defend in an interview? Why?