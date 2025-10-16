# Day 07 — Interview Prep Day 7

_Generated on 2025-10-01 04:00._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### S3 Multipart
```php
// 1) init multipart -> 2) generate signed part URLs -> 3) complete
```

#### Indexing+EXPLAIN
#### MySQL — Covering Index (Stop SELECT *)
```sql
-- Bad
SELECT * FROM posts WHERE author_id = ? ORDER BY created_at DESC LIMIT 20;

-- Good: only needed cols + composite index
CREATE INDEX idx_posts_author_created ON posts(author_id, created_at DESC);

SELECT id, title, created_at
FROM posts
WHERE author_id = ?
ORDER BY created_at DESC
LIMIT 20;
```
**Why**: "Using index" => tránh back-to-table lookup → giảm IO, latency.


#### MySQL — Đọc EXPLAIN (example)
```sql
EXPLAIN SELECT id,title,created_at FROM posts
WHERE author_id = 42
ORDER BY created_at DESC
LIMIT 20;
```
**Key fields**  
- `type`: `ref/range` tốt; `ALL` xấu (full scan)  
- `rows`: ước lượng số row đọc  
- `filtered`: % row qua filter  
- `Extra`: "Using index" (covering), "Using filesort" (cần index theo ORDER BY)

**Fix "Using filesort"**: tạo index `(author_id, created_at DESC)`.


#### Design Pattern — Factory
#### Design Pattern — Factory (Channel Notifier)
```php
interface Notifier { public function send(string $to, string $msg): void; }
class EmailNotifier implements Notifier { public function send(string $to, string $msg){ Mail::to($to)->send(new GenericMail($msg)); } }
class SmsNotifier implements Notifier { public function send(string $to, string $msg){ app('twilio')->messages->create($to, ['from'=>config('twilio.from'),'body'=>$msg]); } }

class NotifierFactory {
  public static function make(string $channel): Notifier {
    return match($channel){
      'sms'   => new SmsNotifier(),
      'email' => new EmailNotifier(),
      default => throw new InvalidArgumentException('Unsupported channel')
    };
  }
}

// Usage (Service layer)
$notifier = NotifierFactory::make(config('notify.channel','email'));
$notifier->send($user->email, 'Welcome to our product!');
```
**When to use**: nhiều biến thể tạo object; **Pitfall**: over-engineer nếu chỉ 1-2 loại.


#### MySQL — EXPLAIN
#### MySQL — Đọc EXPLAIN (example)
```sql
EXPLAIN SELECT id,title,created_at FROM posts
WHERE author_id = 42
ORDER BY created_at DESC
LIMIT 20;
```
**Key fields**  
- `type`: `ref/range` tốt; `ALL` xấu (full scan)  
- `rows`: ước lượng số row đọc  
- `filtered`: % row qua filter  
- `Extra`: "Using index" (covering), "Using filesort" (cần index theo ORDER BY)

**Fix "Using filesort"**: tạo index `(author_id, created_at DESC)`.


## Frontend (FE)

#### Error Boundary
```tsx
class ErrorBoundary extends React.Component<{children:React.ReactNode},{hasError:boolean}>{
  state={hasError:false}; componentDidCatch(err:any){ this.setState({hasError:true}); }
  render(){ return this.state.hasError? <p>Something went wrong.</p> : this.props.children; }
}
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 7 with emphasis on reliability and performance.*
- **Tech Qs**:  
  1) *How do you ensure idempotency for mutations?*  
  2) *Explain a query optimization you shipped and its impact.*


### Interview Focus
- Patterns: Factory
- MySQL: EXPLAIN
- Pitfalls:
- Avoid SELECT *; keep covering index effective.
- Set consistent ordering for pagination.
- Add jitter to retries to avoid herd effect.
- Keep transactions short; avoid long-held locks.

**Case Study D7**: Implement idempotent create-order with DB upsert + Redis lock; return 201 on first call, 200 cached for retries.

### Practice Tasks
- Build the feature end-to-end; write why each design choice fits interview expectations.

### Acceptance Criteria
- BE: đúng schema/status; test Feature; log structured.
- FE: list/mutation OK; handle loading/error/empty; mutation rollback tốt.
- DB: có EXPLAIN + index/partition đề xuất.


### Docs & References
- [AWS S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/mpuoverview.html)
- [Database](https://dev.mysql.com/doc/)

### Checklist
- [ ] Code BE+FE end-to-end
- [ ] 1 Feature test (BE) + 1 RTL/Playwright (FE)
- [ ] Ghi chú EXPLAIN & tối ưu
- [ ] 2 câu trả lời kỹ thuật (EN)


### Reflection
- What trade-off today would you defend in an interview? Why?