# Day 15 — Interview Prep Day 15

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Rate Limiting
```php
RateLimiter::for('api', fn($r)=> Limit::perMinute(120)->by($r->ip()));
```

#### Custom Validation Rules
```php
Validator::extend('safe_slug', fn($attr,$val)=> preg_match('/^[a-z0-9-]+$/',$val));
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

#### useEffect
```tsx
useEffect(()=>{ document.title = `Count ${count}`; return () => {/* cleanup */}; }, [count]);
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 15 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: debounce, denormalization, idempotency, observability, partitioning, rate limiting, rollback, throughput
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