# Day 18 — Interview Prep Day 18

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Broadcasting
```php
broadcast(new OrderCreated($order))->toOthers();
```

#### Job Middleware
```php
class SkipIfStale { public function handle($job,$next){ if($job->updatedRecently()) return; $next($job); } }
```

#### Observers
```php
Post::observe(PostObserver::class); // created/updated/deleted
```

#### Design Pattern — Event Sourcing
```php
// Event table (simplified)
/*
CREATE TABLE events (
  id BIGINT PRIMARY KEY AUTO_INCREMENT,
  aggregate_id BIGINT, type VARCHAR(64), payload JSON,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
*/
function appendEvent($aggId, $type, $payload){
  DB::table('events')->insert(['aggregate_id'=>$aggId,'type'=>$type,'payload'=>json_encode($payload)]);
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

#### useRef
```tsx
const inputRef = useRef<HTMLInputElement>(null); useEffect(()=>{ inputRef.current?.focus(); },[]);
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 18 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: backpressure, consistency, denormalization, isolation level, latency, observability, retries, throughput
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