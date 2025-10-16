# Day 13 — Coding Test

_Generated from your plan on 2025-10-01 06:24._

### Plan Notes
_(none)_

## Backend (BE)

```php
// routes/api.php
Route::apiResource('posts', PostController::class);
```

```php
class PostController extends Controller {
  public function index(Request $r){
    $q = Post::query()
      ->when($r->q, fn($q,$v)=>$q->where('title','like',"%$v%"))
      ->with(['author:id,name'])
      ->orderByDesc('created_at');
    return PostResource::collection($q->paginate($r->integer('per_page',20)));
  }
  public function store(StorePostRequest $req){
    $post = Post::create($req->validated());
    return (new PostResource($post))->response()->setStatusCode(201);
  }
}
```

```php
class StorePostRequest extends FormRequest {
  public function rules(){
    return ['title'=>'required|string|max:255','body'=>'required|string','author_id'=>'required|integer|exists:users,id'];
  }
}
```

## Frontend (FE)

```tsx
const [count,setCount] = useState(0);
```

```tsx
useEffect(()=>{ document.title = `Count ${count}`; return ()=>{/* cleanup */}; }, [count]);
```

```tsx
const expensive = useMemo(()=> heavy(data), [data]);
```

```tsx
const onClick = useCallback(()=> doThing(id), [id]);
```

### English (Interview)
- **Pitch (30–45s)**: *Hi, I'm Minh, a Laravel + React engineer. Today I focused on coding test.*
- **Tech Qs**: 1) *Walk through API → DB → UI.* 2) *How to avoid N+1 and ensure idempotency?*
- **Vocab**: latency, throughput, idempotency, pagination, memoization, partitioning, rollback, retry


### Practice Tasks
- Build end-to-end; add 1 BE + 1 FE test; capture EXPLAIN & index/partition note.

### Acceptance Criteria
- Correct schema/status + normalized error; FE handles loading/error/empty; show EXPLAIN.

### Docs & References
- (add relevant docs)

### Reflection
- Which trade-off today will you defend in interview and why?