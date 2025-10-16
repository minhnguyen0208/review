# Day 17 — Interview Prep Day 17

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Notifications
```php
$user->notify(new ResetPasswordNotification($token));
```

#### Broadcasting
```php
broadcast(new OrderCreated($order))->toOthers();
```

#### Feature Flags
```php
if(Feature::enabled('new-header')){ /* render new */ } else { /* old */ }
```

#### Design Pattern — CQRS
```php
// Command & Handler
class CreateOrderCommand { public function __construct(public array $data){} }
class CreateOrderHandler {
  public function handle(CreateOrderCommand $cmd){
    return DB::transaction(function() use($cmd){
      $order = Order::create($cmd->data);
      event(new OrderCreated($order->id, $order->total));
      return $order;
    });
  }
}
// Projector (read model)
class OrderProjector {
  public function onOrderCreated(OrderCreated $e){
    ReadOrder::updateOrCreate(['id'=>$e->id], ['total'=>$e->total,'status'=>'created']);
  }
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

#### useCallback
```tsx
const onClick = useCallback(()=> doThing(id), [id]); // stable ref for deps
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 17 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: backpressure, debounce, denormalization, isolation level, latency, observability, rate limiting, throughput
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