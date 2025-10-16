# Day 19 — Interview Prep Day 19

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Job Middleware
```php
class SkipIfStale { public function handle($job,$next){ if($job->updatedRecently()) return; $next($job); } }
```

#### Horizon
```txt
- Monitor queue throughput, failures, retries; tune backoff/maxTries
```

#### Casting & Accessors
```php
protected $casts = ['meta'=>'array'];
public function getTitleAttribute($v){ return trim($v); }
```

#### Design Pattern — Factory
```php
interface Notifier { public function send(string $to, string $msg): void; }
class EmailNotifier implements Notifier { public function send(string $to, string $msg){ Mail::to($to)->send(new GenericMail($msg)); } }
class SmsNotifier implements Notifier { public function send(string $to, string $msg){ app('twilio')->messages->create($to, ['from'=>config('twilio.from'),'body'=>$msg]); } }
class NotifierFactory {
  public static function make(string $channel): Notifier {
    return match($channel){ 'sms'=>new SmsNotifier(), 'email'=>new EmailNotifier(), default=>throw new InvalidArgumentException('Unsupported channel') };
  }
}
// Usage
$notifier = NotifierFactory::make(config('notify.channel','email'));
$notifier->send($user->email, 'Welcome!');
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

#### useReducer
```tsx
function reducer(s, a){ switch(a.type){ case 'inc': return {...s, n:s.n+1}; default:return s; } }
const [state, dispatch] = useReducer(reducer, { n:0 });
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 19 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: backpressure, consistency, denormalization, isolation level, memoization, observability, partitioning, rollback
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