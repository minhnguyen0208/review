# Day 05 — Interview Prep Day 5

_Generated on 2025-10-01 04:00._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Transactions+Outbox
```php
DB::transaction(function(){
  $order = Order::create([...]);
  Outbox::create(['event'=>'OrderCreated','payload'=>json_encode(['id'=>$order->id])]);
});
// worker drains Outbox -> Kafka/SNS; consumer is idempotent.
```

#### Auth+Sanctum
```php
$token = $user->createToken('web')->plainTextToken;
Route::middleware('auth:sanctum')->get('/me', fn(Request $r)=> $r->user());
```

#### Design Pattern — CQRS
#### Architectural Pattern — CQRS (Laravel-style)
```php
// Command (Write)
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

// Projector (Read model denormalized)
class OrderProjector {
  public function onOrderCreated(OrderCreated $e){
    ReadOrder::updateOrCreate(['id'=>$e->id], ['total'=>$e->total, 'status'=>'created']);
  }
}
```
**Trade-offs**: scale đọc tốt; **Cons**: phức tạp, eventual consistency.


#### MySQL — Idempotency Key
#### MySQL — Idempotent API (table + usage)
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
**Interview**: giải thích thứ tự thao tác để tránh race (unique + upsert).


## Frontend (FE)

#### A11y Focus Trap Modal
```tsx
// focus first tabbable, close on ESC; restore focus on unmount
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 5 with emphasis on reliability and performance.*
- **Tech Qs**:  
  1) *How do you ensure idempotency for mutations?*  
  2) *Explain a query optimization you shipped and its impact.*


### Interview Focus
- Patterns: CQRS
- MySQL: Idempotency Key
- Pitfalls:
- Avoid SELECT *; keep covering index effective.
- Set consistent ordering for pagination.
- Add jitter to retries to avoid herd effect.
- Keep transactions short; avoid long-held locks.

**Case Study D5**: Implement idempotent create-order with DB upsert + Redis lock; return 201 on first call, 200 cached for retries.

### Practice Tasks
- Build the feature end-to-end; write why each design choice fits interview expectations.

### Acceptance Criteria
- BE: đúng schema/status; test Feature; log structured.
- FE: list/mutation OK; handle loading/error/empty; mutation rollback tốt.
- DB: có EXPLAIN + index/partition đề xuất.


### Docs & References
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