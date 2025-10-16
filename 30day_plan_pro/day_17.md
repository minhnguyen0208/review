# Day 17 — Interview Prep Day 17

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

#### Design Pattern — CQRS
#### Architectural Pattern — CQRS (Laravel-style)
```php
// Command (Write)
class CreateOrderCommand { public function __construct(public array $data){} }
class CreateOrderHandler {
  public function handle(CreateOrderCommand $cmd){
    return DB::transaction(function() use($cmd){
      $order = Order::create($cmd->data);
      event(new OrderCreated($order->id, $order->total));
      return $order;
    });
  }
}

// Projector (Read model denormalized)
class OrderProjector {
  public function onOrderCreated(OrderCreated $e){
    ReadOrder::updateOrCreate(['id'=>$e->id], ['total'=>$e->total, 'status'=>'created']);
  }
}
```
**Trade-offs**: scale đọc tốt; **Cons**: phức tạp, eventual consistency.


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

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 17 with emphasis on reliability and performance.*
- **Tech Qs**:  
  1) *How do you ensure idempotency for mutations?*  
  2) *Explain a query optimization you shipped and its impact.*


### Interview Focus
- Patterns: CQRS
- MySQL: EXPLAIN
- Pitfalls:
- Avoid SELECT *; keep covering index effective.
- Set consistent ordering for pagination.
- Add jitter to retries to avoid herd effect.
- Keep transactions short; avoid long-held locks.

**Case Study D17**: Implement idempotent create-order with DB upsert + Redis lock; return 201 on first call, 200 cached for retries.

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
- [React Query](https://tanstack.com/query/latest)
- [Validation](https://laravel.com/docs/validation)

### Checklist
- [ ] Code BE+FE end-to-end
- [ ] 1 Feature test (BE) + 1 RTL/Playwright (FE)
- [ ] Ghi chú EXPLAIN & tối ưu
- [ ] 2 câu trả lời kỹ thuật (EN)


### Reflection
- What trade-off today would you defend in an interview? Why?