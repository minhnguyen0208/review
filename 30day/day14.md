# Day 14 — Advanced Queues (Middleware, Throttling, Unique Jobs, Horizon) | React Performance Deep‑Dive (useTransition, deferred, Web Vitals)

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (Laravel — Advanced)**
- **Job Middleware**: `withoutOverlapping`, `RateLimited`/throttling theo key.
- **Unique jobs** (dedup): khoá theo business key để ngăn dispatch trùng.
- **Chaining & Conditional batching**: `Bus::chain`, `Bus::batch` với hooks chi tiết.
- **Priorities & multiple queues**: route job vào hàng đợi chuyên trách; quan sát với **Horizon**.
- **DLQ pattern**: điều kiện đẩy vào dead‑letter (không retry mãi).

**Frontend (React — Advanced Perf)**
- **Concurrent features**: `useTransition`, `useDeferredValue` để tách cập nhật chậm/nhanh.
- **Web Vitals** (LCP/CLS/INP) & đo đạc qua `web-vitals` + React Profiler.
- **Bundle & code‑split chiến lược**: route‑level + component‑level; prefetch có kiểm soát.
- **Image/asset optimization**: lazy images, responsive sizes; tránh layout shift.

---

## Backend (BE)

> **Mục tiêu phỏng vấn:** kiểm soát tắc nghẽn, đảm bảo *idempotency* và *throughput* bằng middleware/throttle/unique keys; giám sát bằng Horizon.

### 1) Job Middleware — withoutOverlapping / RateLimited
```php
use Illuminate\Queue\Middleware\WithoutOverlapping;
use Illuminate\Queue\Middleware\RateLimited;

class SyncCompanyJob implements ShouldQueue {
  public function __construct(public int $companyId){}
  public function middleware(){
    return [
      new WithoutOverlapping("sync:company:{$this->companyId}"), // ngăn chạy song song cùng key
      (new RateLimited('sync-company'))->dontRelease(), // tôn trọng limiter tên 'sync-company'
    ];
  }
  public function handle(){
    // ... sync logic ...
  }
}
// AppServiceProvider: định nghĩa limiter
RateLimiter::for('sync-company', fn() => Limit::perMinute(30)->by('sync-company'));
```

### 2) Unique Jobs (dedup) với Cache lock
```php
class ExportReportJob implements ShouldQueue {
  public function __construct(public string $reportHash){}
  public function handle(){
    $lock = Cache::lock("job:export:{$this->reportHash}", 600);
    if (!$lock->get()) return; // đã có job tương tự đang chạy
    try {
      // ... thực thi export ...
    } finally {
      optional($lock)->release();
    }
  }
}
```

### 3) Chaining & Conditional Batching
```php
// Chain tuần tự (dừng khi 1 job fail)
Bus::chain([ new PrepareData($id), new GenerateCsv($id), new UploadS3($id) ])->onQueue('exports')->dispatch();

// Batch có then/catch/finally linh hoạt
$batch = Bus::batch([ new IndexChunk($a), new IndexChunk($b), new IndexChunk($c) ])
  ->onQueue('indexing')
  ->allowFailures()   // không fail cả batch khi 1 job fail
  ->then(fn($b)=> Log::info('done', ['ok'=>$b->processedJobs()]) )
  ->catch(fn($b,$e)=> Log::error('batch_fail',['e'=>$e->getMessage()]) )
  ->finally(fn($b)=> /* cleanup */ )
  ->dispatch();
```

### 4) Queue priorities & Horizon
```php
// .env
QUEUE_CONNECTION=redis
// horizon.php: define queues với trọng số
'queues' => [
  'high' => [ 'connection' => 'redis', 'queue' => ['high','default'], 'balance' => 'auto', 'maxProcesses' => 10 ],
  'low'  => [ 'connection' => 'redis', 'queue' => ['low'], 'maxProcesses' => 2 ],
];
// Dispatch
JobA::dispatch()->onQueue('high');
JobB::dispatch()->onQueue('low');
```
- **Horizon**: xem throughput, thời gian chờ, tỉ lệ lỗi; đặt alert qua Slack/email.  
- **DLQ**: ghi nhận job thất bại vĩnh viễn → đưa vào bảng/queue “dead‑letter” để xử lý thủ công.

### 5) Testing nâng cao
```php
public function test_rate_limited_job_is_released()
{
  RateLimiter::for('sync-company', fn()=> Limit::perMinute(1)->by('k'));
  Queue::fake();
  dispatch(new SyncCompanyJob(1));
  Queue::assertPushed(SyncCompanyJob::class);
}
```

---

## Frontend (FE)

> **Mục tiêu phỏng vấn:** phân tách cập nhật đắt/chậm, giữ UI tương tác mượt; đo đạc bằng Web Vitals + Profiler.

### 1) useTransition — tách cập nhật chậm
```tsx
const [isPending, startTransition] = React.useTransition();
function onSearch(q: string){
  startTransition(()=> setQuery(q)); // render kết quả có thể chậm, nhưng input vẫn mượt
}
return <>
  <input onChange={e=> onSearch(e.target.value)} />
  {isPending && <span>Updating…</span>}
</>;
```

### 2) useDeferredValue — hoãn giá trị nặng
```tsx
const deferred = React.useDeferredValue(query, { timeoutMs: 400 });
const filtered = React.useMemo(()=> heavyFilter(items, deferred), [items, deferred]);
```

### 3) Web Vitals + Profiler (đo thay vì đoán)
```ts
import { onCLS, onLCP, onINP } from 'web-vitals';
onCLS(console.log); onLCP(console.log); onINP(console.log);
```
- Dùng Profiler để xác định component *re-render* nhiều → thêm memoization đúng chỗ.  
- Tránh *layout shift*: đặt kích thước ảnh, dùng `loading="lazy"`, preload font đúng cách.

### 4) Bundle strategy
- Phân tách theo **route** và **screen** nhiều dữ liệu.  
- Prefetch *có điều kiện* (hover/idle) thay vì mọi thứ.  
- Phân tích bundle (ví dụ `source-map-explorer`) để xoá dependency thừa.

---

## English for Interview (skills only)

**Q&A (Advanced Queues & Perf)**
- *How do you prevent duplicate jobs under load?* — Job middleware `WithoutOverlapping` hoặc cache locks theo business key; ensure idempotency inside `handle()`.
- *When would you prioritize queues?* — Critical paths (notifications, payments) vào `high`; batch/indexing vào `low` để không chặn tác vụ quan trọng.
- *How do you keep typing smooth during heavy renders?* — `useTransition`/`useDeferredValue` to separate urgent vs non‑urgent updates; profile and split work.

---

### Query Optimization (Supplement)
> Chủ đề mới: **Index Condition Pushdown (ICP)**, **Leading wildcard pitfalls**, **tmp_table_size & memory tmp tables**, **ONLY_FULL_GROUP_BY** & grouping chính xác.

**Index Condition Pushdown (ICP)**
- Bật mặc định trong 8.0; giúp filter sớm tại cấp index → giảm I/O.
```sql
EXPLAIN SELECT * FROM posts WHERE author_id=? AND title LIKE 'abc%';
-- Chú ý Extra: Using index condition
```

**Leading wildcard**
```sql
-- Tránh LIKE '%term' (không sargable). Dùng FULLTEXT / ngram / reverse index nếu cần hậu tố.
```

**Temporary tables**
- Tăng `tmp_table_size`/`max_heap_table_size` hợp lý để hạn chế spill sang disk; nhưng phải đo và cân nhắc RAM.

**ONLY_FULL_GROUP_BY**
- Khi bật, đảm bảo cột `SELECT` phù hợp với `GROUP BY` (hoặc dùng aggregates/ANY_VALUE) để tránh lỗi & kế hoạch sai.

---

### Activity
- Thêm middleware **RateLimited** + **WithoutOverlapping** cho 1 job thực tế.
- Cấu hình **Horizon** 2 hàng đợi `high`/`low`; thử dispatch và quan sát throughput.
- FE: demo `useTransition` vs không dùng (gõ nhanh + filter nặng).

### Practice Tasks
- Viết test xác nhận rate limit/overlap chặn chạy song song.
- Tạo biểu đồ Web Vitals (log) và kết luận 2 cải tiến có số đo.
- (Supplement) Chạy 1 query có/không ICP, ghi lại sự khác biệt `EXPLAIN`.

### Acceptance Criteria
- Job không chạy trùng; throughput ổn khi giới hạn.
- UI mượt ở nhập liệu; bundle giảm kích thước sau split hợp lý.
- Có ghi chú tối ưu query (ICP/temporary/only_full_group_by).

### Docs & References
- Job Middleware: https://laravel.com/docs/queues#job-middleware
- Laravel Horizon: https://laravel.com/docs/horizon
- React useTransition/useDeferredValue: https://react.dev/reference/react
- Web Vitals: https://web.dev/vitals/
- MySQL ICP: https://dev.mysql.com/doc/refman/8.0/en/index-condition-pushdown-optimization.html
