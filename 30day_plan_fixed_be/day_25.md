# Day 25 — Interview Prep Day 25

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Casting & Accessors
```php
protected $casts = ['meta'=>'array'];
public function getTitleAttribute($v){ return trim($v); }
```

#### Soft Deletes
```php
use SoftDeletes; Post::withTrashed()->find($id); Post::onlyTrashed()->restore();
```

#### API+Validation+Resource
```php
// routes/api.php
Route::apiResource('posts', PostController::class);

// App/Http/Controllers/PostController.php
class PostController extends Controller {
  public function index(Request $r){
    $q = Post::query()
      ->when($r->q, fn($q,$v)=>$q->where('title','like',"%$v%"))
      ->when($r->author_id, fn($q,$v)=>$q->where('author_id',$v))
      ->orderByDesc('created_at');
    return PostResource::collection($q->paginate($r->integer('per_page',20)));
  }
  public function store(StorePostRequest $req){
    $post = Post::create($req->validated());
    return (new PostResource($post))->response()->setStatusCode(201);
  }
}
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

#### MySQL — Idempotency Key
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
**Race**: rely on PK + upsert to prevent duplicates.


## Frontend (FE)

#### A11y Modal
```tsx
// focus first tabbable; ESC to close; restore focus on unmount
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 25 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: backpressure, consistency, debounce, denormalization, latency, memoization, observability, partitioning
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
- [Validation](https://laravel.com/docs/validation)

### Reflection
- What trade-off today will you highlight in an interview?