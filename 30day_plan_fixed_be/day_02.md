# Day 02 — Interview Prep Day 2

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

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

#### Design Pattern — Strategy
```php
interface PricingStrategy { public function price(float $base, array $ctx=[]): float; }
class DefaultPricing implements PricingStrategy { public function price(float $b, array $ctx=[]): float { return $b; } }
class PremiumPricing implements PricingStrategy { public function price(float $b, array $ctx=[]): float { return round($b*0.95,2); } }
class CheckoutService {
  public function __construct(private PricingStrategy $strategy){}
  public function total(float $base, array $ctx=[]): float { return $this->strategy->price($base,$ctx); }
}
// Binding
$this->app->bind(PricingStrategy::class, fn()=> new PremiumPricing());
$total = (new CheckoutService(app(PricingStrategy::class)))->total(100.0);
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

#### useEffect
```tsx
useEffect(()=>{ document.title = `Count ${count}`; return () => {/* cleanup */}; }, [count]);
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 2 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: backpressure, consistency, debounce, isolation level, observability, rate limiting, rollback, throughput
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
- [Eloquent](https://laravel.com/docs/eloquent)
- [Laravel](https://laravel.com/docs)
- [React](https://react.dev/learn)

### Reflection
- What trade-off today will you highlight in an interview?