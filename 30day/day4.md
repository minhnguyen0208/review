# Day 04 — Transactions & Idempotency + Outbox Pattern (Laravel) | React Forms & Validation (+ Debounced Search)

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (Laravel + MySQL)**
- **Database Transaction**: nhóm thao tác tạo/cập nhật nhiều bảng; rollback khi lỗi.
- **Idempotency**: xử lý request lặp lại an toàn (key theo header + body).
- **Outbox Pattern**: ghi sự kiện bền vững trong cùng transaction, worker phát đi sau (chống mất sự kiện).
- **MySQL**: Isolation levels (đọc phù hợp), khóa hàng (`SELECT ... FOR UPDATE`), unique constraints hỗ trợ idempotency.
- **Design Pattern**: **Unit of Work** (gắn với transaction) + event dispatch tách rời side-effects.

**Frontend (React)**
- **Controlled Forms**: `onChange` cập nhật state; hiển thị lỗi field theo envelope `{code,message,errors[]}`.
- **Validation**: sync (client checks) + async (BE 422); merge hai nguồn lỗi gọn gàng.
- **Debounced Search**: custom hook `useDebounce` để tránh spam API; kết hợp vào form/search box.

**Deliverables**
- BE: Endpoint `POST /orders` tạo order + outbox event trong **1 transaction**, hỗ trợ **Idempotency-Key**.
- FE: Form tạo order (tối giản) + thông báo lỗi field; ô search có debounce.

---
- **ACID & isolation level** lựa chọn phù hợp (mặc định `READ COMMITTED` cho CRUD).
- **Row locking** với `SELECT ... FOR UPDATE` ở các thao tác giảm tồn kho/cập nhật số dư.
- **Unique/Business key** để hỗ trợ **idempotency** (ngăn tạo trùng).
- **Idempotency‑Key**: hash(header+body), lưu & trả lại response nếu duplicate.
- **Outbox pattern**: at‑least‑once delivery, retry riêng, theo dõi `dispatched_at`.
- **Eventual consistency**: consumer phải **idempotent** khi xử lý event lặp.
- **Controlled forms**: đồng bộ state nhập + mapping lỗi 422 field‑level.
- **Debounced search** bằng custom hook `useDebounce` để giảm gọi API thừa.
- **Merge validation**: client checks + server 422; UI a11y (error `role="alert"`).
## Backend (BE)

> **Mục tiêu phỏng vấn:** chứng minh xử lý **tính nhất quán** và **an toàn khi retry**, chủ động nói về “at-least-once delivery” và vì sao cần Outbox.

**1) Transaction + Unit of Work — trước khi xem code**  
Vì sao: nhiều thao tác cần cùng thành công hoặc cùng huỷ; gộp write + outbox trong 1 transaction.

```php
// app/Http/Controllers/OrderController.php
class OrderController extends Controller {
  public function store(CreateOrderRequest $req){
    $payload = $req->validated();

    return DB::transaction(function() use ($payload){
      // 1) Write domain
      $order = Order::create([
        'user_id' => $payload['user_id'],
        'total'   => $payload['total'],
        'status'  => 'NEW',
      ]);

      // 2) Outbox event (persisted same tx)
      Outbox::create([
        'event'   => 'OrderCreated',
        'payload' => json_encode(['id'=>$order->id, 'total'=>$order->total]),
      ]);

      return response()->json(['id'=>$order->id], 201);
    });
  }
}
```

**2) Idempotency-Key (header) — trước khi xem code**  
Vì sao: client retry không tạo trùng đơn hàng; hash từ header + body, lưu và trả lại response cũ nếu trùng.

```php
// Middleware hoặc trong Controller trước khi handle
$keySource = $req->header('Idempotency-Key') . '|' . $req->getContent();
$key = hash('sha256', $keySource);

$hit = DB::table('api_idempotency')->where('key_hash', $key)->first();
if ($hit){
  return response()->json(json_decode($hit->response, true), 200);
}

// ... handle và lưu kết quả
$result = ['id'=>$order->id];
DB::table('api_idempotency')->updateOrInsert(
  ['key_hash'=>$key],
  ['response'=>json_encode($result)]
);
```

**3) Outbox Dispatcher Worker — trước khi xem code**  
Vì sao: publish sự kiện sau khi transaction commit; có thể retry riêng, không mất event.

```php
// app/Console/Commands/DispatchOutbox.php
class DispatchOutbox extends Command {
  protected $signature = 'outbox:dispatch';
  public function handle(){
    Outbox::whereNull('dispatched_at')->limit(100)->get()->each(function($rec){
      try {
        event(new \App\Events\OrderCreated($rec->payload));
        $rec->update(['dispatched_at'=>now()]);
      } catch (\Throwable $e){
        \Log::warning('Outbox dispatch failed', ['id'=>$rec->id, 'err'=>$e->getMessage()]);
      }
    });
  }
}
```

**4) MySQL: Isolation & Locking (ghi chú)**
- **READ COMMITTED** đa số là đủ cho API CRUD (giảm phantom read so với READ UNCOMMITTED).  
- Dùng `SELECT ... FOR UPDATE` khi cần **lock dòng** trước khi ghi (ví dụ: giảm tồn kho).  
- Kết hợp **UNIQUE KEY** để chống trùng khoá nghiệp vụ (vd `orders(unique_business_id)`).

---

## Frontend (FE)

> **Mục tiêu phỏng vấn:** form controlled, hiển thị lỗi field rõ ràng, có **debounce** khi search để giảm tải server.

**1) Controlled Form + lỗi field từ 422**
```tsx
type FieldErr = Record<string,string[]>;

const [form, setForm] = useState({ user_id:'', total:'' });
const [errors, setErrors] = useState<FieldErr>({});
const [submitting, setSubmitting] = useState(false);
const onChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  setForm(prev => ({ ...prev, [e.target.name]: e.target.value }));
};

async function submit(){
  setSubmitting(true); setErrors({});
  const r = await fetch('/api/orders', {
    method:'POST',
    headers:{ 'Content-Type':'application/json', 'Idempotency-Key': crypto.randomUUID() },
    body: JSON.stringify(form)
  });
  if (r.status === 422){
    const j = await r.json();
    setErrors(j.errors || {});
  } else if (!r.ok){
    // show toast from {code,message}
  } else {
    // success flow
  }
  setSubmitting(false);
}
```

**2) Debounced Search Hook (tái sử dụng)**
```tsx
function useDebounce<T>(value:T, delay=300){
  const [v, setV] = useState(value);
  useEffect(()=>{ const id = setTimeout(()=>setV(value), delay); return ()=>clearTimeout(id); }, [value, delay]);
  return v;
}

const [q, setQ] = useState('');
const dq = useDebounce(q, 400);
useEffect(()=>{
  if(!dq) return;
  // fetch(`/api/products?q=${encodeURIComponent(dq)}`)
}, [dq]);
```

**3) UI hiển thị lỗi field**
```tsx
<input name="user_id" value={form.user_id} onChange={onChange} />
{errors.user_id && <p role="alert">{errors.user_id[0]}</p>}

<input name="total" value={form.total} onChange={onChange} />
{errors.total && <p role="alert">{errors.total[0]}</p>}
```

---

## English for Interview (skills only)

**Technical Q&A**
1) *Why do we need an Outbox pattern?* — To ensure reliable event publishing with DB writes (**atomicity**). Publish after commit, with retryable worker.  
2) *How do you design idempotent create endpoints?* — Idempotency key (header+body hash), unique constraints, and return cached response for duplicates.  
3) *Which isolation level for typical APIs and why?* — Usually **READ COMMITTED** balances consistency/performance; escalate when truly needed.

**Behavioral (S.T.A.R.)**
- *Preventing double-charging*: added idempotency + unique constraints + outbox; observed zero duplicates post-fix.

---

### Activity
**Backend (Laravel + MySQL)**
- **Database Transaction**: nhóm thao tác tạo/cập nhật nhiều bảng; rollback khi lỗi.
- **Idempotency**: xử lý request lặp lại an toàn (key theo header + body).
- **Outbox Pattern**: ghi sự kiện bền vững trong cùng transaction, worker phát đi sau (chống mất sự kiện).
- **MySQL**: Isolation levels (đọc phù hợp), khóa hàng (`SELECT ... FOR UPDATE`), unique constraints hỗ trợ idempotency.
- **Design Pattern**: **Unit of Work** (gắn với transaction) + event dispatch tách rời side-effects.
- Implement BE endpoints and run database migration/index creation
- Verify EXPLAIN plan and capture a note/screenshot
- Build FE screen and verify loading/error/empty states
- Write 1 short interview takeaway (2–3 sentences)

### Practice Tasks
- `POST /orders`: transaction tạo order + outbox ghi `OrderCreated`.  
- Idempotency middleware: detect duplicate và trả response trước đó.  
- React form có lỗi 422 field-level; search box debounce.

### Acceptance Criteria
- BE: transaction all-or-nothing; outbox có worker phát và retry; idempotency hoạt động (no duplicate).  
- FE: form controlled, hiển thị lỗi field; debounce phát request hợp lý.

### Docs & References
- Laravel Database & Transactions: https://laravel.com/docs/database
- Laravel Events: https://laravel.com/docs/events
- Laravel Validation: https://laravel.com/docs/validation
- React (Forms & Effects): https://react.dev/learn
