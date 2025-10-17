# Day 11 — API Documentation (OpenAPI/Resources) | React API Client & Error Boundaries

### Plan Notes (mục tiêu & phạm vi ôn tập trong ngày)
**Backend (Laravel)**
- OpenAPI (yaml), Laravel API Resources, request/response examples, versioning
**Frontend (React)**
- API client wrapper (fetch), error boundary, retry/backoff
**Notes**
- Follow CSV if present

---

## Backend (BE)
> **Mục tiêu phỏng vấn:** trình bày API rõ ràng, sample request/response nhất quán, và có chiến lược versioning.

**OpenAPI (trích đoạn)**
```yaml
openapi: 3.0.3
info:
  title: Sample API
  version: 1.0.0
paths:
  /posts:
    get:
      summary: List posts
      parameters:
        - in: query
          name: q
          schema: { type: string }
      responses:
        '200':
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/PostCollection'
components:
  schemas:
    Post:
      type: object
      properties: { id: {type: integer}, title: {type: string} }
    PostCollection:
      type: object
      properties: { data: { type: array, items: { $ref: '#/components/schemas/Post' } } }
```

**Laravel API Resource**
```php
class PostResource extends JsonResource { public function toArray($req){
  return ['id'=>$this->id,'title'=>$this->title,'created_at'=>$this->created_at];
}}
```

### Query Optimization (Supplement)
> Chủ đề mới (khác Day 9/10): **Partitioning**, **Temporary tables & join buffer**, **optimizer_switch** (quan sát, không ép buộc).

**Range Partitioning (date)**
```sql
ALTER TABLE events PARTITION BY RANGE (YEAR(created_at)) (
  PARTITION p2023 VALUES LESS THAN (2024),
  PARTITION p2024 VALUES LESS THAN (2025),
  PARTITION pmax  VALUES LESS THAN MAXVALUE
);
-- Chỉ cân nhắc nếu bảng cực lớn và queries theo range năm/tháng.
```

**Temporary table vs Derived**
```sql
CREATE TEMPORARY TABLE recent_ids (id BIGINT PRIMARY KEY);
INSERT INTO recent_ids SELECT id FROM posts WHERE created_at >= NOW() - INTERVAL 7 DAY;
SELECT p.* FROM posts p JOIN recent_ids r ON r.id = p.id;
-- Đo trước/sau: đôi khi tmp table giúp avoid re-scan/duplicate work.
```

**Join Buffer & optimizer_switch (read-only)**
```sql
SET SESSION optimizer_switch='block_nested_loop=on';
-- Chỉ dùng để quan sát behavior trong môi trường test, không kết luận vội.
```


---

## Frontend (FE)
> **Mục tiêu phỏng vấn:** API layer gọn, xử lý lỗi/ retry có kiểm soát, UI không sập.

**API client wrapper (fetch + retry)**
```ts
async function api(path: string, init: RequestInit = {}, retries=1){
  const r = await fetch(path, { ...init, headers: { 'Content-Type':'application/json', ...(init.headers||{}) } });
  if(!r.ok && retries>0 && r.status >=500){
    await new Promise(res=>setTimeout(res, 300));
    return api(path, init, retries-1);
  }
  return r;
}
```

**Error Boundary**
```tsx
class ErrorBoundary extends React.Component<any, {hasError:boolean}>{
  constructor(props:any){ super(props); this.state={hasError:false}; }
  static getDerivedStateFromError(){ return {hasError:true}; }
  render(){ return this.state.hasError ? <p role="alert">Something went wrong.</p> : this.props.children; }
}
```


---

## English for Interview (skills only)
Explain an API to a non-technical stakeholder

---

### Activity
- Viết trích đoạn OpenAPI cho `/posts` và `/posts/{id}`.
- Tạo PostResource + ví dụ response.
- Benchmark một truy vấn sau khi bật/tắt partition prune (nếu có).

### Practice Tasks
- Viết OpenAPI trích đoạn + PostResource.
- FE: tạo API wrapper + ErrorBoundary demo.
- (Supplement) Thử 1 trong 3 tối ưu (partition/tmp table/optimizer_switch) và ghi nhận trước/sau.

### Acceptance Criteria
- Spec & Resource đồng bộ (field khớp, kiểu dữ liệu chuẩn).
- FE wrapper hoạt động; ErrorBoundary hiển thị an toàn.
- Ghi chú đo đạc bổ sung rõ ràng, không thay thế chủ đề BE chính.

### Docs & References
- OpenAPI: https://spec.openapis.org/oas/latest.html
- Laravel Resources: https://laravel.com/docs/eloquent-resources
- React Error Boundaries: https://react.dev/reference/react/Component#error-handling