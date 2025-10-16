# Day 29 — Interview Prep Day 29

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Config Caching
```bash
php artisan config:cache && php artisan route:cache
```

#### S3 Multipart
```php
// 1) init multipart; 2) generate signed part URLs; 3) complete upload
```

#### Transactions+Outbox
```php
DB::transaction(function(){
  $order = Order::create([...]);
  Outbox::create(['event'=>'OrderCreated','payload'=>json_encode(['id'=>$order->id])]);
});
// Outbox worker -> broker; consumers must be idempotent.
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

#### MySQL — Batch Upsert
```php
$chunks = array_chunk($rows, 1000);
foreach($chunks as $chunk){
  Model::upsert($chunk, ['unique_key'], ['name','status','updated_at']);
}
```
**Tip**: wrap in transaction if business requires atomicity; avoid massive single statements.


## Frontend (FE)

#### useMemo
```tsx
const expensive = useMemo(()=> heavy(data), [data]); // memo only if heavy & stable
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 29 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: backpressure, debounce, idempotency, memoization, observability, optimistic UI, partitioning, throughput
- **Q&A**: 1) *How do you ensure idempotency for create endpoints?*  2) *When do you choose client caching vs server caching?*
- **Behavioral (STAR)**: *Tell me about a production incident you owned end‑to‑end.*


### Practice Tasks
- Build feature end-to-end; capture EXPLAIN & tests.
- Record 60s answer on a topic from today.

### Acceptance Criteria
- BE schema/status correct + tests; FE handles loading/error/empty; DB EXPLAIN + index note.

### Docs & References
- [AWS S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html)
- [Database](https://dev.mysql.com/doc/)
- [Laravel](https://laravel.com/docs)
- [React](https://react.dev/learn)

### Reflection
- What trade-off today will you highlight in an interview?