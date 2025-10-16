# Day 03 — Queue / Retry / DLQ (Laravel) | React useMemo & useCallback

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (Laravel)**
- **Queue basics**: dispatch job, xử lý async (không block request).
- **Retry strategy**: `tries`, `backoff`, handle exception.
- **Dead Letter Queue (DLQ)**: ghi lại job fail để dễ debug, tránh mất dữ liệu.
- **Event → Job flow**: tách sự kiện business khỏi tác vụ nặng.
- **MySQL**: lock row bằng `select for update`, idempotency key khi queue chạy song song.
- **Design Pattern**: **Strategy** (ví dụ job xử lý nhiều provider khác nhau).

**Frontend (React)**
- **useMemo**: tối ưu tính toán nặng, derived state.
- **useCallback**: giữ hàm ổn định, tránh re-render không cần.
- Ví dụ: danh sách item filter/search có memoized list; callback ổn định để pass xuống component con.

**Deliverables**
- Laravel job `SendReportJob` với retry & DLQ logging.
- React list filter với `useMemo`, stable button handler với `useCallback`.

---
- **Composite/Covering index**: tối ưu filter + sort, `Using index` cho select phủ chỉ mục.
- **MySQL Histograms (8.0)**: cải thiện cardinality khi không thể tạo index hoặc dữ liệu lệch.
- **TanStack Query**: caching/fetching/staleTime; invalidate sau mutation.
- **English**: **Q&A Set #1** (offset vs keyset, idempotency & retry, khi nào dùng histogram).

## Backend (BE)

### MySQL — Composite / Covering Index & Histograms
**Composite vs Covering**  
- *Composite*: nhiều cột trong một index, tối ưu filter + sort (vd `(author_id, created_at DESC)`).  
- *Covering*: query chỉ cần đọc index (Extra: `Using index`) vì các cột select đều nằm trong index.

**Histograms (MySQL 8.0)**: cải thiện cardinality estimate khi không thể tạo index hoặc dữ liệu phân bố lệch.
```sql
ANALYZE TABLE posts UPDATE HISTOGRAM ON category_id WITH 128 BUCKETS;
-- Xoá histogram
ANALYZE TABLE posts DROP HISTOGRAM ON category_id;
```
Kiểm tra plan trước/sau để thấy optimizer chọn plan tốt hơn.


> **Mục tiêu phỏng vấn:** chứng minh hiểu rõ Queue/Retry/DLQ, cách làm job idempotent, khi nào dùng `backoff`.

**1) Job cơ bản + Retry**
```php
// app/Jobs/SendReportJob.php
class SendReportJob implements ShouldQueue {
  use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

  public $tries = 3; 
  public $backoff = [60, 300, 600]; // retry sau 1’, 5’, 10’

  public function handle(){
    // gọi API external
    $res = Http::post('https://partner/api/report', ['data' => $this->payload]);
    if($res->failed()){
      throw new \Exception("Partner failed");
    }
  }

  public function failed(\Throwable $e){
    // log DLQ
    \Log::error("Job failed", ['id'=>$this->job->getJobId(), 'error'=>$e->getMessage()]);
  }
}
```

**2) Dispatch & Idempotency**
```php
// Controller
SendReportJob::dispatch($payload)->onQueue('reports');

// Idempotency: check key trước khi dispatch
if(Cache::has('job:report:'.$payload['id'])) return;
Cache::put('job:report:'.$payload['id'], true, now()->addMinutes(10));
SendReportJob::dispatch($payload);
```

**3) Strategy Pattern trong Job**
> Ví dụ: report có nhiều provider khác nhau (S3, Partner API). Job chọn strategy theo config.

```php
interface ReportStrategy { public function send(array $data): void; }

class S3ReportStrategy implements ReportStrategy {
  public function send(array $data){ Storage::disk('s3')->put('report.json', json_encode($data)); }
}

class ApiReportStrategy implements ReportStrategy {
  public function send(array $data){ Http::post('https://api/partner', $data); }
}

class SendReportJob implements ShouldQueue {
  public function handle(ReportStrategy $strategy){
    $strategy->send($this->payload);
  }
}
```

---

## Frontend (FE)

### FE — TanStack Query (fetching/caching/stale)
```tsx
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

const qc = useQueryClient();

function usePosts(){
  return useQuery({
    queryKey: ['posts'],
    queryFn: async()=> {
      const r = await fetch('/api/posts');
      if(!r.ok) throw new Error('Network');
      return r.json();
    },
    staleTime: 30_000,
    refetchOnWindowFocus: false
  });
}

const { data, isLoading, error, refetch } = usePosts();

const createPost = useMutation({
  mutationFn: async(body:any)=>{
    const r = await fetch('/api/posts',{ method:'POST', headers:{'Content-Type':'application/json'}, body: JSON.stringify(body)});
    if(!r.ok) throw new Error('Create failed'); return r.json();
  },
  onSuccess: ()=> qc.invalidateQueries({ queryKey:['posts'] })
});
```


> **Mục tiêu phỏng vấn:** biết khi nào dùng memo/callback, giải thích “premature optimization vs cần thiết”.

**1) useMemo cho derived state**
```tsx
const [search, setSearch] = useState('');
const filtered = useMemo(
  () => items.filter(i => i.name.toLowerCase().includes(search.toLowerCase())),
  [items, search]
);
```

**2) useCallback giữ handler ổn định**
```tsx
const onSelect = useCallback((id:number)=>{
  setSelected(id);
}, []);
```

**3) Passing xuống component con**
```tsx
<ItemList items={filtered} onSelect={onSelect} />
```

---

## English for Interview

### English — Q&A Set #1
- **Q:** How do you decide between offset and keyset pagination?  
  **A:** Keyset for large/real-time lists (stable & fast), offset for small admin views or when random access needed.
- **Q:** How do retries interact with idempotency?  
  **A:** Client/server retries are safe when create endpoints use idempotency keys and writes are designed to be safe to reapply.
- **Q:** When would you add a histogram instead of another index?  
  **A:** When a column is highly skewed and not frequently filtered directly, histograms can guide the optimizer without adding write overhead of an index.
 (skills only)

**Technical Q&A**
1) *How do you make queue jobs idempotent?*  
   **Frame**: use idempotency key (cache/db lock), skip duplicate dispatch, safe DB writes.
2) *How do you design a retry strategy?*  
   **Frame**: exponential backoff, limit tries, DLQ logging.
3) *When to use useMemo/useCallback?*  
   **Frame**: only when heavy computation or stable prop needed; otherwise skip (avoid premature optimization).

**Behavioral (S.T.A.R.)**
- *Handling production job storm*: scaled workers, added DLQ + idempotency, then replayed safely.

---

### Activity
**Backend (Laravel)**
- **Queue basics**: dispatch job, xử lý async (không block request).
- **Retry strategy**: `tries`, `backoff`, handle exception.
- **Dead Letter Queue (DLQ)**: ghi lại job fail để dễ debug, tránh mất dữ liệu.
- **Event → Job flow**: tách sự kiện business khỏi tác vụ nặng.
- **MySQL**: lock row bằng `select for update`, idempotency key khi queue chạy song song.
- Implement BE endpoints and run database migration/index creation
- Verify EXPLAIN plan and capture a note/screenshot
- Build FE screen and verify loading/error/empty states
- Write 1 short interview takeaway (2–3 sentences)

### Practice Tasks
- Viết `SendReportJob` với retry + DLQ log.
- Controller dispatch có cache-based idempotency key.
- React filter list dùng `useMemo`, stable onClick handler bằng `useCallback`.

### Acceptance Criteria
- Job retry max 3 lần, fail log ra DLQ.
- Không job duplicate khi gửi cùng payload.
- FE filter chạy nhanh, không re-render dư thừa.

### Docs & References
- Laravel Queues: https://laravel.com/docs/queues
- Laravel Jobs & Failures: https://laravel.com/docs/queues#dealing-with-failed-jobs
- React useMemo/useCallback: https://react.dev/learn
