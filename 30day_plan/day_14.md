# Day 14 — Batch/Job — End-to-end

_Generated on 2025-10-01 03:37._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### BE — REST API cho Batch
```php
// routes/api.php
Route::apiResource('batchs', BatchController::class);

// App/Http/Controllers/BatchController.php
class BatchController extends Controller {
  public function index(Request $r){
    $q = Batch::query()
      ->when($r->q, fn($q,$v)=>$q->where('title','like', "%$v%"))   // search
      ->when($r->from, fn($q,$v)=>$q->whereDate('created_at','>=',$v)) // filter
      ->orderByDesc('created_at');
    return BatchResource::collection($q->paginate($r->integer('per_page',20)));
  }
  public function store(StoreBatchRequest $req){
    $model = Batch::create($req->validated());
    return (new BatchResource($model))->response()->setStatusCode(201);
  }
  public function show(Batch $model){ return new BatchResource($model->load('jobs')); }
}

// App/Http/Requests/StoreBatchRequest.php
class StoreBatchRequest extends FormRequest {
  public function rules(): array {
    return [
      'title' => 'required|string|max:255',
      'job_id' => 'nullable|integer|exists:jobs,id',
      'body' => 'required|string'
    ];
  }
}
```

#### BE — Eloquent nâng cao (Batch ⇄ Job)
```php
class Batch extends Model {
  public function jobs(){ return $this->hasMany(Job::class); }
  public function scopeOfOwner($q, $userId){ return $q->where('user_id',$userId); }
}

// Eager loading có constraint
$rows = Batch::with(['jobs' => fn($q)=>$q->latest()->limit(3)])
  ->ofOwner($ownerId)
  ->when(request('q'), fn($q,$v)=>$q->whereFullText('title', $v))
  ->paginate(20);

// Subquery select: latest related timestamp
$rows = Batch::query()->addSelect(['latest_job_at' => Job::select('created_at')
  ->whereColumn('jobs.batch_id', 'batchs.id')->latest()->limit(1)])->get();

// withCount nhiều quan hệ
$rows = Batch::withCount(['jobs'])->orderByDesc('jobs_count')->limit(50)->get();
```

#### BE — Queue + Outbox cho Batch
```php
DB::transaction(function() {
  $batch = Batch::create([...]);
  Outbox::create(['event'=>'BatchCreated','payload'=>json_encode($batch)]);
});

// Worker: gửi event an toàn (retries + DLQ)
class ForwardOutbox implements ShouldQueue {
  public $tries=5; public $backoff=[60,180,600];
  public function handle(){ /* publish to broker */ }
}
```
**Lưu ý**: đảm bảo consumer idempotent; dùng khoá tự nhiên hoặc bảng idempotency.


#### BE — Cache + Redis Lock (Batch recompute)
```php
$value = Cache::remember('stats:batch:daily', now()->addMinutes(15), function(){
  return app(ComputeBatchStats::class)->run();
});

$result = Cache::lock('recompute:batch', 60)->block(10, function(){
  return app(ComputeBatchStats::class)->runHeavy();
});
```
**Note**: TTL hợp lý; tránh stampede bằng lock + early recompute.


#### MySQL — Index/EXPLAIN cho Batch
```sql
CREATE INDEX idx_batchs_owner_created ON batchs(user_id, created_at DESC);

EXPLAIN SELECT id, title, created_at
FROM batchs
WHERE user_id = ?
ORDER BY created_at DESC
LIMIT 20;
```
**Tip**: chỉ select cột cần; dùng covering index để giảm IO.


#### Design Pattern — Strategy (Batch)
```php
interface BatchPricing { public function price(float $base): float; }
class DefaultBatchPricing implements BatchPricing { public function price($b){return $b;} }
class PromoBatchPricing implements BatchPricing { public function price($b){return $b*0.9;} }
```

## Frontend (FE)

#### FE — Form Batch (RHF + Zod)
```tsx
import { useForm } from 'react-hook-form';
type Form = { title:string; body:string; };
export default function BatchForm(){
  const { register, handleSubmit, formState:{errors} } = useForm<Form>();
  const onSubmit = (d:Form)=> fetch('/api/batchs',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(d)});

  return (<form onSubmit={handleSubmit(onSubmit)} className="space-y-3 max-w-md">
    <input className="border p-2 w-full" placeholder="Batch title" {...register('title',{required:true, minLength:3})}/>
    {errors.title && <span className="text-red-600 text-sm">Title too short</span>}
    <textarea className="border p-2 w-full" placeholder="Body" {...register('body',{required:true})}/>
    <button className="border rounded-xl px-4 py-2">Save</button>
  </form>);
}
```

#### FE — List Batch (React Query + Infinite)
```tsx
import { useInfiniteQuery } from '@tanstack/react-query';
const fetchPage = async ({pageParam=1})=> (await fetch(`/api/batchs?page=${pageParam}`)).json();
export default function BatchList(){
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({
    queryKey:['batchs'], queryFn: fetchPage, getNextPageParam: (last)=> last.next_page ?? false
  });
  return (<div>
    {data?.pages.flatMap(p=>p.data).map((it:any)=><div key={it.id}>{it.title}</div>)}
    <button disabled={!hasNextPage||isFetchingNextPage} onClick={()=>fetchNextPage()}>
      {isFetchingNextPage? 'Loading…':'Load more'}
    </button>
  </div>);
}
```

#### FE — Modal A11y cho Batch
```tsx
// Focus trap + ESC close applied; see implementation from previous days adjusted for Batch
```

### English (Interview)
- **Self-intro (45s)**: *Hi, I'm Minh... Today I worked on batch/job — end-to-end with a focus on reliability and performance.*
- **Vocab**: authorization, debounce, idempotency, memoization, reliability, sanitization, scalability, throughput
- **Practice**: 1 STAR story + 2 technical answers recorded.


### Interview Questions
- Batch/Job: mô tả quan hệ, chiến lược tránh N+1?
- Idempotency: bạn xử lý duplicate request thế nào ở tầng DB/API?
- Khi nào phân mảnh (partition) bảng? Ràng buộc với PK/unique?
- Tình huống cần Outbox? So sánh với trực tiếp publish.


### Practice Tasks
- Build create/list UI for Batch + secure API; write 2 tests.

### Docs & References
- [Laravel](https://laravel.com/docs)
- [Eloquent](https://laravel.com/docs/eloquent)
- [Validation](https://laravel.com/docs/validation)
- [Queues](https://laravel.com/docs/queues)
- [Cache](https://laravel.com/docs/cache)
- [Database](https://dev.mysql.com/doc/)
- [React](https://react.dev/learn)
- [React Router](https://reactrouter.com/en/main)
- [React Query](https://tanstack.com/query/latest)
- [TypeScript](https://www.typescriptlang.org/docs/)
- [Tailwind](https://tailwindcss.com/docs)
- [Jest](https://jestjs.io/docs/getting-started)
- [Playwright](https://playwright.dev/docs/intro)
- [Docker](https://docs.docker.com/)
- [OWASP](https://owasp.org/www-project-top-ten/)

### Acceptance Criteria
- API Batch: đúng schema (201/204/404/422); lỗi chuẩn hoá.
- FE: có loading/error rõ ràng; infinite scroll hoặc form validate chạy đúng.
- MySQL: EXPLAIN không còn ALL; có index phù hợp.
- Có 1 Feature test (BE) + 1 test (RTL/Playwright) cho FE.


### Checklist
- [ ] BE + FE end-to-end
- [ ] 1–2 tests pass
- [ ] Đo EXPLAIN/metrics, ghi chú tối ưu
- [ ] Luyện EN: vocab + 2 câu trả lời


### Reflection
- Điều gì khó nhất? Bạn tối ưu thế nào?