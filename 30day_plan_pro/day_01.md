# Day 01 — Interview Prep Day 1

_Generated on 2025-10-01 04:00._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### API+Validation+Resource
```php
// routes/api.php
Route::apiResource('posts', PostController::class);

// Controller
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

// Request
class StorePostRequest extends FormRequest {
  public function rules(): array {
    return ['title'=>'required|string|max:255','body'=>'required|string','author_id'=>'required|integer|exists:users,id'];
  }
}
```

#### Eloquent Performance
```php
// Eager loading with constraints
$posts = Post::with(['comments'=>fn($q)=>$q->latest()->limit(5)])->paginate(20);

// Subquery: latest comment timestamp
$posts = Post::addSelect(['latest_comment_at'=>Comment::select('created_at')
  ->whereColumn('comments.post_id','posts.id')->latest()->limit(1)])->get();

// Aggregate counters
$posts = Post::withCount(['comments','likes'])->orderByDesc('comments_count')->limit(50)->get();

// Scope + when + fulltext
class Post extends Model { function scopeOfAuthor($q,$a){return $q->where('author_id',$a);} }
$rows = Post::ofAuthor($authorId)->when($q, fn($x)=>$x->whereFullText('title',$q))->paginate();
```

#### Design Pattern — Factory
#### Design Pattern — Factory (Channel Notifier)
```php
interface Notifier { public function send(string $to, string $msg): void; }
class EmailNotifier implements Notifier { public function send(string $to, string $msg){ Mail::to($to)->send(new GenericMail($msg)); } }
class SmsNotifier implements Notifier { public function send(string $to, string $msg){ app('twilio')->messages->create($to, ['from'=>config('twilio.from'),'body'=>$msg]); } }

class NotifierFactory {
  public static function make(string $channel): Notifier {
    return match($channel){
      'sms'   => new SmsNotifier(),
      'email' => new EmailNotifier(),
      default => throw new InvalidArgumentException('Unsupported channel')
    };
  }
}

// Usage (Service layer)
$notifier = NotifierFactory::make(config('notify.channel','email'));
$notifier->send($user->email, 'Welcome to our product!');
```
**When to use**: nhiều biến thể tạo object; **Pitfall**: over-engineer nếu chỉ 1-2 loại.


#### MySQL — Covering Index
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


## Frontend (FE)

#### RHF + Zod
```tsx
import { useForm } from 'react-hook-form';
import { z } from 'zod'; import { zodResolver } from '@hookform/resolvers/zod';
const schema = z.object({ email:z.string().email(), password:z.string().min(8) });
type Form = z.infer<typeof schema>;
export default function Login(){
  const { register, handleSubmit, formState:{errors} } = useForm<Form>({ resolver: zodResolver(schema) });
  const onSubmit = (d:Form)=> console.log(d);
  return (
    <form onSubmit={handleSubmit(onSubmit)} className="space-y-3 max-w-sm">
      <input {...register('email')} className="border p-2 w-full" placeholder="Email"/>
      <p className="text-sm text-red-600">{errors.email?.message}</p>
      <input type="password" {...register('password')} className="border p-2 w-full" placeholder="Password"/>
      <p className="text-sm text-red-600">{errors.password?.message}</p>
      <button className="border rounded px-4 py-2">Login</button>
    </form>
  );
}
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 1 with emphasis on reliability and performance.*
- **Tech Qs**:  
  1) *How do you ensure idempotency for mutations?*  
  2) *Explain a query optimization you shipped and its impact.*


### Interview Focus
- Patterns: Factory
- MySQL: Covering Index
- Pitfalls:
- Avoid SELECT *; keep covering index effective.
- Set consistent ordering for pagination.
- Add jitter to retries to avoid herd effect.
- Keep transactions short; avoid long-held locks.

**Case Study D1**: Implement idempotent create-order with DB upsert + Redis lock; return 201 on first call, 200 cached for retries.

### Practice Tasks
- Build the feature end-to-end; write why each design choice fits interview expectations.

### Acceptance Criteria
- BE: đúng schema/status; test Feature; log structured.
- FE: list/mutation OK; handle loading/error/empty; mutation rollback tốt.
- DB: có EXPLAIN + index/partition đề xuất.


### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Eloquent](https://laravel.com/docs/eloquent)
- [Laravel](https://laravel.com/docs)
- [React](https://react.dev/learn)
- [Validation](https://laravel.com/docs/validation)

### Checklist
- [ ] Code BE+FE end-to-end
- [ ] 1 Feature test (BE) + 1 RTL/Playwright (FE)
- [ ] Ghi chú EXPLAIN & tối ưu
- [ ] 2 câu trả lời kỹ thuật (EN)


### Reflection
- What trade-off today would you defend in an interview? Why?