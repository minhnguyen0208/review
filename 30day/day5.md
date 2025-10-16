# Day 05 — Caching & Redis

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (Laravel)**
- Caching & Redis
**Frontend (React)**
- **Mutation flow** (create/update) + mapping lỗi 422 vào form.
- **(Tuỳ chọn)** TanStack Query cho invalidate sau khi tạo bản ghi.

---

## Backend (BE)
> **Mục tiêu phỏng vấn:** chứng minh kiểm soát cache/invalidation, lỗi chuẩn hoá, log có ngữ cảnh.
**Caching & Cache Invalidation — trước khi xem code**  
Vì sao: giảm tải DB, nhưng cần chiến lược invalidation rõ ràng (time‑based, key‑based, event‑based).
```php
// put/get with TTL
$stats = Cache::remember('stats:daily', now()->addMinutes(15), fn()=> computeStats());

// tag-based (if using a taggable driver like redis)
Cache::tags(['posts'])->remember('posts:popular', 600, fn()=> Post::popular()->take(10)->get());

// invalidate on write
Cache::tags(['posts']).flush(); // hoặc xoá theo key cụ thể
```
**Redis Lock (avoid duplicate work)**
```php
$result = Cache::lock('refresh_token_'.$companyId, 30)->block(10, function() use($refreshToken){
  return Http::asForm()->retry(2, 250)->post(config('sso.token_url'), [
    'grant_type'=>'refresh_token','refresh_token'=>$refreshToken
  ])->json();
});
```
**Error Envelope chuẩn hóa**
```php
// app/Exceptions/Handler.php (render)
return response()->json([
  'code'    => $status,
  'message' => $e->getMessage() ?: Response::$statusTexts[$status],
  'errors'  => method_exists($e, 'errors') ? $e->errors() : null,
], $status);
```
**Logging & Context**
```php
Log::withContext(['request_id'=>Str::uuid(), 'user'=>optional(auth()->user())->id]);
Log::info('order.created', ['order_id'=>$order->id, 'total'=>$order->total]);
```
**PHPUnit Feature Test (smoke)**
```php
public function test_can_list_posts(){
  $res = $this->getJson('/api/posts');
  $res->assertOk()->assertJsonStructure(['data']);
}
```

---

## Frontend (FE)
> **Mục tiêu phỏng vấn:** thao tác form/mutation, hiển thị lỗi field, invalidate dữ liệu hợp lý.
**React — Mutation flow (POST) với error mapping**
```tsx
import { useState } from 'react';

const [loading, setLoading] = useState(false);
const [errors, setErrors] = useState<Record<string,string[]>>({});

async function createPost(body:any){
  setLoading(true); setErrors({});
  const r = await fetch('/api/posts', { method:'POST', headers:{'Content-Type':'application/json'}, body: JSON.stringify(body) });
  if (r.status === 422){ const j = await r.json(); setErrors(j.errors || {}); }
  else if (!r.ok){ /* toast from {code,message} */ }
  setLoading(false);
}
```
**Form a11y & field errors**
```tsx
<input name="title" aria-invalid={!!errors.title} />
{errors.title && <p role="alert">{errors.title[0]}</p>}
```

---

## English for Interview (skills only)
**English — Q&A (Set for Day 5)**
- *How do you decide what to cache and for how long?* — Identify hot paths and data volatility; pick TTL or event‑driven invalidation; always plan for cache stampede.
- *How do you standardize errors for a React team?* — A JSON envelope `{code,message,errors[]}` and clear 4xx/5xx mapping; samples in API docs.
- *How do you verify reliability?* — Feature tests for happy path; contract tests for error envelopes; add request IDs in logs.


---

### Activity
- BE 2–3h: implement 1 feature or refactor
- 2 LeetCode (E/M)
- write unit tests
- summarize learnings. | FE 2h: extend a React component/page
- add tests
- check a11y
- document decisions. | English 1–1.5h: shadow 15‑20 min
- mock interview Q&A
- record & review. | Queues/Cache: jobs, retries, backoff, RateLimiter
- measure throughput
- log timings and memory.

### Practice Tasks
- Áp dụng cache cho endpoint nóng; thêm invalidation hợp lý.
- Chuẩn hoá error envelope trong `Handler::render`.
- FE: form tạo/sửa với mapping lỗi 422; (tuỳ chọn) mutation invalidate bằng React Query.

### Acceptance Criteria
- Cache có hiệu lực, invalidation đúng; không stale nguy hiểm.
- Lỗi trả về `{code,message,errors[]}`; FE hiển thị field errors rõ ràng.
- Log có `request_id` và thông tin bối cảnh (user, endpoint).

### Docs & References
- Laravel Cache: https://laravel.com/docs/cache
- Laravel Errors/Exceptions: https://laravel.com/docs/errors
- Laravel Logging: https://laravel.com/docs/logging
- React (Forms & Effects): https://react.dev/learn
- TanStack Query: https://tanstack.com/query/latest