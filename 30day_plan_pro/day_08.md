# Day 08 — Interview Prep Day 8

_Generated on 2025-10-01 04:00._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

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


#### API+Validation+Resource
```php
// routes/api.php
Route::apiResource('posts', PostController::class);

// Controller
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

// Request
class StorePostRequest extends FormRequest {
  public function rules(): array {
    return ['title'=>'required|string|max:255','body'=>'required|string','author_id'=>'required|integer|exists:users,id'];
  }
}
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


#### MySQL — Partition by Date
#### MySQL — Partition theo ngày
```sql
ALTER TABLE logs
PARTITION BY RANGE (TO_DAYS(created_at))(
  PARTITION p2025 VALUES LESS THAN (TO_DAYS('2026-01-01')),
  PARTITION pmax  VALUES LESS THAN MAXVALUE
);
```
**Lưu ý**: cột partition cần nằm trong PK/unique (MySQL 8.0 constraint).


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

#### Suspense + Code Splitting
```tsx
import { lazy, Suspense } from 'react';
const Reports = lazy(()=> import('./Reports'));
export default function App(){ return <Suspense fallback={<div>Loading…</div>}><Reports/></Suspense>; }
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 8 with emphasis on reliability and performance.*
- **Tech Qs**:  
  1) *How do you ensure idempotency for mutations?*  
  2) *Explain a query optimization you shipped and its impact.*


### Interview Focus
- Patterns: Strategy
- MySQL: Partition by Date
- Pitfalls:
- Avoid SELECT *; keep covering index effective.
- Set consistent ordering for pagination.
- Add jitter to retries to avoid herd effect.
- Keep transactions short; avoid long-held locks.

**Case Study D8**: Implement idempotent create-order with DB upsert + Redis lock; return 201 on first call, 200 cached for retries.

### Practice Tasks
- Build the feature end-to-end; write why each design choice fits interview expectations.

### Acceptance Criteria
- BE: đúng schema/status; test Feature; log structured.
- FE: list/mutation OK; handle loading/error/empty; mutation rollback tốt.
- DB: có EXPLAIN + index/partition đề xuất.


### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Laravel](https://laravel.com/docs)
- [React](https://react.dev/learn)
- [Validation](https://laravel.com/docs/validation)

### Checklist
- [ ] Code BE+FE end-to-end
- [ ] 1 Feature test (BE) + 1 RTL/Playwright (FE)
- [ ] Ghi chú EXPLAIN & tối ưu
- [ ] 2 câu trả lời kỹ thuật (EN)


### Reflection
- What trade-off today would you defend in an interview? Why?