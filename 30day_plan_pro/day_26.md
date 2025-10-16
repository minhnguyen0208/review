# Day 26 — Interview Prep Day 26

_Generated on 2025-10-01 04:00._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Eloquent Performance
```php
// Eager loading with constraints
$posts = Post::with(['comments'=>fn($q)=>$q->latest()->limit(5)])->paginate(20);

// Subquery: latest comment timestamp
$posts = Post::addSelect(['latest_comment_at'=>Comment::select('created_at')
  ->whereColumn('comments.post_id','posts.id')->latest()->limit(1)])->get();

// Aggregate counters
$posts = Post::withCount(['comments','likes'])->orderByDesc('comments_count')->limit(50)->get();

// Scope + when + fulltext
class Post extends Model { function scopeOfAuthor($q,$a){return $q->where('author_id',$a);} }
$rows = Post::ofAuthor($authorId)->when($q, fn($x)=>$x->whereFullText('title',$q))->paginate();
```

#### Cache+Redis Lock
```php
// Cache computed stats 15m
$stats = Cache::remember('stats:daily', now()->addMinutes(15), fn()=>computeStats());

// Lock critical section (idempotent refresh)
$result = Cache::lock('refresh_token_'.$companyId, 60)->block(10, function () use ($refreshToken) {
  return Http::asForm()->retry(2, 250)->post(config('sso.token_url'), ['grant_type'=>'refresh_token','refresh_token'=>$refreshToken])->json();
});
```

#### Design Pattern — Strategy
#### Design Pattern — Strategy (Pricing)
```php
interface PricingStrategy { public function price(float $base, array $ctx=[]): float; }
class DefaultPricing implements PricingStrategy { public function price(float $b, array $ctx=[]): float { return $b; } }
class PremiumPricing implements PricingStrategy { public function price(float $b, array $ctx=[]): float { return round($b * 0.95, 2); } }

class CheckoutService {
  public function __construct(private PricingStrategy $strategy){}
  public function total(float $base, array $ctx=[]): float { return $this->strategy->price($base, $ctx); }
}

// Service Provider binding (AppServiceProvider)
$this->app->bind(PricingStrategy::class, fn()=> new PremiumPricing());

// Usage
$total = (new CheckoutService(app(PricingStrategy::class)))->total(100.0);
```
**Interview tip**: So sánh Strategy vs if/else; nói về **testability** & **open/closed**.


#### MySQL — Covering Index
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


## Frontend (FE)

#### A11y Focus Trap Modal
```tsx
// focus first tabbable, close on ESC; restore focus on unmount
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 26 with emphasis on reliability and performance.*
- **Tech Qs**:  
  1) *How do you ensure idempotency for mutations?*  
  2) *Explain a query optimization you shipped and its impact.*


### Interview Focus
- Patterns: Strategy
- MySQL: Covering Index
- Pitfalls:
- Avoid SELECT *; keep covering index effective.
- Set consistent ordering for pagination.
- Add jitter to retries to avoid herd effect.
- Keep transactions short; avoid long-held locks.

**Case Study D26**: Implement idempotent create-order with DB upsert + Redis lock; return 201 on first call, 200 cached for retries.

### Practice Tasks
- Build the feature end-to-end; write why each design choice fits interview expectations.

### Acceptance Criteria
- BE: đúng schema/status; test Feature; log structured.
- FE: list/mutation OK; handle loading/error/empty; mutation rollback tốt.
- DB: có EXPLAIN + index/partition đề xuất.


### Docs & References
- [Cache](https://laravel.com/docs/cache)
- [Database](https://dev.mysql.com/doc/)
- [Eloquent](https://laravel.com/docs/eloquent)

### Checklist
- [ ] Code BE+FE end-to-end
- [ ] 1 Feature test (BE) + 1 RTL/Playwright (FE)
- [ ] Ghi chú EXPLAIN & tối ưu
- [ ] 2 câu trả lời kỹ thuật (EN)


### Reflection
- What trade-off today would you defend in an interview? Why?