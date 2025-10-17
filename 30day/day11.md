# Day 11 — Advanced Aggregations & Reporting (Window Functions) | React Error Boundaries & Suspense

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (Laravel + MySQL)**
- window functions, reporting queries
- optimize grouping
- json aggregation patterns
- **Query Optimization**: đọc `EXPLAIN/ANALYZE`; hành động sau plan (index/viết lại).
- **JSON_ARRAYAGG** trong `GROUP BY` để gom object con theo nhóm.
- **Sargable predicates**; **composite/covering index** khớp `WHERE` + `ORDER BY`.
**Frontend (React)**
- error boundaries, suspense for data fetching
**Notes**
- include EXPLAIN ANALYZE, histograms decision, sargable rewrite

---

## Backend (BE)
> **Mục tiêu phỏng vấn:** dùng window functions/JSON aggregation hợp lý, và tối ưu sau khi đọc EXPLAIN.
**Window functions — rolling sums/top-N per group**
```sql
SELECT
  p.author_id,
  p.id,
  p.created_at,
  ROW_NUMBER() OVER (PARTITION BY p.author_id ORDER BY p.created_at DESC) AS rn,
  SUM(p.views) OVER (PARTITION BY p.author_id ORDER BY p.created_at ROWS BETWEEN 6 PRECEDING AND CURRENT ROW) AS views_7_posts
FROM posts p
WHERE p.created_at >= CURDATE() - INTERVAL 90 DAY;
-- Top 3 bài gần nhất theo author: WHERE rn <= 3
```
**JSON_ARRAYAGG theo nhóm**
```sql
SELECT
  u.id AS author_id,
  JSON_ARRAYAGG(JSON_OBJECT('id',p.id,'title',p.title) ORDER BY p.created_at DESC) AS recent_posts
FROM users u
LEFT JOIN posts p ON p.author_id = u.id
GROUP BY u.id;
```
**EXPLAIN → Hậu kiểm & tối ưu**
- Nếu `type=ALL` trên `posts` → thêm index theo thời gian/author (vd `(author_id, created_at DESC)`).
- Nếu `Using temporary`/`Using filesort` ở GROUP BY → giảm select list, cân nhắc `ANY_VALUE()` hoặc materialize trước khi group.
- Nếu pred không sargable (`DATE(p.created_at)=...`) → rewrite thành range.
- Chạy `ANALYZE TABLE` sau thay đổi lớn; cân nhắc **histogram** cho cột skewed.

**Laravel snippet (Query Builder)**
```php
$res = DB::table('posts as p')
  ->selectRaw("p.author_id, p.id, p.created_at,
               ROW_NUMBER() OVER (PARTITION BY p.author_id ORDER BY p.created_at DESC) as rn")
  ->where('p.created_at','>=', now()->subDays(90))
  ->orderByDesc('p.created_at')
  ->limit(1000)
  ->get();
```

---

## Frontend (FE)
> **Mục tiêu phỏng vấn:** bảo vệ UI với Error Boundaries, cân nhắc Suspense đúng chỗ.
**Error Boundary (TypeScript)**
```tsx
import React from 'react';

type State = { hasError: boolean };
export class ErrorBoundary extends React.Component<React.PropsWithChildren, State>{
  state: State = { hasError: false };
  static getDerivedStateFromError(){ return { hasError: true }; }
  componentDidCatch(err: any){ console.error(err); }
  render(){ return this.state.hasError ? <p role="alert">Something went wrong.</p> : this.props.children; }
}
```
**Suspense integration (optional)**
```tsx
// With React Query experimental suspense: <React.Suspense fallback="Loading..."><Page/></React.Suspense>
```

---

## English for Interview (skills only)
**Behavioral — Performance Incident**
- *Situation*: Report page timed out after data growth.
- *Action*: Added composite index `(author_id, created_at)`, rewrote date predicate to a range, replaced OFFSET with keyset; verified with `EXPLAIN ANALYZE`.
- *Result*: P95 dropped from 3.2s → 480ms; CPU/IO reduced significantly.

- Explain a performance incident and how you fixed it with indexing/rewrites

---

### Activity
- Viết 1 truy vấn window function + kiểm tra EXPLAIN.
- Gom dữ liệu bằng JSON_ARRAYAGG theo nhóm; đo kích thước JSON.
- Ghi 3 hành động tối ưu cụ thể sau khi đọc plan.

### Practice Tasks
- Thêm index phục vụ query window/nhóm; kiểm tra lại EXPLAIN.
- BE endpoint trả aggregate cần thiết; FE demo error boundary bọc quanh list.
- Ghi note before/after (rows/Extra/latency).

### Acceptance Criteria
- EXPLAIN cho query chính không còn full scan/ filesort (khi khả thi).
- Endpoint trả dữ liệu đúng; error boundary hoạt động khi mock lỗi.
- Có ghi chú tối ưu rõ ràng (index/viết lại).

### Docs & References
- MySQL Window Functions: https://dev.mysql.com/doc/refman/8.0/en/window-functions.html
- MySQL EXPLAIN/ANALYZE: https://dev.mysql.com/doc/refman/8.0/en/explain-output.html
- Laravel Query Builder: https://laravel.com/docs/queries
- React Error Boundaries: https://react.dev/reference/react/Component#catching-rendering-errors-with-an-error-boundary