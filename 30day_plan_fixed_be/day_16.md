# Day 16 — Interview Prep Day 16

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Custom Validation Rules
```php
Validator::extend('safe_slug', fn($attr,$val)=> preg_match('/^[a-z0-9-]+$/',$val));
```

#### Notifications
```php
$user->notify(new ResetPasswordNotification($token));
```

#### Testing (Pest)
```php
test('api returns resource shape', function() {
  $res = $this->getJson('/api/posts')->assertOk()->json();
  expect($res)->toHaveKeys(['data','links','meta']);
});
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

#### MySQL — Covering Index
```sql
-- Bad: SELECT *
SELECT * FROM posts WHERE author_id = ? ORDER BY created_at DESC LIMIT 20;

-- Good: only needed cols + matching composite index
CREATE INDEX idx_posts_author_created ON posts(author_id, created_at DESC);
SELECT id, title, created_at FROM posts WHERE author_id=? ORDER BY created_at DESC LIMIT 20;
```
**Reasoning**: "Using index" in EXPLAIN => avoid back-to-table lookup → lower IO/latency.


## Frontend (FE)

#### useMemo
```tsx
const expensive = useMemo(()=> heavy(data), [data]); // memo only if heavy & stable
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 16 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: backpressure, consistency, idempotency, latency, observability, partitioning, rollback, throughput
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