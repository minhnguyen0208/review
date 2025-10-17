# Day 15 — Transactions & Concurrency (Deadlocks, Retries, `FOR UPDATE SKIP LOCKED`) | React Hook Form + Zod

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (Laravel + MySQL)**
- **Transaction boundaries**: wrap logic đúng chỗ; tránh giao dịch quá dài.
- **Deadlock handling**: phát hiện, **retry with jitter**, ghi log ngữ cảnh.
- **Pessimistic locking**: `SELECT ... FOR UPDATE` / `SKIP LOCKED` cho hàng đợi công việc.
- **Optimistic locking (phiên bản)**: cột `version` hoặc `updated_at` để phát hiện ghi đè.
- **Outbox vs 2PC**: chọn Outbox (at-least-once) cho event nhất quán.

**Frontend (React)**
- **React Hook Form (RHF) + Zod**: uncontrolled inputs hiệu năng tốt, mapping lỗi BE (422).
- **Async validation**: check trùng lặp (username/email) trước khi submit.
- **Form perf**: tránh re-render toàn form, dùng `Controller` khi cần controlled.

---

## Backend (BE)

> **Mục tiêu phỏng vấn:** kiểm soát cạnh tranh ghi/đọc, xử lý deadlock an toàn, và thiết kế luồng khoá hợp lý để không nghẽn hệ thống.

### 1) Retry on Deadlock (transaction helper)
```php
use Illuminate\Support\Facades\DB;
use Illuminate\Database\QueryException;

function withDeadlockRetry(callable $fn, int $max=3, int $baseMs=100){
  $attempt = 0;
  beginning:
  try {
    return DB::transaction(function() use ($fn){ return $fn(); }, 3);
  } catch (QueryException $e) {
    $sqlState = $e->errorInfo[0] ?? null;
    $errno    = $e->errorInfo[1] ?? null; // 1213 deadlock, 1205 lock wait timeout
    if (in_array($errno, [1213,1205]) && $attempt++ < $max){
      usleep(($baseMs + random_int(0, $baseMs)) * 1000);
      goto beginning;
    }
    throw $e;
  }
}
```

### 2) Pessimistic Locking & Work Queue (SKIP LOCKED)
```php
// Lấy 10 task chưa xử lý, tránh giành giật cùng worker khác
$tasks = DB::select("
  SELECT id FROM tasks
  WHERE status = 'pending'
  ORDER BY id
  FOR UPDATE SKIP LOCKED
  LIMIT 10
");
foreach ($tasks as $t) {
  // xử lý...
  DB::table('tasks')->where('id',$t->id)->update(['status'=>'processing']);
}
```

### 3) Optimistic Locking (version column)
```php
// migration: add `version` int default 0
// update có điều kiện theo version hiện tại
$affected = DB::table('posts')
  ->where('id', $id)
  ->where('version', $currentVersion)
  ->update(['title'=>$title, 'version'=> DB::raw('version + 1')]);

if ($affected === 0) {
  throw new \RuntimeException('Conflict: version changed'); // 409
}
```

### 4) Outbox Pattern (gọn)
```php
DB::transaction(function() use ($order){
  // 1) thay đổi trạng thái domain
  $order->update(['status'=>'PAID']);
  // 2) ghi event vào outbox
  DB::table('outbox')->insert([
    'event_type'=>'order.paid',
    'payload'=> json_encode(['order_id'=>$order->id]),
    'dispatched_at'=> null,
    'created_at'=> now()
  ]);
});
// consumer riêng đọc outbox và publish -> đánh dấu dispatched_at
```

**Lưu ý phỏng vấn**
- Khoá theo **thứ tự nhất quán** để giảm deadlock (vd: luôn lock theo `id` tăng).
- Giao dịch **ngắn**: đọc trước, chuẩn bị dữ liệu ngoài transaction, chỉ bao bọc phần ghi.

---

## Frontend (FE)

> **Mục tiêu phỏng vấn:** xây form nhanh/nhẹ với RHF+Zod, vẫn map được lỗi 422 từ Laravel.

### 1) React Hook Form + Zod
```tsx
import { useForm } from 'react-hook-form';
import { z } from 'zod';
import { zodResolver } from '@hookform/resolvers/zod';

const schema = z.object({
  email: z.string().email(),
  username: z.string().min(3),
  password: z.string().min(8)
});

type FormData = z.infer<typeof schema>;

export default function SignUpForm(){
  const { register, handleSubmit, formState: { errors, isSubmitting }, setError } =
    useForm<FormData>({ resolver: zodResolver(schema), mode:'onBlur' });

  const onSubmit = async (data: FormData) => {
    const r = await fetch('/api/signup', { method:'POST', headers:{'Content-Type':'application/json'}, body: JSON.stringify(data) });
    if (r.status === 422){
      const j = await r.json();
      for (const [field, msgs] of Object.entries(j.errors || {})) {
        setError(field as keyof FormData, { type:'server', message: (msgs as string[])[0] });
      }
      return;
    }
    // success...
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <input placeholder="Email" {...register('email')} />
      {errors.email && <p role="alert">{errors.email.message}</p>}

      <input placeholder="Username" {...register('username')} />
      {errors.username && <p role="alert">{errors.username.message}</p>}

      <input type="password" placeholder="Password" {...register('password')} />
      {errors.password && <p role="alert">{errors.password.message}</p>}

      <button disabled={isSubmitting}>Create account</button>
    </form>
  );
}
```

### 2) Async validation (duplicate check)
```tsx
// ví dụ đơn giản: debounce + HEAD request
async function isUsernameTaken(u: string){
  const r = await fetch('/api/users/exists?username='+encodeURIComponent(u), { method:'HEAD' });
  return r.status === 200; // 200=exists, 404=not found
}
```

### 3) Khi cần controlled (with Controller)
```tsx
import { Controller, useForm } from 'react-hook-form';
// ...
<Controller
  name="bio"
  control={control}
  render={({ field }) => <Textarea {...field} />}
/>
```

---

## English for Interview (skills only)

**Q&A (Concurrency & Forms)**
- *How do you resolve deadlocks in production?* — Detect by SQLSTATE/errno (1213/1205), retry with jitter and cap, shorten transactions, lock in a consistent order.
- *Why choose optimistic vs pessimistic locking?* — Optimistic for low-conflict, user-facing edits; pessimistic for queues or high-contention resources.
- *Why RHF over controlled forms?* — Uncontrolled inputs reduce re-renders; integrate Zod for schema; still map server 422 easily.

---

### Query Optimization (Supplement)
> Chủ đề mới: **COUNT(*) nhanh** (covering index), **`ORDER BY ... LIMIT` với index đúng hướng**, **Approx count** cho thống kê, **Replica lag & read-scaling**.

**COUNT(*) nhanh bằng index phủ**
```sql
-- Nếu chỉ đếm theo điều kiện trên cột được index phủ, MySQL dùng index-only:
SELECT COUNT(*) FROM posts FORCE INDEX (idx_posts_status_created)
WHERE status='PUBLISHED' AND created_at >= '2025-01-01';
```

**ORDER BY ... LIMIT khớp index**
```sql
-- Index (status, created_at DESC) giúp tránh filesort:
SELECT id, title FROM posts
WHERE status='PUBLISHED'
ORDER BY created_at DESC
LIMIT 20;
```

**Approx count (analytics)**
- Với bảng rất lớn, tránh `COUNT(*)` thật; dùng **bảng số liệu** cập nhật định kỳ hoặc công cụ OLAP. Trên MySQL có thể duy trì **counter** theo ngày.

**Replica lag & read scaling**
- Tách **read** sang replica; chú ý **lag** → có thể đọc dữ liệu cũ sau khi ghi.  
- Các thao tác cần **fresh read** nên đọc **primary** (ví dụ: ngay sau khi user cập nhật hồ sơ).

---

### Activity
- Viết helper `withDeadlockRetry()` và chạy thử với 2 worker cố tình tạo deadlock.
- Tạo bảng `tasks` và demo lấy task bằng `FOR UPDATE SKIP LOCKED` (2 worker song song).
- FE: form đăng ký bằng **RHF+Zod**, map lỗi 422, thêm check username trùng.

### Practice Tasks
- Thiết kế chiến lược **optimistic/pessimistic** cho 2 luồng cụ thể trong dự án của bạn.
- Ghi lại retry logs (attempt, errno) khi gặp deadlock giả lập.
- (Supplement) Thử so sánh `COUNT(*)` thường vs đếm bằng index phủ.

### Acceptance Criteria
- Deadlock được retry thành công, giao dịch ngắn; không rơi vào vòng lặp vô hạn.
- FE form hoạt động mượt, lỗi field hiển thị đúng; async check chạy ổn.
- Có ghi chú rõ ràng về `COUNT(*)` nhanh và chiến lược đọc replica.

### Docs & References
- Laravel Database: https://laravel.com/docs/database
- MySQL Locking & Concurrency: https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-transaction-model.html
- MySQL SKIP LOCKED: https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html
- React Hook Form: https://react-hook-form.com/
- Zod: https://zod.dev/
