# Day 01 — REST API + Eloquent Relations + React Core Hooks

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (Laravel)**
- RESTful routes với `apiResource`, cấu trúc Controller, **Request Validation** bằng FormRequest.  
- **Eloquent relations**: `belongsTo`, `hasMany`; tránh **N+1** với `with()` (kèm giới hạn), `withCount`, subquery `addSelect`.  
- **API Resource** để chuẩn hoá JSON; status code: **201/422/400/500**; quy ước lỗi `{code,message,errors[]}` cho FE.  
- **MySQL**: tạo **covering index** `(author_id, created_at DESC)`; đọc `EXPLAIN` (type/rows/Extra) → loại **filesort**.  
- **Design Pattern**: **Repository** để tách Controller khỏi ORM, tăng testability (điểm cộng khi phỏng vấn).

**Frontend (React)**
- **Ôn core hooks trước**: `useState`, `useEffect`, `useMemo`, `useCallback`.  
- Triển khai UI **loading / error / empty**; cleanup trong `useEffect` để tránh setState khi unmount.  
- List cơ bản từ `/api/posts`, touch nhẹ performance (memo hoá danh sách dẫn xuất, callback ổn định).

**Deliverables**
- `GET /posts`, `POST /posts` (201/422), tài liệu JSON + validation, **1 Feature test** (BE).  
- FE list gồm loading/error/empty rõ ràng, không warning, error có `role="alert"`.  
- Ghi `EXPLAIN` kèm giải thích **tại sao index** xoá filesort.

---
- **Eager loading nâng cao** với ràng buộc (giới hạn số bản ghi con), tránh N+1 có kiểm soát.
- **Model Scopes** (local/global) để tái sử dụng điều kiện truy vấn.
- **Subqueries** (`withCount`, `addSelect`) để tính số liệu mà không tạo N+1.
- **React Hooks refresh patterns**: version key, interval + cleanup để refetch an toàn.
- **English**: luyện **Self‑introduction (60s)**: Name → Stack → 1–2 thành tựu → Strengths → Fit.

## Backend (BE)

### Eloquent Scopes (local & global)
**Local scope:** gom logic lọc phổ biến vào Model.
```php
// app/Models/Post.php
class Post extends Model {
  public function scopePublished($q){ return $q->where('status','PUBLISHED'); }
}
// usage
$rows = Post::published()->with('author:id,name')->paginate(20);
```
**Global scope:** tự động áp vào mọi query của model.
```php
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Scope;

class NotDeletedScope implements Scope{
  public function apply(Builder $builder, Model $model){
    $builder->whereNull($model->getTable().'.deleted_at');
  }
}
// app/Models/Post.php (booted)
protected static function booted(){
  static::addGlobalScope(new NotDeletedScope);
}
```


> **Mục tiêu phỏng vấn:** API gọn, đúng chuẩn REST; tránh N+1; JSON nhất quán; có cơ sở **về index/EXPLAIN** để bảo vệ giải pháp.

**1) Routes & Controller (REST, phân trang, lọc) — trước khi xem code**  
Vì sao: Chuẩn REST giúp dễ đoán hành vi endpoint; phân trang + tìm kiếm là case phổ biến khi interview.  
Phỏng vấn thường hỏi: “Bạn structure Controller thế nào để dễ test/maintain?” → tách validate (FormRequest), tách query (Repository).

```php
// routes/api.php
Route::apiResource('posts', PostController::class);

// app/Http/Controllers/PostController.php
class PostController extends Controller {
  public function index(Request $r){
    $q = Post::query()
      ->when($r->q, fn($q,$v)=>$q->where('title','like',"%$v%"))
      ->with(['author:id,name'])                // tránh N+1 cho author
      ->orderByDesc('created_at');

    return PostResource::collection(
      $q->paginate($r->integer('per_page', 20))
    );
  }

  public function store(StorePostRequest $req){
    $post = Post::create($req->validated());
    return (new PostResource($post))->response()->setStatusCode(201);
  }
}
```

**2) Validation (FormRequest) — trước khi xem code**  
Vì sao: đẩy rule ra FormRequest giúp Controller mảnh, dễ test; mapping lỗi chuẩn để FE hiểu được.

```php
// app/Http/Requests/StorePostRequest.php
class StorePostRequest extends FormRequest {
  public function rules(){
    return [
      'title'     => 'required|string|max:255',
      'body'      => 'required|string',
      'author_id' => 'required|integer|exists:users,id',
    ];
  }
  public function authorize(){ return true; }
}
```

**3) Eloquent relations + metrics (withCount + subquery) — trước khi xem code**  
Vì sao: phỏng vấn hay hỏi tránh **N+1**; đồng thời vẫn có **số liệu** (count) và **latest timestamp** mà không thêm N+1.

```php
// app/Models/Post.php
class Post extends Model {
  public function author(){ return $this->belongsTo(User::class,'author_id'); }
  public function comments(){ return $this->hasMany(Comment::class); }
}

// popular posts + latest_comment_at mà không N+1
$posts = Post::withCount(['comments','likes'])
  ->addSelect(['latest_comment_at' => Comment::select('created_at')
    ->whereColumn('comments.post_id','posts.id')->latest()->limit(1)])
  ->orderByDesc('comments_count')
  ->paginate(20);
```

**4) Repository pattern (lý do dùng trong interview) — trước khi xem code**  
Vì sao: tách query khỏi Controller, giúp test độc lập, **dễ thay đổi chiến lược** (đổi sang read model/caching mà không chạm Controller).

```php
interface PostRepository {
  public function paginate(array $filters, int $perPage=20): \Illuminate\Contracts\Pagination\LengthAwarePaginator;
}

class EloquentPostRepository implements PostRepository {
  public function paginate(array $filters, int $perPage=20){
    return Post::query()
      ->when($filters['q']??null, fn($q,$v)=>$q->whereFullText('title',$v))
      ->with(['author:id,name'])
      ->orderByDesc('created_at')
      ->paginate($perPage);
  }
}
```

**5) MySQL: covering index + đọc EXPLAIN — trước khi xem code**  
Vì sao: câu hỏi “tối ưu truy vấn” gần như chắc gặp; cần **lý luận chuẩn** về vì sao index này xoá filesort.

```sql
-- Chỉ select các cột cần (id, title, created_at) + index khớp điều kiện/ordering
CREATE INDEX idx_posts_author_created ON posts(author_id, created_at DESC);

EXPLAIN SELECT id, title, created_at
FROM posts
WHERE author_id = ?
ORDER BY created_at DESC
LIMIT 20;
-- Kỳ vọng: type=ref/range; rows nhỏ; Extra: "Using index"; tránh "Using filesort".
```

---

## Frontend (FE)

### React Hooks — Refresh patterns (không dùng lib)
Một số cách "refresh" dữ liệu khi dùng core hooks:
```tsx
// 1) Refresh bằng "version key"
const [version, setVersion] = useState(0);
useEffect(()=>{
  let alive = true;
  (async ()=>{
    const r = await fetch('/api/posts?ver='+version);
    const j = await r.json();
    if(alive) setPosts(j.data);
  })();
  return ()=>{ alive=false };
}, [version]);
// Button
<button onClick={()=> setVersion(v=>v+1)}>Refresh</button>
```

```tsx
// 2) Refresh theo "interval" có cleanup
useEffect(()=>{
  const id = setInterval(()=> setVersion(v=>v+1), 30_000);
  return ()=> clearInterval(id);
}, []);
```


> **Mục tiêu phỏng vấn:** render list có đủ **loading/error/empty**; ưu tiên **core hooks**; cleanup chuẩn để không rò rỉ.

**1) State & Effect (và vì sao cần cleanup)**  
Vì sao: fetch bất đồng bộ dễ gây “setState on unmounted”. Cleanup đảm bảo **không leak** khi component bị unmount.

```tsx
const [posts, setPosts] = useState<any[]>([]);
const [loading, setLoading] = useState(true);
const [error, setError] = useState<string|null>(null);

useEffect(()=>{
  let alive = true;
  (async ()=>{
    try{
      const r = await fetch('/api/posts');
      if(!r.ok) throw new Error('Network');
      const data = await r.json();
      if(alive) setPosts(data.data);
    }catch(e:any){
      if(alive) setError(e.message || 'Error');
    }finally{
      if(alive) setLoading(false);
    }
  })();
  return ()=>{ alive=false; }; // cleanup: tránh setState sau unmount
}, []);
```

**2) UI states + tối ưu nhỏ (memo + callback ổn định)**  
Vì sao: giúp trả lời câu hỏi “Bạn quản lý performance FE như thế nào?” ở mức **thực dụng**.

```tsx
const filtered = useMemo(()=> posts.filter(p => !!p.title), [posts]);
const onRefresh = useCallback(()=> window.location.reload(), []);

if (loading) return <p>Loading…</p>;
if (error)   return <p role="alert">Failed: {error}</p>;
if (filtered.length === 0) return <p>No posts yet.</p>;

return (
  <div>
    <button onClick={onRefresh}>Refresh</button>
    <ul>{filtered.map(p => <li key={p.id}>{p.title}</li>)}</ul>
  </div>
);
```

---

## English for Interview

### English — Self‑Introduction (60s template)
- **Structure**: Name → Years/stack → 1–2 impactful projects → What you’re good at → What you want next.
- **Sample**:  
  *Hi, I’m Minh. I’ve spent the last 3 years building Laravel APIs and React front‑ends. Recently, I designed a paginated REST API with proper indexing and a React UI with solid loading/error states. I focus on performance (EXPLAIN plans, caching) and clear error contracts for the UI. I’m excited to bring this pragmatism to a product team where I can own features end‑to‑end.*
- **Tips**: keep to ~60s; add measurable outcomes (latency ↓, errors ↓), and align with the role’s stack.
 (skills only)

**Technical Q&A (có khung trả lời ngắn gọn)**
1) *How do you prevent N+1 while keeping memory usage in check?*  
   **Frame**: Constrained eager-loading (`with` + limit), `withCount`, subqueries cho aggregates; đo bằng query log + memory profile.  
2) *Why does `(author_id, created_at DESC)` remove filesort for this query?*  
   **Frame**: Index khớp filter + order; chỉ select cột trong index → **Using index**; pagination ổn định.  
3) *How do you design an error envelope the UI can rely on?*  
   **Frame**: `{code,message,errors[]}`; 4xx/5xx map rõ; FE hiển thị `role="alert"` + map field errors cho form.

**Behavioral (S.T.A.R.)**
- *Tell me about a production incident you owned.*  
  **S/T**: spike 500 sau deploy → **A**: rollback, thêm EXPLAIN gate vào CI, chuẩn hoá lỗi → **R**: hết regression tháng sau.

**Useful phrases**
- “I validated this with an EXPLAIN plan and a before/after latency chart.”  
- “We normalized errors so the UI could deterministically render states.”  
- “Aggregates were moved to subqueries to keep eager loads small.”

---

### Activity
**Backend (Laravel)**
- RESTful routes với `apiResource`, cấu trúc Controller, **Request Validation** bằng FormRequest.
- **Eloquent relations**: `belongsTo`, `hasMany`; tránh **N+1** với `with()` (kèm giới hạn), `withCount`, subquery `addSelect`.
- **API Resource** để chuẩn hoá JSON; status code: **201/422/400/500**; quy ước lỗi `{code,message,errors[]}` cho FE.
- **MySQL**: tạo **covering index** `(author_id, created_at DESC)`; đọc `EXPLAIN` (type/rows/Extra) → loại **filesort**.
- **Design Pattern**: **Repository** để tách Controller khỏi ORM, tăng testability (điểm cộng khi phỏng vấn).
- Implement BE endpoints and run database migration/index creation
- Verify EXPLAIN plan and capture a note/screenshot
- Build FE screen and verify loading/error/empty states
- Write 1 short interview takeaway (2–3 sentences)

### Practice Tasks
- `GET /posts`, `POST /posts` dùng **FormRequest** + **PostResource**; viết **1 Feature test** cho index/store.  
- FE list có loading/error/empty; memo hoá danh sách dẫn xuất; nút refresh đơn giản.  
- Lưu `EXPLAIN` + viết 5 dòng giải thích **vì sao** index khớp `WHERE author_id` + `ORDER BY created_at DESC`.

### Acceptance Criteria
- **201/422** đúng; lỗi JSON chuẩn `{code,message,errors}`.  
- `EXPLAIN` **không** còn filesort; index đúng thứ tự.  
- UI có **loading/error/empty**; không warning; error có `role="alert"`.

### Docs & References
- Laravel: https://laravel.com/docs  
- Eloquent: https://laravel.com/docs/eloquent  
- Validation: https://laravel.com/docs/validation  
- Database (MySQL): https://dev.mysql.com/doc/  
- React (Core Hooks): https://react.dev/learn
