# Day 29 — Interview Prep Day 29

_Generated on 2025-10-01 03:52._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Testing(Feature)
```php
public function test_user_can_login(){
  $u=User::factory()->create(['password'=>bcrypt('secret')]);
  $this->post('/login',['email'=>$u->email,'password'=>'secret'])->assertRedirect('/dashboard');
}
```

#### API+Validation
```php
// routes/api.php
Route::apiResource('posts', PostController::class);

// App/Http/Controllers/PostController.php
class PostController extends Controller {
  public function index(Request $r){
    $q = Post::query()
      ->when($r->q, fn($q,$v)=>$q->where('title','like',"%$v%"))
      ->when($r->author_id, fn($q,$v)=>$q->where('author_id',$v))
      ->orderByDesc('created_at');
    return PostResource::collection($q->paginate($r->integer('per_page',20)));
  }
  public function store(StorePostRequest $req){
    $post = Post::create($req->validated());
    return (new PostResource($post))->response()->setStatusCode(201);
  }
}
```

#### Design Pattern — CQRS
Separate write (normalized) and read (denormalized) models.

#### MySQL — Batch Upsert
Use upsert/ON DUP KEY UPDATE; chunk batches; wrap in txn when needed.

## Frontend (FE)

#### RQ Mutation Optimistic
```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';
function useUpdatePost(){
  const qc = useQueryClient();
  return useMutation({
    mutationFn: (p:any)=> fetch('/api/posts/'+p.id,{method:'PUT',headers:{'Content-Type':'application/json'},body:JSON.stringify(p)}).then(r=>r.json()),
    onMutate: async (p)=>{ await qc.cancelQueries({queryKey:['posts']}); const prev=qc.getQueryData<any[]>(['posts']);
      qc.setQueryData<any[]>(['posts'], old=> old?.map(x=>x.id===p.id?{...x,...p}:x) ?? []); return {prev}; },
    onError: (_e,_p,ctx)=> ctx?.prev && qc.setQueryData(['posts'], ctx.prev),
    onSettled: ()=> qc.invalidateQueries({queryKey:['posts']})
  });
}
```

#### RQ List
```tsx
import { useQuery } from '@tanstack/react-query';
async function fetchPosts(){ const r=await fetch('/api/posts'); if(!r.ok) throw new Error(); return r.json(); }
export function Posts(){ const {data,isLoading,error}=useQuery({queryKey:['posts'],queryFn:fetchPosts}); /* render */ }
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 29 with an emphasis on reliability and performance.*
- **2 Tech Q&A to rehearse**
  1) *Explain your approach to idempotent APIs and retries.*
  2) *How do you design pagination and prevent N+1 issues?*


### Interview Focus
- Pattern: **CQRS** — why/when & trade-offs.
- MySQL: **Batch Upsert** — demonstrate with EXPLAIN or schema.
- BE deep-dive: Testing(Feature), API+Validation.
- FE deep-dive: RQ Mutation Optimistic, RQ List.
- **Common pitfalls**:
- Optimistic update without rollback corrupts UI state.
- SELECT * prevents covering index.
- Not normalizing error schema complicates FE.
- Eager loading without constraints can explode memory.


### Practice Tasks
- Implement: **Interview Prep Day 29** feature end-to-end (BE & FE), commit with README explaining design choices.
- Add EXPLAIN notes + index/partition proposal if needed.
- Write 1 Feature test (BE) + 1 RTL or Playwright test (FE).


### Acceptance Criteria
- BE endpoints return correct schema & status (201/204/404/422) with tests.
- FE states: loading/error/empty handled; mutation has optimistic update + rollback.
- Query hot-path has EXPLAIN screenshot/notes + index plan.


### Docs & References
- [Cache](https://laravel.com/docs/cache)
- [Database](https://dev.mysql.com/doc/)
- [Docker](https://docs.docker.com/)
- [Eloquent](https://laravel.com/docs/eloquent)
- [Jest](https://jestjs.io/docs/getting-started)
- [Laravel](https://laravel.com/docs)
- [OWASP](https://owasp.org/www-project-top-ten/)
- [Playwright](https://playwright.dev/docs/intro)
- [Queues](https://laravel.com/docs/queues)
- [React](https://react.dev/learn)
- [React Query](https://tanstack.com/query/latest)
- [React Router](https://reactrouter.com/en/main)
- [Sanctum](https://laravel.com/docs/sanctum)
- [Tailwind](https://tailwindcss.com/docs)
- [TypeScript](https://www.typescriptlang.org/docs/)
- [Validation](https://laravel.com/docs/validation)

### Checklist
- [ ] BE implemented & tested (Feature/Pest)
- [ ] FE implemented & tested (RTL/Playwright)
- [ ] Performance notes (EXPLAIN/metrics) committed
- [ ] 2 interview answers recorded (EN)


### Reflection
- What would you change if you had to scale this 10x?
- Which trade-off was most impactful today?