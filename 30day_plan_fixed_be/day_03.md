# Day 03 — Interview Prep Day 3

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

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

#### Queues+Retry+DLQ
```php
class SyncJob implements ShouldQueue {
  use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
  public $tries=3; public $backoff=[60,300,900];
  public function handle(){ Partner::syncOrFail(); }
}
// Horizon for throughput/failures; DLQ for analysis.
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

#### MySQL — Partition by Date
```sql
ALTER TABLE logs
PARTITION BY RANGE (TO_DAYS(created_at))(
  PARTITION p2025 VALUES LESS THAN (TO_DAYS('2026-01-01')),
  PARTITION pmax  VALUES LESS THAN MAXVALUE
);
```
**Gotcha**: partitioning column must appear in every UNIQUE/PRIMARY key (MySQL 8.0).


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

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 3 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: backpressure, consistency, denormalization, isolation level, observability, partitioning, rate limiting, rollback
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
- [Queues](https://laravel.com/docs/queues)
- [React](https://react.dev/learn)

### Reflection
- What trade-off today will you highlight in an interview?