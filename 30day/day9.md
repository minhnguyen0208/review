# Day 09 — Query Optimization (MySQL) + Aggregations (JSON_ARRAYAGG) | React List Performance (Virtualization)

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (MySQL + Laravel)**
- **Query Optimization**: đọc `EXPLAIN`/`EXPLAIN ANALYZE`, xác định bottleneck (rows, filtered, Extra).
- **Aggregations**: `JSON_ARRAYAGG`, `GROUP_CONCAT`, window functions (8.0+) cho tổng hợp linh hoạt.
- **Sargable predicates**: tránh hàm lên cột index; **covering/composite index** khớp `WHERE` + `ORDER BY`.
- **Rewrite patterns**: `UNION ALL` thay `OR` trên cột khác index, `EXISTS` vs `IN`, keyset pagination.
- **Stats & metadata**: `ANALYZE TABLE`, histograms (8.0) khi phân phối lệch.
- **Post‑EXPLAIN actions**: thêm/chỉnh index, đổi join order, limit cột select, cân nhắc materialized subquery.

**Frontend (React)**
- **Performance list**: virtualization (react‑window), memo hoá row renderer, stable callbacks.
- **Avoid re‑renders**: key đúng, tách state, `useMemo/useCallback` vừa đủ.

**Deliverables**
- 1 endpoint trả về danh sách posts kèm `JSON_ARRAYAGG` comments (sample) + trang FE virtualized list.

---

## Backend (BE)

> **Mục tiêu phỏng vấn:** đọc `EXPLAIN` → suy luận đúng vấn đề → đưa ra hành động tối ưu cụ thể (index/viết lại truy vấn).

### 1) Tổng hợp bằng JSON_ARRAYAGG (MySQL 5.7.22+/8.0)
Khi cần gói các items con theo nhóm (ví dụ comments của post):
```sql
SELECT
  p.id,
  p.title,
  JSON_ARRAYAGG(JSON_OBJECT('id', c.id, 'body', c.body)) AS comments
FROM posts p
LEFT JOIN comments c ON c.post_id = p.id
WHERE p.author_id = ?
GROUP BY p.id, p.title
ORDER BY p.created_at DESC
LIMIT 20;
```
**Lưu ý:**
- Nếu cần **ordering trong mảng**, MySQL 8.0.14+ có `JSON_ARRAYAGG(x ORDER BY c.created_at)`.
- Với tập dữ liệu lớn, cân nhắc tách endpoint (list + detail) hoặc keyset pagination để không gom quá nhiều.

### 2) Sargable predicates & index phủ
Tránh làm mất khả năng dùng index:
```sql
-- KHÔNG sargable (hàm lên cột):
WHERE DATE(created_at) = '2025-10-01'
-- Thay bằng sargable:
WHERE created_at >= '2025-10-01' AND created_at < '2025-10-02'
```
Thiết kế index khớp filter+sort:
```sql
CREATE INDEX idx_posts_author_created ON posts(author_id, created_at DESC);
-- Query
SELECT id, title, created_at FROM posts
WHERE author_id = ?
ORDER BY created_at DESC
LIMIT 20; -- tránh "Using filesort", hy vọng "Using index"
```

### 3) Rewrite patterns
- **OR → UNION ALL** khi mỗi nhánh dùng index khác nhau:
```sql
SELECT id FROM posts WHERE author_id = 10
UNION ALL
SELECT id FROM posts WHERE status = 'PUBLISHED';
```
- **IN vs EXISTS**: `EXISTS` tốt khi bảng phụ có index và điều kiện tương quan.
```sql
SELECT p.* FROM posts p
WHERE EXISTS (
  SELECT 1 FROM comments c
  WHERE c.post_id = p.id AND c.flagged = 1
);
```
- **Keyset** (đã học): nhanh và ổn định hơn OFFSET cho dữ liệu lớn.

### 4) Đọc EXPLAIN & hành động sau đó
Ví dụ với truy vấn tổng hợp:
```sql
EXPLAIN SELECT p.id, p.title,
  JSON_ARRAYAGG(JSON_OBJECT('id', c.id, 'body', c.body)) AS comments
FROM posts p
LEFT JOIN comments c ON c.post_id = p.id
WHERE p.author_id = ?
GROUP BY p.id, p.title
ORDER BY p.created_at DESC
LIMIT 20;
```
**Checklist diễn giải:**
- `type` nên là `ref`/`range` cho `p`; `ref` cho `c` trên `post_id`.
- `rows` nhỏ (lọc tốt); `Extra` không có `Using temporary; Using filesort` trên `p`.
- Nếu thấy `Using filesort`/`Using temporary` trên `p` → thêm `(author_id, created_at)`; giảm cột `GROUP BY` (dùng `p.id` + `ANY_VALUE(p.title)` nếu phù hợp 8.0).
- Nếu `c` đọc quá nhiều → thêm index `(post_id)` hoặc `(post_id, created_at)` nếu cần order.

**Hành động sau EXPLAIN (theo thứ tự ưu tiên):**
1) **Thêm/chỉnh composite index** khớp `WHERE` + `ORDER BY`.  
2) **Sargable rewrite**: bỏ hàm trên cột, tách điều kiện.  
3) **Giảm select list**: chỉ các cột cần thiết (hỗ trợ covering).  
4) **Đổi cấu trúc truy vấn**: `EXISTS`/`UNION ALL`, materialize subquery nếu cần.  
5) **ANALYZE TABLE** cập nhật thống kê; cân nhắc **histogram** cho cột lệch.  
6) **EXPLAIN ANALYZE** (8.0.18+) đo runtime thực, phát hiện nút tốn thời gian.

### 5) Laravel — triển khai SQL & Resource
```php
// Repository
public function listWithCommentsJson(int $authorId){
  return DB::table('posts as p')
    ->leftJoin('comments as c','c.post_id','=','p.id')
    ->where('p.author_id',$authorId)
    ->groupBy('p.id','p.title')
    ->orderByDesc('p.created_at')
    ->limit(20)
    ->selectRaw("p.id, p.title, JSON_ARRAYAGG(JSON_OBJECT('id', c.id, 'body', c.body)) as comments")
    ->get();
}
```

---

## Frontend (FE)

> **Mục tiêu phỏng vấn:** render danh sách lớn mượt mà, tránh re‑render dư thừa, giữ handler/states ổn định.

### 1) Virtualized list (react-window)
```tsx
import { FixedSizeList as List } from 'react-window';

function Row({ index, style, data }: any){
  const item = data.items[index];
  return <div style={style}>{item.title}</div>;
}

<List
  height={500}
  width={600}
  itemCount={items.length}
  itemSize={40}
  itemData={{ items }}
>
  {Row}
</List>
```
### 2) Tránh re‑render không cần
- Dùng `React.memo` cho row; `itemData` ổn định (memo hoá).  
- Giữ `key` ổn định (sử dụng `id`, tránh index).  
- Kết hợp **TanStack Query** với `select` để map dữ liệu trước khi render.

---

## English for Interview (skills only)

**Q&A (Reading EXPLAIN)**
- *How do you approach a slow query?* — Reproduce, capture query + params, `EXPLAIN`/`ANALYZE`, identify table scans/filesort/temp, propose specific index/rewrite, measure after.
- *JSON_ARRAYAGG vs JOIN + app aggregation?* — Database aggregation reduces round‑trips and preserves order; watch out for huge arrays (consider paging or detail endpoint).
- *When to use histograms?* — When distribution is skewed and adding an index is not justified; helps optimizer choose better plans.

---

### Activity
- Tạo index `(author_id, created_at)` và so sánh `EXPLAIN` trước/sau.
- Viết truy vấn `JSON_ARRAYAGG` comments theo post, kiểm tra thứ tự trong mảng.
- Ghi lại 3 hành động cụ thể sau khi đọc `EXPLAIN` cho truy vấn của bạn.

### Practice Tasks
- Áp dụng sargable rewrite cho 2 truy vấn có hàm lên cột.
- Triển khai endpoint tổng hợp bằng `JSON_ARRAYAGG` + FE virtualized list.
- Viết note “before/after” về `rows`, `Extra`, và thời gian thực tế (nếu có `EXPLAIN ANALYZE`).

### Acceptance Criteria
- `EXPLAIN` giảm `rows` và không còn `Using filesort` cho query chính (khi khả thi).
- Endpoint trả JSON hợp lệ; FE list mượt khi hiển thị 5k+ items (virtualized).
- Ghi chú hậu kiểm có đề xuất chỉ mục/viết lại rõ ràng.

### Docs & References
- MySQL EXPLAIN/ANALYZE: https://dev.mysql.com/doc/refman/8.0/en/explain-output.html  
- MySQL JSON Functions: https://dev.mysql.com/doc/refman/8.0/en/json-function-reference.html  
- MySQL Histograms: https://dev.mysql.com/doc/refman/8.0/en/column-statistics-histograms.html  
- Laravel Query Builder: https://laravel.com/docs/queries  
- react-window: https://github.com/bvaughn/react-window
