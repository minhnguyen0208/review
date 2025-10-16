# Day 01 — Interview Prep Day 1

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### API+Validation+Resource
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

#### Eloquent Performance
```php
// Eager loading with constraints to avoid N+1 + memory blowup
$posts = Post::with(['comments'=>fn($q)=>$q->latest()->limit(5)])->paginate(20);

// Subquery select (latest comment ts)
$posts = Post::addSelect(['latest_comment_at'=>Comment::select('created_at')
  ->whereColumn('comments.post_id','posts.id')->latest()->limit(1)])->get();

// Counters + sort by counts
$popular = Post::withCount(['comments','likes'])->orderByDesc('comments_count')->limit(50)->get();

// Scope + fulltext
class Post extends Model { function scopeOfAuthor($q,$a){return $q->where('author_id',$a);} }
$rows = Post::ofAuthor($authorId)->when($kw, fn($x)=>$x->whereFullText('title',$kw))->paginate();
```

#### Design Pattern — Factory
```php
interface Notifier { public function send(string $to, string $msg): void; }
class EmailNotifier implements Notifier { public function send(string $to, string $msg){ Mail::to($to)->send(new GenericMail($msg)); } }
class SmsNotifier implements Notifier { public function send(string $to, string $msg){ app('twilio')->messages->create($to, ['from'=>config('twilio.from'),'body'=>$msg]); } }
class NotifierFactory {
  public static function make(string $channel): Notifier {
    return match($channel){ 'sms'=>new SmsNotifier(), 'email'=>new EmailNotifier(), default=>throw new InvalidArgumentException('Unsupported channel') };
  }
}
// Usage
$notifier = NotifierFactory::make(config('notify.channel','email'));
$notifier->send($user->email, 'Welcome!');
```

#### MySQL — Covering Index
```sql
-- Bad: SELECT *
SELECT * FROM posts WHERE author_id = ? ORDER BY created_at DESC LIMIT 20;

-- Good: only needed cols + matching composite index
CREATE INDEX idx_posts_author_created ON posts(author_id, created_at DESC);
SELECT id, title, created_at FROM posts WHERE author_id=? ORDER BY created_at DESC LIMIT 20;
```
**Reasoning**: "Using index" in EXPLAIN => avoid back-to-table lookup → lower IO/latency.


## Frontend (FE)

#### useState
```tsx
const [count,setCount] = useState(0);
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 1 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: backpressure, denormalization, isolation level, observability, partitioning, rate limiting, retries, rollback
- **Q&A**: 1) *How do you ensure idempotency for create endpoints?*  2) *When do you choose client caching vs server caching?*
- **Behavioral (STAR)**: *Tell me about a production incident you owned end‑to‑end.*


### Practice Tasks
- Build feature end-to-end; capture EXPLAIN & tests.
- Record 60s answer on a topic from today.

### Acceptance Criteria
- BE schema/status correct + tests; FE handles loading/error/empty; DB EXPLAIN + index note.

### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Eloquent](https://laravel.com/docs/eloquent)
- [Laravel](https://laravel.com/docs)
- [React](https://react.dev/learn)
- [Validation](https://laravel.com/docs/validation)

### Reflection
- What trade-off today will you highlight in an interview?