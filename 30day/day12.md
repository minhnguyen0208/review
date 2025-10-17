# Day 12 — Caching Advanced (Tags/Invalidation) | React Context + Reducer (State Management)

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (Laravel)**
- **Cache tags** với Redis, selective invalidation cho domain.
- **Request caching** (per-request), tránh n+1 external calls.
- **Cache stampede**: lock/early recompute.

**Frontend (React)**
- **Context + useReducer** cho global state nhỏ.
- **Memoized selectors** để giảm re-render.

---

## Backend (BE)
> **Mục tiêu phỏng vấn:** cache có chiến lược, tránh stale/đua tranh, invalidation rõ ràng.

**Tag-based Cache & Stampede Guard**
```php
// Cache::tags require taggable driver (redis)
$posts = Cache::tags(['posts'])->remember('posts:popular', 600, function(){
  return Post::popular()->take(10)->get();
});

// Stampede guard via lock
$val = Cache::lock('recompute:stats', 30)->block(5, function(){
  return recompute_stats();
});
```

**Request Cache (per-request)**
```php
// app/Http/Middleware/RequestCache.php
public function handle($req, Closure $next){
  app()->scoped('req.cache', fn()=> new \Illuminate\Support\Collection());
  return $next($req);
}
// somewhere in service
$cache = app('req.cache');
if($cache->has('external:token')) return $cache->get('external:token');
$token = Http::asForm()->post('...')->json('access_token');
$cache->put('external:token', $token);
```

---

## Frontend (FE)
> **Mục tiêu phỏng vấn:** quản lý state gọn, tránh re-render toàn cây.

**Context + useReducer**
```tsx
const StateCtx = React.createContext<any>(null);
const DispatchCtx = React.createContext<any>(()=>{});
function reducer(s:any, a:any){ if(a.type==='setUser') return {...s, user:a.user}; return s; }
export function Provider({children}:{children:React.ReactNode}){
  const [state, dispatch] = React.useReducer(reducer, { user:null });
  const value = React.useMemo(()=>state,[state]);
  return <DispatchCtx.Provider value={dispatch}><StateCtx.Provider value={value}>{children}</StateCtx.Provider></DispatchCtx.Provider>
}
function useUser(){ const s = React.useContext(StateCtx); return s.user; }
```

---

## English for Interview (skills only)
- *How do you balance consistency vs latency with caching?* — Clear TTLs, event-driven invalidation, and metrics for stale rate.
- *How do you prevent cache stampede?* — Locks, background refresh, and jittered TTLs.

---

### Query Optimization (Supplement)
> Chủ đề mới: **Slow Query Log**, **Optimizer Trace**, **Index Merge**, **Prefix Index**, **Covering with narrower selects**.

**Enable & read Slow Query Log**
```sql
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 0.5; -- seconds
```

**Optimizer Trace (diagnostics)**
```sql
SET optimizer_trace='enabled=on';
SELECT ...; -- your query
SELECT * FROM information_schema.OPTIMIZER_TRACE\G
```

**Index Merge (watch out)** — khi thấy `index_merge` trong EXPLAIN, thử viết lại filter (UNION ALL theo cột) để tối ưu.

**Prefix Index cho VARCHAR**
```sql
CREATE INDEX idx_users_email_pref ON users(email(12));
```

**Covering via narrower selects** — chỉ `SELECT` các cột cần để kích hoạt `Using index` khi phù hợp.

---

### Activity
- Áp dụng cache tags và viết 1 case invalidation cụ thể.
- Bật slow query log, bắt 1 truy vấn chậm và ghi lại nguyên nhân từ OPTIMIZER_TRACE.

### Practice Tasks
- Cài đặt cache tags + stampede guard cho 1 endpoint nóng.
- FE: Context + Reducer cho user/session + selector hook.
- (Supplement) Thu 1 slow query → đọc EXPLAIN & OPTIMIZER_TRACE → đề xuất tối ưu.

### Acceptance Criteria
- Invalidation đúng phạm vi (không xoá quá tay).
- UI re-render tối thiểu khi state thay đổi cục bộ.
- Có báo cáo ngắn về slow query (triệu chứng → nguyên nhân → hành động).

### Docs & References
- Laravel Cache: https://laravel.com/docs/cache
- React Context & Reducer: https://react.dev/learn
- MySQL Slow Query Log: https://dev.mysql.com/doc/refman/8.0/en/slow-query-log.html
- Optimizer Trace: https://dev.mysql.com/doc/refman/8.0/en/optimizer-trace.html
