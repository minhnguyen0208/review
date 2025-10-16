# Day 12 — Interview Prep Day 12

_Generated on 2025-10-01 04:00._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Queues+Retry+DLQ
```php
class SyncJob implements ShouldQueue {
  use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
  public $tries=3; public $backoff=[60,300,900];
  public function handle(){ Partner::syncOrFail(); }
}
// Horizon dashboard to inspect failures; setup dead-letter queue for analysis.
```

#### Transactions+Outbox
```php
DB::transaction(function(){
  $order = Order::create([...]);
  Outbox::create(['event'=>'OrderCreated','payload'=>json_encode(['id'=>$order->id])]);
});
// worker drains Outbox -> Kafka/SNS; consumer is idempotent.
```

#### Design Pattern — Event Sourcing
#### Architectural Pattern — Event Sourcing (Concept + Snapshot)
```txt
- Persist events: OrderCreated, ItemAdded, Paid, Shipped...
- Rehydrate aggregate by replaying events; snapshot mỗi N events để nhanh hơn.
- Projections để query nhanh.
```
**Interview angle**: audit trail tự nhiên, rollback theo thời gian; nhưng **dev cost cao**.


#### MySQL — EXPLAIN
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


## Frontend (FE)

#### A11y Focus Trap Modal
```tsx
// focus first tabbable, close on ESC; restore focus on unmount
```

#### React Query — Mutation (Optimistic)
```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';
function useUpdatePost(){
  const qc = useQueryClient();
  return useMutation({
    mutationFn: (p:any)=> fetch('/api/posts/'+p.id,{method:'PUT',headers:{'Content-Type':'application/json'},body:JSON.stringify(p)}).then(r=>r.json()),
    onMutate: async (p)=>{ await qc.cancelQueries({queryKey:['posts']}); const prev=qc.getQueryData<any>(['posts']); qc.setQueryData(['posts'], (old:any)=> ({...old, data: old.data.map((x:any)=> x.id===p.id?{...x,...p}:x)})); return {prev}; },
    onError: (_e,_p,ctx)=> ctx?.prev && qc.setQueryData(['posts'], ctx.prev),
    onSettled: ()=> qc.invalidateQueries({queryKey:['posts']})
  });
}
```

#### Suspense + Code Splitting
```tsx
import { lazy, Suspense } from 'react';
const Reports = lazy(()=> import('./Reports'));
export default function App(){ return <Suspense fallback={<div>Loading…</div>}><Reports/></Suspense>; }
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 12 with emphasis on reliability and performance.*
- **Tech Qs**:  
  1) *How do you ensure idempotency for mutations?*  
  2) *Explain a query optimization you shipped and its impact.*


### Interview Focus
- Patterns: Event Sourcing
- MySQL: EXPLAIN
- Pitfalls:
- Avoid SELECT *; keep covering index effective.
- Set consistent ordering for pagination.
- Add jitter to retries to avoid herd effect.
- Keep transactions short; avoid long-held locks.

**Case Study D12**: Implement idempotent create-order with DB upsert + Redis lock; return 201 on first call, 200 cached for retries.

### Practice Tasks
- Build the feature end-to-end; write why each design choice fits interview expectations.

### Acceptance Criteria
- BE: đúng schema/status; test Feature; log structured.
- FE: list/mutation OK; handle loading/error/empty; mutation rollback tốt.
- DB: có EXPLAIN + index/partition đề xuất.


### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Laravel](https://laravel.com/docs)
- [Queues](https://laravel.com/docs/queues)
- [React](https://react.dev/learn)
- [React Query](https://tanstack.com/query/latest)

### Checklist
- [ ] Code BE+FE end-to-end
- [ ] 1 Feature test (BE) + 1 RTL/Playwright (FE)
- [ ] Ghi chú EXPLAIN & tối ưu
- [ ] 2 câu trả lời kỹ thuật (EN)


### Reflection
- What trade-off today would you defend in an interview? Why?