# Day 14 — Interview Prep Day 14

_Generated on 2025-10-01 04:00._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Auth+Sanctum
```php
$token = $user->createToken('web')->plainTextToken;
Route::middleware('auth:sanctum')->get('/me', fn(Request $r)=> $r->user());
```

#### S3 Multipart
```php
// 1) init multipart -> 2) generate signed part URLs -> 3) complete
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


#### MySQL — Batch Upsert
#### MySQL — Batch Upsert (Laravel)
```php
// chunked upsert
$chunks = array_chunk($rows, 1000);
foreach($chunks as $chunk){
  Model::upsert($chunk, ['unique_key'], ['name','status','updated_at']);
}
```
**Tip**: gộp cột cần update; bọc transaction nếu business cần tính nguyên vẹn.


## Frontend (FE)

#### Error Boundary
```tsx
class ErrorBoundary extends React.Component<{children:React.ReactNode},{hasError:boolean}>{
  state={hasError:false}; componentDidCatch(err:any){ this.setState({hasError:true}); }
  render(){ return this.state.hasError? <p>Something went wrong.</p> : this.props.children; }
}
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 14 with emphasis on reliability and performance.*
- **Tech Qs**:  
  1) *How do you ensure idempotency for mutations?*  
  2) *Explain a query optimization you shipped and its impact.*


### Interview Focus
- Patterns: Strategy
- MySQL: Batch Upsert
- Pitfalls:
- Avoid SELECT *; keep covering index effective.
- Set consistent ordering for pagination.
- Add jitter to retries to avoid herd effect.
- Keep transactions short; avoid long-held locks.

**Case Study D14**: Implement idempotent create-order with DB upsert + Redis lock; return 201 on first call, 200 cached for retries.

### Practice Tasks
- Build the feature end-to-end; write why each design choice fits interview expectations.

### Acceptance Criteria
- BE: đúng schema/status; test Feature; log structured.
- FE: list/mutation OK; handle loading/error/empty; mutation rollback tốt.
- DB: có EXPLAIN + index/partition đề xuất.


### Docs & References
- [AWS S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html)
- [Database](https://dev.mysql.com/doc/)
- [Laravel](https://laravel.com/docs)
- [Validation](https://laravel.com/docs/validation)

### Checklist
- [ ] Code BE+FE end-to-end
- [ ] 1 Feature test (BE) + 1 RTL/Playwright (FE)
- [ ] Ghi chú EXPLAIN & tối ưu
- [ ] 2 câu trả lời kỹ thuật (EN)


### Reflection
- What trade-off today would you defend in an interview? Why?