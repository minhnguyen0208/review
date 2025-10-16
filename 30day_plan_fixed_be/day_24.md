# Day 24 — Interview Prep Day 24

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Observers
```php
Post::observe(PostObserver::class); // created/updated/deleted
```

#### Casting & Accessors
```php
protected $casts = ['meta'=>'array'];
public function getTitleAttribute($v){ return trim($v); }
```

#### S3 Multipart
```php
// 1) init multipart; 2) generate signed part URLs; 3) complete upload
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

#### MySQL — Batch Upsert
```php
$chunks = array_chunk($rows, 1000);
foreach($chunks as $chunk){
  Model::upsert($chunk, ['unique_key'], ['name','status','updated_at']);
}
```
**Tip**: wrap in transaction if business requires atomicity; avoid massive single statements.


## Frontend (FE)

#### Suspense + Split
```tsx
import { lazy, Suspense } from 'react';
const Reports = lazy(()=> import('./Reports'));
export default function App(){ return <Suspense fallback={<div>Loading…</div>}><Reports/></Suspense>; }
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 24 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: backpressure, consistency, isolation level, latency, memoization, observability, partitioning, rate limiting
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

### Reflection
- What trade-off today will you highlight in an interview?