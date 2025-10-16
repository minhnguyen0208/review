# Day 27 — Interview Prep Day 27

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Query Builder vs Eloquent
```php
DB::table('posts')->where('published',1)->count(); // faster aggregate
```

#### API Versioning
```php
Route::prefix('v1')->group(fn()=> Route::apiResource('posts', V1\PostController::class));
```

#### Cache+Redis Lock
```php
// Cache 15m
$stats = Cache::remember('stats:daily', now()->addMinutes(15), fn()=>computeStats());

// Idempotent critical section with lock
$result = Cache::lock('refresh_token_'.$companyId, 60)->block(10, function () use ($refreshToken) {
  return Http::asForm()->retry(2, 250)->post(config('sso.token_url'), [
    'grant_type'=>'refresh_token','refresh_token'=>$refreshToken
  ])->json();
});
```

#### Design Pattern — Repository
```php
interface PostRepository { public function paginate(array $filters,int $perPage=20): \Illuminate\Contracts\Pagination\LengthAwarePaginator; }
class EloquentPostRepository implements PostRepository {
  public function paginate(array $filters, int $perPage=20){
    return Post::query()
      ->when($filters['q']??null, fn($q,$v)=>$q->whereFullText('title',$v))
      ->when($filters['author_id']??null, fn($q,$v)=>$q->where('author_id',$v))
      ->orderByDesc('created_at')->paginate($perPage);
  }
}
class PostController extends Controller {
  public function __construct(private PostRepository $repo){}
  public function index(Request $r){ return PostResource::collection($this->repo->paginate($r->all())); }
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

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 27 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: backpressure, consistency, debounce, denormalization, observability, partitioning, rate limiting, rollback
- **Q&A**: 1) *How do you ensure idempotency for create endpoints?*  2) *When do you choose client caching vs server caching?*
- **Behavioral (STAR)**: *Tell me about a production incident you owned end‑to‑end.*


### Practice Tasks
- Build feature end-to-end; capture EXPLAIN & tests.
- Record 60s answer on a topic from today.

### Acceptance Criteria
- BE schema/status correct + tests; FE handles loading/error/empty; DB EXPLAIN + index note.

### Docs & References
- [Cache](https://laravel.com/docs/cache)
- [Database](https://dev.mysql.com/doc/)
- [Laravel](https://laravel.com/docs)
- [React](https://react.dev/learn)

### Reflection
- What trade-off today will you highlight in an interview?