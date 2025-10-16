# Day 22 — Company/Department — End-to-end

_Generated on 2025-10-01 03:37._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### BE — REST API cho Company
```php
// routes/api.php
Route::apiResource('companys', CompanyController::class);

// App/Http/Controllers/CompanyController.php
class CompanyController extends Controller {
  public function index(Request $r){
    $q = Company::query()
      ->when($r->q, fn($q,$v)=>$q->where('title','like', "%$v%"))   // search
      ->when($r->from, fn($q,$v)=>$q->whereDate('created_at','>=',$v)) // filter
      ->orderByDesc('created_at');
    return CompanyResource::collection($q->paginate($r->integer('per_page',20)));
  }
  public function store(StoreCompanyRequest $req){
    $model = Company::create($req->validated());
    return (new CompanyResource($model))->response()->setStatusCode(201);
  }
  public function show(Company $model){ return new CompanyResource($model->load('departments')); }
}

// App/Http/Requests/StoreCompanyRequest.php
class StoreCompanyRequest extends FormRequest {
  public function rules(): array {
    return [
      'title' => 'required|string|max:255',
      'department_id' => 'nullable|integer|exists:departments,id',
      'body' => 'required|string'
    ];
  }
}
```

#### BE — Eloquent nâng cao (Company ⇄ Department)
```php
class Company extends Model {
  public function departments(){ return $this->hasMany(Department::class); }
  public function scopeOfOwner($q, $userId){ return $q->where('user_id',$userId); }
}

// Eager loading có constraint
$rows = Company::with(['departments' => fn($q)=>$q->latest()->limit(3)])
  ->ofOwner($ownerId)
  ->when(request('q'), fn($q,$v)=>$q->whereFullText('title', $v))
  ->paginate(20);

// Subquery select: latest related timestamp
$rows = Company::query()->addSelect(['latest_department_at' => Department::select('created_at')
  ->whereColumn('departments.company_id', 'companys.id')->latest()->limit(1)])->get();

// withCount nhiều quan hệ
$rows = Company::withCount(['departments'])->orderByDesc('departments_count')->limit(50)->get();
```

#### BE — Queue + Outbox cho Company
```php
DB::transaction(function() {
  $company = Company::create([...]);
  Outbox::create(['event'=>'CompanyCreated','payload'=>json_encode($company)]);
});

// Worker: gửi event an toàn (retries + DLQ)
class ForwardOutbox implements ShouldQueue {
  public $tries=5; public $backoff=[60,180,600];
  public function handle(){ /* publish to broker */ }
}
```
**Lưu ý**: đảm bảo consumer idempotent; dùng khoá tự nhiên hoặc bảng idempotency.


#### BE — Cache + Redis Lock (Company recompute)
```php
$value = Cache::remember('stats:company:daily', now()->addMinutes(15), function(){
  return app(ComputeCompanyStats::class)->run();
});

$result = Cache::lock('recompute:company', 60)->block(10, function(){
  return app(ComputeCompanyStats::class)->runHeavy();
});
```
**Note**: TTL hợp lý; tránh stampede bằng lock + early recompute.


#### MySQL — Index/EXPLAIN cho Company
```sql
CREATE INDEX idx_companys_owner_created ON companys(user_id, created_at DESC);

EXPLAIN SELECT id, title, created_at
FROM companys
WHERE user_id = ?
ORDER BY created_at DESC
LIMIT 20;
```
**Tip**: chỉ select cột cần; dùng covering index để giảm IO.


#### Design Pattern — Unit of Work (Company)
```php
DB::transaction(function(){ /* group multiple writes for Company */ });
```

## Frontend (FE)

#### FE — Form Company (RHF + Zod)
```tsx
import { useForm } from 'react-hook-form';
type Form = { title:string; body:string; };
export default function CompanyForm(){
  const { register, handleSubmit, formState:{errors} } = useForm<Form>();
  const onSubmit = (d:Form)=> fetch('/api/companys',{method:'POST',headers:{'Content-Type':'application/json'},body:JSON.stringify(d)});

  return (<form onSubmit={handleSubmit(onSubmit)} className="space-y-3 max-w-md">
    <input className="border p-2 w-full" placeholder="Company title" {...register('title',{required:true, minLength:3})}/>
    {errors.title && <span className="text-red-600 text-sm">Title too short</span>}
    <textarea className="border p-2 w-full" placeholder="Body" {...register('body',{required:true})}/>
    <button className="border rounded-xl px-4 py-2">Save</button>
  </form>);
}
```

#### FE — List Company (React Query + Infinite)
```tsx
import { useInfiniteQuery } from '@tanstack/react-query';
const fetchPage = async ({pageParam=1})=> (await fetch(`/api/companys?page=${pageParam}`)).json();
export default function CompanyList(){
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteQuery({
    queryKey:['companys'], queryFn: fetchPage, getNextPageParam: (last)=> last.next_page ?? false
  });
  return (<div>
    {data?.pages.flatMap(p=>p.data).map((it:any)=><div key={it.id}>{it.title}</div>)}
    <button disabled={!hasNextPage||isFetchingNextPage} onClick={()=>fetchNextPage()}>
      {isFetchingNextPage? 'Loading…':'Load more'}
    </button>
  </div>);
}
```

#### FE — Modal A11y cho Company
```tsx
// Focus trap + ESC close applied; see implementation from previous days adjusted for Company
```

### English (Interview)
- **Self-intro (45s)**: *Hi, I'm Minh... Today I worked on company/department — end-to-end with a focus on reliability and performance.*
- **Vocab**: debounce, filesort, idempotency, indexing, observability, optimistic UI, sanitization, throughput
- **Practice**: 1 STAR story + 2 technical answers recorded.


### Interview Questions
- Company/Department: mô tả quan hệ, chiến lược tránh N+1?
- Idempotency: bạn xử lý duplicate request thế nào ở tầng DB/API?
- Khi nào phân mảnh (partition) bảng? Ràng buộc với PK/unique?
- Tình huống cần Outbox? So sánh với trực tiếp publish.


### Practice Tasks
- Build create/list UI for Company + secure API; write 2 tests.

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
- API Company: đúng schema (201/204/404/422); lỗi chuẩn hoá.
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