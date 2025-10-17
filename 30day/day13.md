# Day 13 — Design Patterns in Laravel (Repository/Service, Factory, Observer) | React Router v6 + Code Splitting

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (Laravel)**
- **Repository + Service layers**: tách persistence khỏi business logic; dễ test/mock.
- **Factory/Strategy**: chọn implementation theo config/runtime.
- **Observer / Events-Listeners**: side‑effects tách rời, dễ retry/queue.
- **Validation & Transaction boundaries**: service chịu trách nhiệm consistency.

**Frontend (React)**
- **React Router v6**: route config, nested routes, loader‑like patterns.
- **Code splitting**: `lazy` + `Suspense`, preloading khi hover.
- **Error & NotFound routes**: UX nhất quán, đo TTFB/LCP sau split.

---

## Backend (BE)

> **Mục tiêu phỏng vấn:** trình bày cách tách lớp, test dễ, và mở rộng không chạm core (Open/Closed).

### 1) Repository + Service
```php
// app/Repositories/PostRepository.php
interface PostRepository {
  public function find(int $id): ?Post;
  public function list(array $filter): LengthAwarePaginator;
  public function create(array $data): Post;
}

class EloquentPostRepository implements PostRepository {
  public function find(int $id): ?Post { return Post::find($id); }
  public function list(array $filter): \Illuminate\Contracts\Pagination\LengthAwarePaginator {
    return Post::query()
      ->when($filter['author_id'] ?? null, fn($q,$v)=>$q->where('author_id',$v))
      ->orderByDesc('created_at')
      ->paginate($filter['per_page'] ?? 20);
  }
  public function create(array $data): Post { return Post::create($data); }
}

// app/Services/PostService.php
class PostService {
  public function __construct(private PostRepository $repo) {}
  public function publish(array $data): Post {
    return DB::transaction(function() use ($data){
      $post = $this->repo->create($data + ['status'=>'DRAFT']);
      // business rule...
      $post->update(['status'=>'PUBLISHED']);
      event(new \App\Events\PostPublished($post->id));
      return $post;
    });
  }
}
```

**DI binding**
```php
// AppServiceProvider
$this->app->bind(PostRepository::class, EloquentPostRepository::class);
```

### 2) Factory/Strategy Pattern
```php
interface StorageStrategy { public function put(string $key, string $content): void; }

class S3Storage implements StorageStrategy {
  public function put(string $key, string $content): void { Storage::disk('s3')->put($key, $content); }
}
class LocalStorage implements StorageStrategy {
  public function put(string $key, string $content): void { Storage::disk('local')->put($key, $content); }
}

// Factory
class StorageFactory {
  public static function make(): StorageStrategy {
    return config('filesystems.default') === 's3' ? new S3Storage() : new LocalStorage();
  }
}

// Usage in Service
$strategy = StorageFactory::make();
$strategy->put("posts/{$post->id}.json", json_encode($post));
```

### 3) Observer / Events-Listeners
```php
// app/Observers/PostObserver.php
class PostObserver {
  public function created(Post $post){
    dispatch(new \App\Jobs\IndexPostJob($post->id)); // non-blocking
  }
}
// AppServiceProvider
Post::observe(PostObserver::class);

// or Events-Listeners
class PostPublished { public function __construct(public int $postId){} }
class SendNotifyOnPublished implements ShouldQueue {
  public function handle(PostPublished $e){ Notification::send(...); }
}
```

### 4) Testing Patterned Code
```php
public function test_publish_queues_notification(){
  Bus::fake();
  $svc = app(PostService::class);
  $svc->publish(['title'=>'x','body'=>'y','author_id'=>User::factory()->create()->id]);
  Bus::assertDispatched(\App\Jobs\IndexPostJob::class);
}
```

---

## Frontend (FE)

> **Mục tiêu phỏng vấn:** routing rõ ràng, tải nhanh nhờ split hợp lý, không vỡ UX khi lỗi.

### 1) React Router v6 — Nested routes
```tsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import AppLayout from './AppLayout';
import Home from './Home';
import PostDetail from './PostDetail';

const router = createBrowserRouter([
  { path: '/', element: <AppLayout/>,
    children: [
      { index: true, element: <Home/> },
      { path: 'posts/:id', element: <PostDetail/> },
      { path: '*', element: <p>Not Found</p> }
    ]
  }
]);
export default function App(){ return <RouterProvider router={router}/>; }
```

### 2) Code Splitting + Suspense
```tsx
const PostDetail = React.lazy(()=> import('./PostDetail'));
function App(){
  return (
    <React.Suspense fallback={<p>Loading...</p>}>
      <RouterProvider router={router}/>
    </React.Suspense>
  );
}
```

**Preload khi hover (nhẹ)**
```tsx
function LinkPrefetch({to, children}:{to:string, children:React.ReactNode}){
  return <a href={to} onMouseEnter={()=> import('./PostDetail') }>{children}</a>;
}
```

---

## English for Interview (skills only)

**Architecture & Patterns (Q&A)**
- *Why use Repository + Service instead of fat controllers?* — Separation of concerns, easier testing/mocking, swap persistence without breaking business logic.
- *When to choose Observer vs Event/Listener?* — Observer cho lifecycle của model; Event/Listener phù hợp domain events đa nguồn, có queue/retry.
- *How do you justify adding a Factory/Strategy?* — When there are multiple interchangeable implementations; improves extensibility and testability.

---

### Query Optimization (Supplement)
> Chủ đề mới (khác các ngày trước): **Join order heuristics**, **FK indexes bắt buộc**, **Column order trong composite index**, **EXPLAIN FORMAT=JSON**.

**Foreign key indexes (bắt buộc cho hiệu năng)**
```sql
ALTER TABLE comments ADD INDEX idx_comments_post (post_id);
ALTER TABLE posts ADD INDEX idx_posts_author (author_id);
```

**Column order heuristics**
- Cột có **selectivity cao** trước; khớp thứ tự `WHERE` + `ORDER BY` khi có thể.
```sql
CREATE INDEX idx_posts_author_created ON posts(author_id, created_at DESC);
```

**EXPLAIN FORMAT=JSON + cost_info**
```sql
EXPLAIN FORMAT=JSON
SELECT p.id, p.title FROM posts p
JOIN comments c ON c.post_id = p.id
WHERE p.author_id = ? AND c.flagged = 1\G
-- Xem "cost_info", "chosen_range_access_summary"
```

**Hành động sau EXPLAIN**
- Bổ sung index còn thiếu (đặc biệt FK/sort).
- Giảm cột select (covering).
- Đổi join order bằng cách thêm điều kiện lên bảng dẫn (sargable).

---

### Activity
- Trích xuất 1 flow và refactor sang Repository + Service + Event/Listener.
- Áp dụng React Router v6 + lazy cho 1 trang chi tiết.
- Kiểm tra FK indexes cho các mối quan hệ chính, thêm index nếu thiếu.

### Practice Tasks
- Viết test cho `PostService::publish()` với Bus::fake/Notification::fake.
- Tạo route nested và màn hình NotFound.
- (Supplement) Chạy `EXPLAIN FORMAT=JSON` cho 1 query joins; ghi lại cost_info.

### Acceptance Criteria
- Service/Repository tách rõ, test pass; side‑effects sang listener/job.
- Router hoạt động với nested routes + lazy; UX không bị trắng khi loading/error.
- Có ghi chú tối ưu query (FK index/column order/EXPLAIN JSON).

### Docs & References
- Laravel Events & Queues: https://laravel.com/docs/events
- Service Container/DI: https://laravel.com/docs/container
- React Router v6: https://reactrouter.com/en/main
- React lazy/Suspense: https://react.dev/reference/react/lazy
- MySQL EXPLAIN JSON: https://dev.mysql.com/doc/refman/8.0/en/explain-output.html#explain-json-output
