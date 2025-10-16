# Day 02 — Auth (Sanctum) + Policies/Gates + Rate Limiting | React useRef & useReducer

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (Laravel)**
- Token-based **Auth (Sanctum)** cho SPA/mobile; bảo vệ route với `auth:sanctum`; endpoint `/me`.  
- **Authorization**: **Policies/Gates** cho rule theo model/action; testable; tách khỏi Controller.  
- **Rate Limiting**: chống abuse theo user/IP; trả 429 có hint retry; nói được tiêu chí “burst vs steady”.

**Frontend (React)**
- **useRef**: tham chiếu mutable không trigger render (focus input, giữ timer, scroll…).  
- **useReducer**: state phức tạp nhiều nhánh (idle/loading/success/error), predictable updates, dễ test.

**Deliverables**
- `/login` (trả token), `/me` (protected), Policy `update-post`, Rate limit cho API.  
- FE: form login auto-focus (`useRef`), flow dùng `useReducer`.

---
- **chunkById** để xử lý dữ liệu lớn ổn định hơn `chunk()`.
- **Keyset pagination** (không OFFSET) cho danh sách lớn/real‑time.
- **CSV export (streamed)**: ghi từng dòng, O(n) thời gian, O(1) bộ nhớ.
- **English**: luyện **Describe Your Project (60–90s)** với số liệu tác động.

## Backend (BE)

### CSV Export — Streamed Response (O(n) memory)
```php
use Symfony\Component\HttpFoundation\StreamedResponse;

Route::get('/reports/export', function () {
  $headers = ['Content-Type' => 'text/csv', 'Content-Disposition' => 'attachment; filename="report.csv"'];
  $callback = function() {
    $out = fopen('php://output', 'w');
    fputcsv($out, ['id','title','created_at']);
    Post::orderBy('id')->chunkById(2000, function($rows) use ($out){
      foreach ($rows as $r) fputcsv($out, [$r->id, $r->title, $r->created_at]);
      fflush($out);
    });
    fclose($out);
  };
  return new StreamedResponse($callback, 200, $headers);
});
```


### Large Dataset — chunkById & Keyset Pagination
**chunkById**: xử lý dữ liệu lớn theo lô, ổn định hơn `chunk()` khi có insert/delete.
```php
Post::where('status','PUBLISHED')
  ->orderBy('id')
  ->chunkById(1000, function($rows){
    foreach($rows as $r){ /* process */ }
  });
```
**Keyset pagination**: phân trang nhanh, không `OFFSET` chậm.
```php
// page sau: WHERE id < last_seen_id ORDER BY id DESC LIMIT 20
$rows = Post::where('id','<',$lastId)->orderByDesc('id')->limit(20)->get();
```


> **Mục tiêu phỏng vấn:** xác thực (token), phân quyền theo model, bảo vệ API bằng rate limit; giải thích rõ ràng sự khác nhau **Authentication vs Authorization**.

**1) Auth (Sanctum) — trước khi xem code**  
Vì sao: SPA cần token đơn giản, secure; middleware `auth:sanctum` + endpoint `/me`.

```php
// Issue personal access token after login
$token = $user->createToken('web')->plainTextToken;

// Protect routes
Route::middleware('auth:sanctum')->get('/me', fn(Illuminate\Http\Request $r) => $r->user());
```

**2) Authorization (Policies/Gates) — trước khi xem code**  
Vì sao: rule theo model/action, centralized & testable, giảm logic ở Controller.

```php
use Illuminate\Support\Facades\Gate;

Gate::define('update-post', fn(User $u, Post $p) => $u->id === $p->author_id);

// Trong Controller
$this->authorize('update-post', $post); // 403 nếu không đủ quyền
```

**3) Rate Limiting — trước khi xem code**  
Vì sao: bảo vệ API khỏi brute-force/abuse; policy per user/IP, hỗ trợ burst.

```php
use Illuminate\Cache\RateLimiting\Limit;
use Illuminate\Support\Facades\RateLimiter;

RateLimiter::for('api', function($request){
  return Limit::perMinute(120)->by(optional($request->user())->id ?: $request->ip());
});
```

**4) Validation cho login — trước khi xem code**  
Vì sao: logic hợp lệ ở request layer; giảm duplication; test đơn giản.

```php
class LoginRequest extends FormRequest {
  public function rules(){
    return ['email'=>'required|email','password'=>'required|string|min:8'];
  }
  public function authorize(){ return true; }
}
```

---

## Frontend (FE)

> **Mục tiêu phỏng vấn:** vận dụng `useRef` cho hành vi DOM, `useReducer` cho state nhiều nhánh; giải thích vì sao dùng reducer thay vì rải rác nhiều `useState`.

**1) useRef — trước khi xem code**  
Vì sao: giữ tham chiếu mutable mà không re-render (focus input khi mount, lưu timer id…).

```tsx
const inputRef = useRef<HTMLInputElement>(null);
useEffect(()=>{ inputRef.current?.focus(); },[]);
```

**2) useReducer — trước khi xem code**  
Vì sao: state machine nhỏ cho form/flow (idle → loading → success/error), predictable & testable.

```tsx
type Action = {type:'start'}|{type:'success', data:any}|{type:'error', message:string};
type State = {status:'idle'|'loading'|'success'|'error', data?:any, message?:string};

function reducer(s:State, a:Action): State {
  switch(a.type){
    case 'start':   return {status:'loading'};
    case 'success': return {status:'success', data:a.data};
    case 'error':   return {status:'error', message:a.message};
    default:        return s;
  }
}
const [state, dispatch] = useReducer(reducer, {status:'idle'});
```

**3) (Tuỳ chọn) React Query cho `/me`**  
Vì sao: quản lý fetching/caching/stale-time gọn hơn `useEffect`.

```tsx
import { useQuery } from '@tanstack/react-query';

async function fetchMe(){ const r = await fetch('/api/me'); if(!r.ok) throw new Error('Unauthorized'); return r.json(); }
const { data, isLoading, error } = useQuery({ queryKey:['me'], queryFn: fetchMe, staleTime: 30_000 });
```

---

## English for Interview

### English — Describe Your Project
- **Template (60–90s)**: Context → Team/role → Problem → Your solution → Impact (numbers) → Lessons.
- **Sample**:  
  *I worked on an internal reporting tool. As the only BE dev, I added keyset pagination and `chunkById` streaming CSV exports. This reduced export time from 8m to 2m and memory stayed under 50MB. I also standardized error envelopes so the React team could render states deterministically.*
 (skills only)

**Technical Q&A**
1) *How do you design token-based auth for SPA?*  
   **Frame**: Sanctum personal tokens, protect with `auth:sanctum`, endpoint `/me`, secure storage strategy, token revocation.  
2) *Authentication vs Authorization?*  
   **Frame**: Identity vs permissions; use **Policies** for model-level checks; Controllers only orchestrate.  
3) *How to throttle abusive clients?*  
   **Frame**: `RateLimiter::for('api')` per user/IP, burst config, return `429` with retry hints, logs/observability.

**Behavioral (S.T.A.R.)**
- *Disagree & commit:* bất đồng về chiến lược auth → thống nhất thử nghiệm POC, đo latency/UX, chốt theo dữ liệu.

---

### Activity
**Backend (Laravel)**
- Token-based **Auth (Sanctum)** cho SPA/mobile; bảo vệ route với `auth:sanctum`; endpoint `/me`.
- **Authorization**: **Policies/Gates** cho rule theo model/action; testable; tách khỏi Controller.
- **Rate Limiting**: chống abuse theo user/IP; trả 429 có hint retry; nói được tiêu chí “burst vs steady”.
**Frontend (React)**
- **useRef**: tham chiếu mutable không trigger render (focus input, giữ timer, scroll…).
- Implement BE endpoints and run database migration/index creation
- Verify EXPLAIN plan and capture a note/screenshot
- Build FE screen and verify loading/error/empty states
- Write 1 short interview takeaway (2–3 sentences)

### Practice Tasks
- Tạo `/login` (trả token), `/me` bảo vệ bởi `auth:sanctum`; thêm Policy `update-post`.  
- FE: auto-focus input bằng `useRef`; quản lý form/login state bằng `useReducer`.  
- Viết 1 feature test cho `/me` (401 → 200 sau khi đính token).

### Acceptance Criteria
- 401/403/429 trả về đúng envelope lỗi; Policy chặn không đúng quyền; rate limit hoạt động.  
- FE không warning; flow state rõ (idle → loading → success/error).

### Docs & References
- Laravel: https://laravel.com/docs  
- Policies & Gates: https://laravel.com/docs/authorization  
- Sanctum: https://laravel.com/docs/sanctum  
- React (Core Hooks): https://react.dev/learn
