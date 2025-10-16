# Day 03 — Interview Prep Day 3

_Generated on 2025-10-01 03:52._

### Plan Notes
- **EN (CSV)**: MySQL Indexing & EXPLAIN

## Backend (BE)

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

#### Auth+Sanctum
```php
$token = $user->createToken('web')->plainTextToken;
Route::middleware('auth:sanctum')->get('/profile', fn(Request $r)=>$r->user());
```

#### Indexing+EXPLAIN
```sql
CREATE INDEX idx_posts_author_created ON posts(author_id, created_at DESC);
EXPLAIN SELECT id,title,created_at FROM posts WHERE author_id=? ORDER BY created_at DESC LIMIT 20;
```

#### Design Pattern — Repository
Abstract persistence behind an interface to decouple domain from ORM.

#### MySQL — Partition by Date
Range partition on date to prune scans; ensure partition columns in PK/unique.

## Frontend (FE)

#### InfiniteQuery
```tsx
import { useInfiniteQuery } from '@tanstack/react-query';
const fetchPage = async ({pageParam=1})=> (await fetch(`/api/posts?page=${pageParam}`)).json();
const { data, fetchNextPage, hasNextPage } = useInfiniteQuery({ queryKey:['posts'], queryFn:fetchPage, getNextPageParam:(last)=> last.next_page ?? false });
```

#### A11y Modal
```tsx
// focus trap + ESC to close
```

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

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 3 with an emphasis on reliability and performance.*
- **2 Tech Q&A to rehearse**
  1) *Explain your approach to idempotent APIs and retries.*
  2) *How do you design pagination and prevent N+1 issues?*


### Interview Focus
- Pattern: **Repository** — why/when & trade-offs.
- MySQL: **Partition by Date** — demonstrate with EXPLAIN or schema.
- BE deep-dive: API+Validation, Auth+Sanctum, Indexing+EXPLAIN.
- FE deep-dive: InfiniteQuery, A11y Modal, RQ Mutation Optimistic.
- **Common pitfalls**:
- Lock TTL too short causes duplicate work.
- Pagination without deterministic order causes duplicates.
- Retry without jitter can thundering-herd.
- SELECT * prevents covering index.


### Practice Tasks
- Implement: **Interview Prep Day 3** feature end-to-end (BE & FE), commit with README explaining design choices.
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