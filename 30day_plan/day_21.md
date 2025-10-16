# Day 21 — Setting/Preference — End-to-end

_Generated on 2025-10-01 03:37._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### BE — REST API cho Setting
```php
// routes/api.php
Route::apiResource('settings', SettingController::class);

// App/Http/Controllers/SettingController.php
class SettingController extends Controller {
  public function index(Request $r){
    $q = Setting::query()
      ->when($r->q, fn($q,$v)=>$q->where('title','like', "%$v%"))   // search
      ->when($r->from, fn($q,$v)=>$q->whereDate('created_at','>=',$v)) // filter
      ->orderByDesc('created_at');
    return SettingResource::collection($q->paginate($r->integer('per_page',20)));
  }
  public function store(StoreSettingRequest $req){
    $model = Setting::create($req->validated());
    return (new SettingResource($model))->response()->setStatusCode(201);
  }
  public function show(Setting $model){ return new SettingResource($model->load('preferences')); }
}

// App/Http/Requests/StoreSettingRequest.php
class StoreSettingRequest extends FormRequest {
  public function rules(): array {
    return [
      'title' => 'required|string|max:255',
      'preference_id' => 'nullable|integer|exists:preferences,id',
      'body' => 'required|string'
    ];
  }
}
```

#### BE — Eloquent nâng cao (Setting ⇄ Preference)
```php
class Setting extends Model {
  public function preferences(){ return $this->hasMany(Preference::class); }
  public function scopeOfOwner($q, $userId){ return $q->where('user_id',$userId); }
}

// Eager loading có constraint
$rows = Setting::with(['preferences' => fn($q)=>$q->latest()->limit(3)])
  ->ofOwner($ownerId)
  ->when(request('q'), fn($q,$v)=>$q->whereFullText('title', $v))
  ->paginate(20);

// Subquery select: latest related timestamp
$rows = Setting::query()->addSelect(['latest_preference_at' => Preference::select('created_at')
  ->whereColumn('preferences.setting_id', 'settings.id')->latest()->limit(1)])->get();

// withCount nhiều quan hệ
$rows = Setting::withCount(['preferences'])->orderByDesc('preferences_count')->limit(50)->get();
```

#### BE — Queue + Outbox cho Setting
```php
DB::transaction(function() {
  $setting = Setting::create([...]);
  Outbox::create(['event'=>'SettingCreated','payload'=>json_encode($setting)]);
});

// Worker: gửi event an toàn (retries + DLQ)
class ForwardOutbox implements ShouldQueue {
  public $tries=5; public $backoff=[60,180,600];
  public function handle(){ /* publish to broker */ }
}
```
**Lưu ý**: đảm bảo consumer idempotent; dùng khoá tự nhiên hoặc bảng idempotency.


#### BE — Cache + Redis Lock (Setting recompute)
```php
$value = Cache::remember('stats:setting:daily', now()->addMinutes(15), function(){
  return app(ComputeSettingStats::class)->run();
});

$result = Cache::lock('recompute:setting', 60)->block(10, function(){
  return app(ComputeSettingStats::class)->runHeavy();
});
```
**Note**: TTL hợp lý; tránh stampede bằng lock + early recompute.


#### MySQL — Index/EXPLAIN cho Setting
```sql
CREATE INDEX idx_settings_owner_created ON settings(user_id, created_at DESC);

EXPLAIN SELECT id, title, created_at
FROM settings
WHERE user_id = ?
ORDER BY created_at DESC
LIMIT 20;
```
**Tip**: chỉ select cột cần; dùng covering index để giảm IO.


#### MySQL — Partition theo ngày (Setting)
```sql
ALTER TABLE settings
PARTITION BY RANGE (TO_DAYS(created_at)) (
  PARTITION p2025 VALUES LESS THAN (TO_DAYS('2026-01-01'))
);
```
**Cảnh báo**: cột partition cần nằm trong PK/unique key (MySQL 8).


#### Design Pattern — Repository (Setting)
```php
interface SettingRepo { public function list(array $f, int $pp=20); }
class EloquentSettingRepo implements SettingRepo {
  public function list(array $f, int $pp=20){
    return Setting::query()->when($f['q']??null, fn($q,$v)=>$q->where('title','like',"%$v%"))->paginate($pp);
  }
}
```

## Frontend (FE)

#### FE — Form Setting (RHF + Zod)
```tsx
import { useForm } from 'react-hook-form';
type Form = { title:string; body:string; };
export default function SettingForm(){
  const { register, handleSubmit, formState:{errors} } = useForm<Form>();
  const onSubmit = (d:Form)=> fetch('/api/settings',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(d)});

  return (<form onSubmit={handleSubmit(onSubmit)} className="space-y-3 max-w-md">
    <input className="border p-2 w-full" placeholder="Setting title" {...register('title',{required:true, minLength:3})}/>
    {errors.title && <span className="text-red-600 text-sm">Title too short</span>}
    <textarea className="border p-2 w-full" placeholder="Body" {...register('body',{required:true})}/>
    <button className="border rounded-xl px-4 py-2">Save</button>
  </form>);
}
```

#### FE — List Setting (React Query + Infinite)
```tsx
import { useInfiniteQuery } from '@tanstack/react-query';
const fetchPage = async ({pageParam=1})=> (await fetch(`/api/settings?page=${pageParam}`)).json();
export default function SettingList(){
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({
    queryKey:['settings'], queryFn: fetchPage, getNextPageParam: (last)=> last.next_page ?? false
  });
  return (<div>
    {data?.pages.flatMap(p=>p.data).map((it:any)=><div key={it.id}>{it.title}</div>)}
    <button disabled={!hasNextPage||isFetchingNextPage} onClick={()=>fetchNextPage()}>
      {isFetchingNextPage? 'Loading…':'Load more'}
    </button>
  </div>);
}
```

#### FE — Modal A11y cho Setting
```tsx
// Focus trap + ESC close applied; see implementation from previous days adjusted for Setting
```

### English (Interview)
- **Self-intro (45s)**: *Hi, I'm Minh... Today I worked on setting/preference — end-to-end with a focus on reliability and performance.*
- **Vocab**: backpressure, debounce, idempotency, indexing, latency, observability, sanitization, scalability
- **Practice**: 1 STAR story + 2 technical answers recorded.


### Interview Questions
- Setting/Preference: mô tả quan hệ, chiến lược tránh N+1?
- Idempotency: bạn xử lý duplicate request thế nào ở tầng DB/API?
- Khi nào phân mảnh (partition) bảng? Ràng buộc với PK/unique?
- Tình huống cần Outbox? So sánh với trực tiếp publish.


### Practice Tasks
- Build create/list UI for Setting + secure API; write 2 tests.

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
- API Setting: đúng schema (201/204/404/422); lỗi chuẩn hoá.
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