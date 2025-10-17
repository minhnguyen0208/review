# Day 10 — API Design v2 + Query Optimization | React Query Advanced (Prefetch, Optimistic Updates)

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (Laravel + MySQL)**
- **API versioning** (path vs header); deprecation policy & docs.
- **Filter/Sort documentation**: query params, types, examples.
- **Error envelope** & rate limiting conventions.
- **Query Optimization**: đọc `EXPLAIN/ANALYZE` (type/rows/Extra); **sargable predicates**; **composite/covering index** khớp `WHERE` + `ORDER BY`.
- **Aggregations**: `JSON_ARRAYAGG`/`GROUP_CONCAT` trong `GROUP BY` để gom dữ liệu con (kiểm soát kích thước).

**Frontend (React)**
- **Prefetch** detail khi hover/focus.
- **Optimistic updates** cho create/update/delete; rollback on error; invalidate on settle.
- **Cache invalidation** selective (queryKey) với TanStack Query.

---

## Backend (BE)

> **Mục tiêu phỏng vấn:** tài liệu hoá API rõ ràng, áp dụng tối ưu truy vấn sau khi đọc EXPLAIN.

**API Versioning & Docs (curl sample)**
```md
GET /v2/posts?q=queue&sort=-created_at&author_id=12
Parameters:
- q: string (FULLTEXT search if available)
- sort: field with optional '-' for desc
- author_id: int
Errors: {code,message,errors[]}
Rate limit: 120 rpm per user/IP
```

**EXPLAIN → Hành động tối ưu**
```sql
EXPLAIN SELECT id,title,created_at FROM posts
WHERE author_id = ? AND created_at >= ?
ORDER BY created_at DESC
LIMIT 20;
```
- Nếu `Extra` có `Using filesort` → thêm index `(author_id, created_at DESC)`.
- Nếu `type=ALL` (full scan) → xem lại predicate sargable, bổ sung index.
- Nếu `rows` lớn bất thường → cân nhắc histogram (8.0) hoặc tách shard theo thời gian.

**JSON_ARRAYAGG trong GROUP BY**
```sql
SELECT p.id, p.title,
       JSON_ARRAYAGG(JSON_OBJECT('id',c.id,'body',c.body) ORDER BY c.created_at) AS comments
FROM posts p
LEFT JOIN comments c ON c.post_id = p.id
WHERE p.author_id = ?
GROUP BY p.id, p.title
ORDER BY p.created_at DESC
LIMIT 20;
```

**Laravel snippet (Repository)**
```php
public function listV2(array $f, int $per=20){
  $q = Post::query()
    ->when($f['author_id']??null, fn($q,$v)=>$q->where('author_id',$v))
    ->when($f['q']??null, fn($q,$v)=>$q->whereFullText(['title','body'],$v));

  $q->orderByDesc('created_at');
  return $q->paginate($per)->withQueryString();
}
```

---

## Frontend (FE)

> **Mục tiêu phỏng vấn:** dùng React Query nâng cao: prefetch + optimistic update + invalidate chọn lọc.

**Prefetch detail on hover**
```tsx
import { useQueryClient } from '@tanstack/react-query';
const qc = useQueryClient();
function PostLink({id, children}:{id:number; children:React.ReactNode}){
  return <a href={'/posts/'+id}
    onMouseEnter={()=> qc.prefetchQuery({ queryKey:['post',id], queryFn:()=> fetch('/api/posts/'+id).then(r=>r.json()) })}>
    {children}
  </a>;
}
```

**Optimistic update (toggle like)**
```tsx
const toggleLike = useMutation({
  mutationFn: async(id:number)=>{
    const r = await fetch('/api/posts/'+id+'/like',{ method:'POST'});
    if(!r.ok) throw new Error('Failed');
    return r.json();
  },
  onMutate: async(id:number)=>{
    await qc.cancelQueries({ queryKey:['posts'] });
    const prev = qc.getQueryData<any>(['posts']);
    qc.setQueryData<any>(['posts'], (old:any)=> ({
      ...old, data: old.data.map((p:any)=> p.id===id? {...p, liked:!p.liked}:p)
    }));
    return { prev };
  },
  onError: (_err, _vars, ctx)=> ctx?.prev && qc.setQueryData(['posts'], ctx.prev),
  onSettled: ()=> qc.invalidateQueries({ queryKey:['posts'] }),
});
```

---

## English for Interview (skills only)

**Explain choices & performance**
- *Why path-based versioning?* — Clear routing, cache friendly; header-based is flexible but less visible.
- *How do you optimize a slow list endpoint?* — Composite index, sargable predicates, keyset pagination, validate with EXPLAIN/ANALYZE.
- *How do you avoid optimistic UI inconsistencies?* — Rollback on error and always invalidate after mutation settles.

---

### Activity
- Viết EXPLAIN cho list v2 và đề xuất 2 tối ưu cụ thể.
- Thêm 1 ví dụ JSON_ARRAYAGG và đo kích thước payload.
- FE: thêm prefetch & 1 optimistic mutation; kiểm tra rollback khi lỗi.

### Practice Tasks
- Document v2 endpoints (params, errors, rate limit).
- Tạo index `(author_id, created_at DESC)` nếu thiếu; thử `EXPLAIN ANALYZE`.
- FE: prefetch detail + optimistic update; invalidate chọn lọc.

### Acceptance Criteria
- Docs có ví dụ curl + schema, rõ cách sort/filter.
- `EXPLAIN` không còn full scan; latency giảm sau tối ưu (ghi lại số liệu).
- FE hoạt động đúng khi lỗi (rollback) và khi thành công (invalidate).

### Docs & References
- Laravel Eloquent/Queries: https://laravel.com/docs/eloquent
- MySQL EXPLAIN/ANALYZE: https://dev.mysql.com/doc/refman/8.0/en/explain-output.html
- MySQL JSON functions: https://dev.mysql.com/doc/refman/8.0/en/json-function-reference.html
- TanStack Query: https://tanstack.com/query/latest
- API Design Guidelines (general)
