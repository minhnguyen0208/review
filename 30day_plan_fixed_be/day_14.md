# Day 14 — Interview Prep Day 14

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Error Handling & Exceptions
```php
// app/Exceptions/Handler.php -> render(): normalize 4xx/5xx JSON
```

#### Rate Limiting
```php
RateLimiter::for('api', fn($r)=> Limit::perMinute(120)->by($r->ip()));
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

#### MySQL — Batch Upsert
```php
$chunks = array_chunk($rows, 1000);
foreach($chunks as $chunk){
  Model::upsert($chunk, ['unique_key'], ['name','status','updated_at']);
}
```
**Tip**: wrap in transaction if business requires atomicity; avoid massive single statements.


## Frontend (FE)

#### useState
```tsx
const [count,setCount] = useState(0);
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 14 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: consistency, denormalization, idempotency, observability, rate limiting, retries, rollback, throughput
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