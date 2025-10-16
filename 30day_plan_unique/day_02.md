# Day 02 — Interview Prep Day 2

_Generated on 2025-10-01 03:52._

### Plan Notes
- **EN (CSV)**: Pagination & Streaming

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

#### Design Pattern — Strategy
Swap algorithms at runtime via interface.
Pros: open/closed. Cons: more classes.

#### MySQL — EXPLAIN
Check type, rows, filtered, Extra; avoid ALL & filesort via proper index.

## Frontend (FE)

#### RQ List
```tsx
import { useQuery } from '@tanstack/react-query';
async function fetchPosts(){ const r=await fetch('/api/posts'); if(!r.ok) throw new Error(); return r.json(); }
export function Posts(){ const {data,isLoading,error}=useQuery({queryKey:['posts'],queryFn:fetchPosts}); /* render */ }
```

#### InfiniteQuery
```tsx
import { useInfiniteQuery } from '@tanstack/react-query';
const fetchPage = async ({pageParam=1})=> (await fetch(`/api/posts?page=${pageParam}`)).json();
const { data, fetchNextPage, hasNextPage } = useInfiniteQuery({ queryKey:['posts'], queryFn:fetchPage, getNextPageParam:(last)=> last.next_page ?? false });
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 2 with an emphasis on reliability and performance.*
- **2 Tech Q&A to rehearse**
  1) *Explain your approach to idempotent APIs and retries.*
  2) *How do you design pagination and prevent N+1 issues?*


### Interview Focus
- Pattern: **Strategy** — why/when & trade-offs.
- MySQL: **EXPLAIN** — demonstrate with EXPLAIN or schema.
- BE deep-dive: Testing(Feature), API+Validation.
- FE deep-dive: RQ List, InfiniteQuery.
- **Common pitfalls**:
- Missing error boundaries hides failures.
- Transaction scope too wide increases deadlock risk.
- Optimistic update without rollback corrupts UI state.
- Pagination without deterministic order causes duplicates.


### Practice Tasks
- Implement: **Interview Prep Day 2** feature end-to-end (BE & FE), commit with README explaining design choices.
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