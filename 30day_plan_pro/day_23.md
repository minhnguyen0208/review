# Day 23 — Interview Prep Day 23

_Generated on 2025-10-01 04:00._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### S3 Multipart
```php
// 1) init multipart -> 2) generate signed part URLs -> 3) complete
```

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

#### React Query — List
```tsx
import { useQuery } from '@tanstack/react-query';
async function fetchPosts(){ const r=await fetch('/api/posts'); if(!r.ok) throw new Error('Network'); return r.json(); }
export function Posts(){
  const { data, isLoading, error } = useQuery({ queryKey:['posts'], queryFn: fetchPosts, staleTime: 30_000 });
  if(isLoading) return <p>Loading…</p>;
  if(error) return <p>Failed to load.</p>;
  return <ul>{data.data.map((p:any)=> <li key={p.id}>{p.title}</li>)}</ul>;
}
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 23 with emphasis on reliability and performance.*
- **Tech Qs**:  
  1) *How do you ensure idempotency for mutations?*  
  2) *Explain a query optimization you shipped and its impact.*


### Interview Focus
- Patterns: CQRS
- MySQL: Partition by Date
- Pitfalls:
- Avoid SELECT *; keep covering index effective.
- Set consistent ordering for pagination.
- Add jitter to retries to avoid herd effect.
- Keep transactions short; avoid long-held locks.

**Case Study D23**: Implement idempotent create-order with DB upsert + Redis lock; return 201 on first call, 200 cached for retries.

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
- [React Query](https://tanstack.com/query/latest)

### Checklist
- [ ] Code BE+FE end-to-end
- [ ] 1 Feature test (BE) + 1 RTL/Playwright (FE)
- [ ] Ghi chú EXPLAIN & tối ưu
- [ ] 2 câu trả lời kỹ thuật (EN)


### Reflection
- What trade-off today would you defend in an interview? Why?