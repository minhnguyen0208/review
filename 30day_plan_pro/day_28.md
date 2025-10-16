# Day 28 — Interview Prep Day 28

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

#### Design Pattern — Unit of Work
#### Design Pattern — Unit of Work (User Onboarding)
```php
class UserOnboarding {
  public function run(array $data) {
    DB::transaction(function() use ($data){
      $user = User::create($data['user']);
      Profile::create(['user_id'=>$user->id] + $data['profile']);
      event(new UserOnboarded($user->id));
    });
  }
}
```
**Note**: giới hạn phạm vi transaction; nhất quán thứ tự lock để tránh deadlock.


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

#### Error Boundary
```tsx
class ErrorBoundary extends React.Component<{children:React.ReactNode},{hasError:boolean}>{
  state={hasError:false}; componentDidCatch(err:any){ this.setState({hasError:true}); }
  render(){ return this.state.hasError? <p>Something went wrong.</p> : this.props.children; }
}
```

#### Suspense + Code Splitting
```tsx
import { lazy, Suspense } from 'react';
const Reports = lazy(()=> import('./Reports'));
export default function App(){ return <Suspense fallback={<div>Loading…</div>}><Reports/></Suspense>; }
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 28 with emphasis on reliability and performance.*
- **Tech Qs**:  
  1) *How do you ensure idempotency for mutations?*  
  2) *Explain a query optimization you shipped and its impact.*


### Interview Focus
- Patterns: Unit of Work
- MySQL: Partition by Date
- Pitfalls:
- Avoid SELECT *; keep covering index effective.
- Set consistent ordering for pagination.
- Add jitter to retries to avoid herd effect.
- Keep transactions short; avoid long-held locks.

**Case Study D28**: Implement idempotent create-order with DB upsert + Redis lock; return 201 on first call, 200 cached for retries.

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

### Checklist
- [ ] Code BE+FE end-to-end
- [ ] 1 Feature test (BE) + 1 RTL/Playwright (FE)
- [ ] Ghi chú EXPLAIN & tối ưu
- [ ] 2 câu trả lời kỹ thuật (EN)


### Reflection
- What trade-off today would you defend in an interview? Why?