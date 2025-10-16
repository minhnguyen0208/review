# Day 30 — Interview Prep Day 30

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### S3 Multipart
```php
// 1) init multipart; 2) generate signed part URLs; 3) complete upload
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

#### Auth+Sanctum
```php
$token = $user->createToken('web')->plainTextToken;
Route::middleware('auth:sanctum')->get('/me', fn(Request $r)=> $r->user());
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

#### useCallback
```tsx
const onClick = useCallback(()=> doThing(id), [id]); // stable ref for deps
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 30 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: backpressure, consistency, idempotency, isolation level, optimistic UI, partitioning, retries, throughput
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
- [React](https://react.dev/learn)
- [Sanctum](https://laravel.com/docs/sanctum)
- [Validation](https://laravel.com/docs/validation)

### Reflection
- What trade-off today will you highlight in an interview?