# Day 20 — Interview Prep Day 20

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Horizon
```txt
- Monitor queue throughput, failures, retries; tune backoff/maxTries
```

#### Migrations & Seeders
```php
Schema::table('posts', function(Blueprint $t){ $t->softDeletes(); });
Post::factory()->count(50)->create();
```

#### Soft Deletes
```php
use SoftDeletes; Post::withTrashed()->find($id); Post::onlyTrashed()->restore();
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

#### Custom Hook
```tsx
function useDebounce<T>(value:T, delay=300){ const [v,setV] = useState(value); useEffect(()=>{ const id=setTimeout(()=>setV(value),delay); return ()=>clearTimeout(id);},[value,delay]); return v; }
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 20 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: backpressure, denormalization, idempotency, latency, observability, optimistic UI, partitioning, rate limiting
- **Q&A**: 1) *How do you ensure idempotency for create endpoints?*  2) *When do you choose client caching vs server caching?*
- **Behavioral (STAR)**: *Tell me about a production incident you owned end‑to‑end.*


### Practice Tasks
- Build feature end-to-end; capture EXPLAIN & tests.
- Record 60s answer on a topic from today.

### Acceptance Criteria
- BE schema/status correct + tests; FE handles loading/error/empty; DB EXPLAIN + index note.

### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Laravel](https://laravel.com/docs)
- [React](https://react.dev/learn)

### Reflection
- What trade-off today will you highlight in an interview?