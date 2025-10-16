# Day 20 — Alert/Rule — End-to-end

_Generated on 2025-10-01 03:37._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### BE — REST API cho Alert
```php
// routes/api.php
Route::apiResource('alerts', AlertController::class);

// App/Http/Controllers/AlertController.php
class AlertController extends Controller {
  public function index(Request $r){
    $q = Alert::query()
      ->when($r->q, fn($q,$v)=>$q->where('title','like', "%$v%"))   // search
      ->when($r->from, fn($q,$v)=>$q->whereDate('created_at','>=',$v)) // filter
      ->orderByDesc('created_at');
    return AlertResource::collection($q->paginate($r->integer('per_page',20)));
  }
  public function store(StoreAlertRequest $req){
    $model = Alert::create($req->validated());
    return (new AlertResource($model))->response()->setStatusCode(201);
  }
  public function show(Alert $model){ return new AlertResource($model->load('rules')); }
}

// App/Http/Requests/StoreAlertRequest.php
class StoreAlertRequest extends FormRequest {
  public function rules(): array {
    return [
      'title' => 'required|string|max:255',
      'rule_id' => 'nullable|integer|exists:rules,id',
      'body' => 'required|string'
    ];
  }
}
```

#### BE — Eloquent nâng cao (Alert ⇄ Rule)
```php
class Alert extends Model {
  public function rules(){ return $this->hasMany(Rule::class); }
  public function scopeOfOwner($q, $userId){ return $q->where('user_id',$userId); }
}

// Eager loading có constraint
$rows = Alert::with(['rules' => fn($q)=>$q->latest()->limit(3)])
  ->ofOwner($ownerId)
  ->when(request('q'), fn($q,$v)=>$q->whereFullText('title', $v))
  ->paginate(20);

// Subquery select: latest related timestamp
$rows = Alert::query()->addSelect(['latest_rule_at' => Rule::select('created_at')
  ->whereColumn('rules.alert_id', 'alerts.id')->latest()->limit(1)])->get();

// withCount nhiều quan hệ
$rows = Alert::withCount(['rules'])->orderByDesc('rules_count')->limit(50)->get();
```

#### BE — Queue + Outbox cho Alert
```php
DB::transaction(function() {
  $alert = Alert::create([...]);
  Outbox::create(['event'=>'AlertCreated','payload'=>json_encode($alert)]);
});

// Worker: gửi event an toàn (retries + DLQ)
class ForwardOutbox implements ShouldQueue {
  public $tries=5; public $backoff=[60,180,600];
  public function handle(){ /* publish to broker */ }
}
```
**Lưu ý**: đảm bảo consumer idempotent; dùng khoá tự nhiên hoặc bảng idempotency.


#### BE — Cache + Redis Lock (Alert recompute)
```php
$value = Cache::remember('stats:alert:daily', now()->addMinutes(15), function(){
  return app(ComputeAlertStats::class)->run();
});

$result = Cache::lock('recompute:alert', 60)->block(10, function(){
  return app(ComputeAlertStats::class)->runHeavy();
});
```
**Note**: TTL hợp lý; tránh stampede bằng lock + early recompute.


#### MySQL — Index/EXPLAIN cho Alert
```sql
CREATE INDEX idx_alerts_owner_created ON alerts(user_id, created_at DESC);

EXPLAIN SELECT id, title, created_at
FROM alerts
WHERE user_id = ?
ORDER BY created_at DESC
LIMIT 20;
```
**Tip**: chỉ select cột cần; dùng covering index để giảm IO.


#### Design Pattern — Strategy (Alert)
```php
interface AlertPricing { public function price(float $base): float; }
class DefaultAlertPricing implements AlertPricing { public function price($b){return $b;} }
class PromoAlertPricing implements AlertPricing { public function price($b){return $b*0.9;} }
```

## Frontend (FE)

#### FE — Form Alert (RHF + Zod)
```tsx
import { useForm } from 'react-hook-form';
type Form = { title:string; body:string; };
export default function AlertForm(){
  const { register, handleSubmit, formState:{errors} } = useForm<Form>();
  const onSubmit = (d:Form)=> fetch('/api/alerts',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(d)});

  return (<form onSubmit={handleSubmit(onSubmit)} className="space-y-3 max-w-md">
    <input className="border p-2 w-full" placeholder="Alert title" {...register('title',{required:true, minLength:3})}/>
    {errors.title && <span className="text-red-600 text-sm">Title too short</span>}
    <textarea className="border p-2 w-full" placeholder="Body" {...register('body',{required:true})}/>
    <button className="border rounded-xl px-4 py-2">Save</button>
  </form>);
}
```

#### FE — List Alert (React Query + Infinite)
```tsx
import { useInfiniteQuery } from '@tanstack/react-query';
const fetchPage = async ({pageParam=1})=> (await fetch(`/api/alerts?page=${pageParam}`)).json();
export default function AlertList(){
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({
    queryKey:['alerts'], queryFn: fetchPage, getNextPageParam: (last)=> last.next_page ?? false
  });
  return (<div>
    {data?.pages.flatMap(p=>p.data).map((it:any)=><div key={it.id}>{it.title}</div>)}
    <button disabled={!hasNextPage||isFetchingNextPage} onClick={()=>fetchNextPage()}>
      {isFetchingNextPage? 'Loading…':'Load more'}
    </button>
  </div>);
}
```

#### FE — Modal A11y cho Alert
```tsx
// Focus trap + ESC close applied; see implementation from previous days adjusted for Alert
```

### English (Interview)
- **Self-intro (45s)**: *Hi, I'm Minh... Today I worked on alert/rule — end-to-end with a focus on reliability and performance.*
- **Vocab**: authorization, backpressure, filesort, idempotency, memoization, partitioning, reliability, throughput
- **Practice**: 1 STAR story + 2 technical answers recorded.


### Interview Questions
- Alert/Rule: mô tả quan hệ, chiến lược tránh N+1?
- Idempotency: bạn xử lý duplicate request thế nào ở tầng DB/API?
- Khi nào phân mảnh (partition) bảng? Ràng buộc với PK/unique?
- Tình huống cần Outbox? So sánh với trực tiếp publish.


### Practice Tasks
- Build create/list UI for Alert + secure API; write 2 tests.

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
- API Alert: đúng schema (201/204/404/422); lỗi chuẩn hoá.
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