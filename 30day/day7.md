# Day 07 — Security & Auth

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (Laravel + MySQL)**
- Security & Auth
**Frontend (React)**
- **Table**: sorting client, pagination, search box có debounce.
- **(Tuỳ chọn)** TanStack Table để quản lý columns/sort/pagination.

---

## Backend (BE)
> **Mục tiêu phỏng vấn:** phân biệt LIKE vs FULLTEXT, thiết kế API filter + pagination nhất quán.
**Filtering & Resource meta**
```php
// Controller
public function index(Request $r){
  $q = Post::query()
     ->when($r->author_id, fn($q,$v)=>$q->where('author_id',$v))
     ->when($r->status, fn($q,$v)=>$q->where('status',$v))
     ->when($r->date_from, fn($q,$v)=>$q->whereDate('created_at','>=',$v))
     ->when($r->date_to, fn($q,$v)=>$q->whereDate('created_at','<=',$v))
     ->orderByDesc('created_at');

  $rows = $q->paginate($r->integer('per_page',20));

  return PostResource::collection($rows)
    ->additional(['meta'=>['filters'=>$r->only(['author_id','status','date_from','date_to'])]]);
}
```
**MySQL FULLTEXT (boolean mode)**
```sql
ALTER TABLE posts ADD FULLTEXT idx_ft_posts_title_body (title, body);
-- Query sample (boolean mode): +must -exclude "prefix*"
SELECT id, title FROM posts
WHERE MATCH(title,body) AGAINST('+laravel +queue -legacy*' IN BOOLEAN MODE)
ORDER BY id DESC
LIMIT 20;
```
```php
// Laravel 9+: whereFullText
$q->when($r->q, fn($q,$v)=> $q->whereFullText(['title','body'], $v));
```

**Lưu ý khi phỏng vấn**: stopwords, stemming, ngôn ngữ; với dữ liệu lớn/đa ngôn ngữ có thể cân nhắc **Scout + Meilisearch/Elasticsearch**.

**Keyset Pagination (ổn định & nhanh)**
```php
// page sau: WHERE (created_at,id) < (last_created_at,last_id)
$q = Post::query()->orderByDesc('created_at')->orderByDesc('id');
if($r->last_id && $r->last_created_at){
  $q->where(function($q2) use($r){
    $q2->where('created_at','<',$r->last_created_at)
       ->orWhere(function($q3) use($r){
         $q3->where('created_at',$r->last_created_at)->where('id','<',$r->last_id);
       });
  });
}
$rows = $q->limit(20)->get();
```

---

## Frontend (FE)
> **Mục tiêu phỏng vấn:** table có sort/paginate/search debounce; code sạch, tách concerns.
**Search with debounce + table render**
```tsx
const [q, setQ] = useState('');
const dq = useDebounce(q, 400);

useEffect(()=>{
  (async ()=>{
    const r = await fetch('/api/posts?q='+encodeURIComponent(dq));
    const j = await r.json();
    setRows(j.data);
  })();
}, [dq]);

function useDebounce<T>(value:T, delay=400){
  const [v, setV] = useState(value);
  useEffect(()=>{ const id = setTimeout(()=>setV(value), delay); return ()=>clearTimeout(id); }, [value, delay]);
  return v;
}
```
**Client sorting & pagination skeleton**
```tsx
const [page, setPage] = useState(1);
const [sort, setSort] = useState<{key:string, dir:'asc'|'desc'}>({key:'created_at', dir:'desc'});
const sorted = useMemo(()=>{
  return [...rows].sort((a,b)=> sort.dir==='asc' ? (a[sort.key]>b[sort.key]?1:-1) : (a[sort.key]<b[sort.key]?1:-1));
}, [rows, sort]);

const pageRows = useMemo(()=>{
  const start = (page-1)*20;
  return sorted.slice(start, start+20);
}, [sorted, page]);

// UI: headers clickable -> setSort, pager buttons -> setPage
```

---

## English for Interview (skills only)
**Q&A (Search & Pagination)**
- *When do you choose FULLTEXT over LIKE searches?* — When needing relevance ranking, multi-term queries, boolean operators; LIKE is fine for small, prefix-only cases.
- *Offset vs keyset pagination?* — Keyset is stable/fast for large lists; offset okay for small admin pages or random access.
- *How do you document your filter API?* — List query params with types, defaults, and examples; include a curl sample and response schema.


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
- record & review. | API: build/secure an endpoint (auth, validation, rate limit), write integration tests.

### Practice Tasks
- Thêm `whereFullText` vào `/posts` khi `q` được cung cấp; fallback sang filter base khi không có `q`.
- FE: table với search debounce, sort client, pagination (hoặc dùng keyset từ BE).
- Viết 1 đoạn docs (curl + response schema) cho endpoint filter/search.

### Acceptance Criteria
- FULLTEXT hoạt động, không lỗi syntax; có so sánh LIKE vs FULLTEXT.
- API trả `links`/`meta` hoặc keyset tokens; filter phản ánh đúng tham số.
- FE search debounce chạy tốt; sort/pagination đúng kỳ vọng.

### Docs & References
- Laravel Eloquent: https://laravel.com/docs/eloquent
- Laravel Pagination & Resources: https://laravel.com/docs/eloquent-resources
- MySQL FULLTEXT: https://dev.mysql.com/doc/refman/8.0/en/fulltext-search.html
- React (Hooks): https://react.dev/learn
- TanStack Table (optional): https://tanstack.com/table/latest