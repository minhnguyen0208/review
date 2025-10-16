# Day 28 — Interview Prep Day 28

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### API Versioning
```php
Route::prefix('v1')->group(fn()=> Route::apiResource('posts', V1\PostController::class));
```

#### Config Caching
```bash
php artisan config:cache && php artisan route:cache
```

#### Queues+Retry+DLQ
```php
class SyncJob implements ShouldQueue {
  use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
  public $tries=3; public $backoff=[60,300,900];
  public function handle(){ Partner::syncOrFail(); }
}
// Horizon for throughput/failures; DLQ for analysis.
```

#### Design Pattern — Unit of Work
```php
class UserOnboarding {
  public function run(array $data){
    DB::transaction(function() use ($data){
      $user = User::create($data['user']);
      Profile::create(['user_id'=>$user->id] + $data['profile']);
      event(new UserOnboarded($user->id));
    });
  }
}
```

#### MySQL — Partition by Date
```sql
ALTER TABLE logs
PARTITION BY RANGE (TO_DAYS(created_at))(
  PARTITION p2025 VALUES LESS THAN (TO_DAYS('2026-01-01')),
  PARTITION pmax  VALUES LESS THAN MAXVALUE
);
```
**Gotcha**: partitioning column must appear in every UNIQUE/PRIMARY key (MySQL 8.0).


## Frontend (FE)

#### useEffect
```tsx
useEffect(()=>{ document.title = `Count ${count}`; return () => {/* cleanup */}; }, [count]);
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 28 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: backpressure, debounce, denormalization, latency, memoization, optimistic UI, partitioning, rate limiting
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
- [Queues](https://laravel.com/docs/queues)
- [React](https://react.dev/learn)

### Reflection
- What trade-off today will you highlight in an interview?