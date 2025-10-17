# Day 10 — Policies & Authorization Deep Dive | React Forms: Controlled vs Uncontrolled

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (Laravel)**
- Policies abilities (viewAny/view/update/delete).
- Gate hooks (before/after) cho cross-cutting rules.
- Resource authorization (`authorizeResource`).

**Frontend (React)**
- Controlled vs uncontrolled inputs; dùng `useRef` cho uncontrolled.
- Giảm re-render trong form lớn; field-level state.

**Notes**
- Follow CSV if present; Query Optimization chỉ là **phần bổ sung**, không thay thế chủ đề BE.

---

## Backend (BE)
> **Mục tiêu phỏng vấn:** làm rõ model-level rules (Policy), cross-cutting với Gate hooks, và tài liệu hoá abilities.

**Policies & Abilities**
```php
// app/Policies/PostPolicy.php
class PostPolicy {
  public function viewAny(User $u){ return true; }
  public function view(User $u, Post $p){ return true; }
  public function update(User $u, Post $p){ return $u->id === $p->author_id; }
  public function delete(User $u, Post $p){ return $u->id === $p->author_id; }
}
// AuthServiceProvider
protected $policies = [ Post::class => PostPolicy::class ];
```

**Gate hooks (before/after)**
```php
Gate::before(function(User $u, string $ability){
  if($u->is_admin) return true;
});
Gate::after(function(User $u, string $ability, bool|null $result){
  Log::info('authz', ['user'=>$u->id, 'ability'=>$ability, 'result'=>$result]);
});
```

**Controller integration**
```php
class PostController extends Controller {
  public function __construct(){
    $this->authorizeResource(Post::class, 'post');
  }
  public function update(UpdatePostRequest $req, Post $post){
    $post->update($req->validated());
    return new PostResource($post);
  }
}
```

### Query Optimization (Supplement) — khác với Day 9 (không thay thế BE plan)
**Generated Columns + Functional Index**
```sql
ALTER TABLE users
  ADD COLUMN email_lc VARCHAR(255) GENERATED ALWAYS AS (LOWER(email)) STORED,
  ADD INDEX idx_users_email_lc (email_lc);
SELECT id FROM users WHERE email_lc = LOWER('USER@EXAMPLE.COM');
```

**Index Hint & STRAIGHT_JOIN (thận trọng)**
```sql
SELECT p.*
FROM posts p STRAIGHT_JOIN comments c ON c.post_id = p.id
WHERE p.author_id = ?;
```

**CTE/Derived Table**
```sql
WITH recent AS (
  SELECT id FROM posts WHERE created_at >= NOW() - INTERVAL 7 DAY
)
SELECT p.*, u.name FROM posts p
JOIN recent r ON r.id = p.id
JOIN users u ON u.id = p.author_id;
```

**Window Functions**
```sql
SELECT author_id, id, created_at,
       ROW_NUMBER() OVER (PARTITION BY author_id ORDER BY created_at DESC) as rn
FROM posts;
```

---

## Frontend (FE)
> **Mục tiêu phỏng vấn:** hiểu trade-off giữa controlled/uncontrolled, giữ form mượt và ít re-render.

**Controlled vs Uncontrolled**
```tsx
// Controlled
const [title, setTitle] = useState('');
<input value={title} onChange={(e)=>setTitle(e.target.value)} />

// Uncontrolled + ref
const ref = useRef<HTMLInputElement>(null);
<form onSubmit={(e)=>{ e.preventDefault(); console.log(ref.current?.value) }}>
  <input ref={ref} defaultValue="hello" />
</form>
```

**Giảm re-render form lớn**
- Tách field thành component con dùng `React.memo`.
- Dùng `useReducer` hoặc lib form để gom dispatch.
- Chỉ giữ trong state **những gì cần render**.

---

## English for Interview (skills only)
**Behavioral & Clarification**
- *When you disagree on an authorization rule, how do you proceed?* — Propose concrete examples, write policy tests, and decide via data/impact.
- *Clarify requirements:* “Should admins bypass all checks, or only content they own?”

---

### Activity
- Thêm `authorizeResource` cho PostController và viết 2 policy tests.
- Đo thử một query dùng **generated column + index** trước/sau.

### Practice Tasks
- Cấu hình Policy cho Post + Gate hooks (before/after).
- Viết README ghi abilities và endpoints tương ứng.
- (Supplement) thêm 1 generated column + functional index cho use-case phù hợp.

### Acceptance Criteria
- Policy chặn/cho phép đúng; log authz có đủ ngữ cảnh.
- FE form hoạt động, không re-render thừa; demo controlled & uncontrolled.
- Supplement query optimization có benchmark ngắn trước/sau.

### Docs & References
- Authorization: https://laravel.com/docs/authorization
- Policies: https://laravel.com/docs/authorization#creating-policies
- React (Forms & Refs): https://react.dev/learn
- MySQL Generated Columns: https://dev.mysql.com/doc/refman/8.0/en/create-table-generated-columns.html
- MySQL Window Functions: https://dev.mysql.com/doc/refman/8.0/en/window-functions.html
