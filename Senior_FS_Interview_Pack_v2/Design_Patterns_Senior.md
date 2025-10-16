# Senior Fullstack Design Patterns — Laravel (PHP) & React

> Mục tiêu: hiểu **khi nào nên dùng**, **trade-offs**, và có **code snippet** thực chiến.

---

## 0) Nguyên tắc nền (Senior Mindset)
- **Start with the problem & constraints** (latency targets, data size, throughput).
- **Prefer composition over inheritance**; interfaces/traits where sensible.
- **Measure first** (profilers, EXPLAIN, tracing) → rồi tối ưu.
- **Operational fitness**: logs, metrics, alerts; rollback & feature flags.
- **ADR (Architecture Decision Record)** cho các quyết định lớn.

---

## 1) Strategy Pattern
**Khi dùng:** hoán đổi thuật toán/policy runtime (ví dụ pricing, rate limit, retry policy).

**Laravel (PHP)**
```php
interface PricingStrategy { public function price(float $base): float; }

class PeakPricing implements PricingStrategy {
    public function __construct(private float $factor) {}
    public function price(float $base): float { return $base * $this->factor; }
}
class FlatPricing implements PricingStrategy {
    public function price(float $base): float { return $base; }
}

class PricingContext {
    public function __construct(private PricingStrategy $strategy) {}
    public function quote(float $base): float { return $this->strategy->price($base); }
}

// Usage (e.g. via service container binding by config/time)
$ctx = new PricingContext(match ($hour) {
   7,8,17,18 => new PeakPricing(1.25),
   default   => new FlatPricing()
});
$total = $ctx->quote(100);
```

**Trade-offs:** nhiều lớp nhỏ → cần DI/IoC để gọn.

---

## 2) Decorator Pattern (caching/logging/metrics)
**Khi dùng:** thêm hành vi (cache, log) mà không đổi code core.

```php
interface ReportRepo { public function findById(int $id): array; }

class DbReportRepo implements ReportRepo {
    public function findById(int $id): array {
        return DB::table('reports')->where('id',$id)->firstOrFail();
    }
}

class CachedReportRepo implements ReportRepo {
    public function __construct(private ReportRepo $inner){}
    public function findById(int $id): array {
        return Cache::remember("report:$id", 600, fn()=> $this->inner->findById($id));
    }
}

// Bind in container: ReportRepo => CachedReportRepo(DbReportRepo)
```

**Pitfall:** invalidation khó → thêm tag cache hoặc TTL ngắn với write-through.

---

## 3) Adapter Pattern (Partner API)
**Khi dùng:** chuẩn hoá nhiều provider về cùng interface.

```php
interface PartnerClient { public function send(array $payload): Response; }

class SamsaraAdapter implements PartnerClient {
  public function __construct(private HttpClient $http){}
  public function send(array $payload): Response {
    // map nội bộ -> samsara schema
    return $this->http->post('/samsara/tx', $payload);
  }
}

class PartnerService {
  public function __construct(private PartnerClient $client){}
  public function pushTx(Tx $tx){ return $this->client->send($tx->toPartner()); }
}
```

**Trade-offs:** thêm tầng map → cần test hợp đồng (contract test).

---

## 4) Chain of Responsibility (Validation Pipeline)
**Khi dùng:** nhiều bước validate/transform tùy chọn.

```php
interface Stage { public function handle(array $ctx, callable $next): array; }

class RegexStage implements Stage {
  public function handle(array $ctx, callable $next): array {
    if(!preg_match('/^[a-zA-Z0-9_\\-@.:\\/]+$/', $ctx['tank_id'] ?? '')) {
      throw new InvalidArgumentException('invalid tank_id');
    }
    return $next($ctx);
  }
}

class NormalizeStage implements Stage {
  public function handle(array $ctx, callable $next): array {
    $ctx['name'] = mb_convert_case($ctx['name'] ?? '', MB_CASE_TITLE);
    return $next($ctx);
  }
}

function pipeline(array $stages, array $payload): array {
  $next = array_reduce(array_reverse($stages), fn($next,$stage) =>
      fn($ctx) => $stage->handle($ctx, $next),
      fn($ctx) => $ctx);
  return $next($payload);
}

// usage
$ctx = pipeline([new RegexStage, new NormalizeStage], $row);
```

**Pitfall:** quá nhiều stage → khó debug → thêm trace id + structured logs.

---

## 5) Circuit Breaker + Retry with Jitter
**Khi dùng:** bảo vệ downstream, tránh retry storm.

```php
final class CircuitBreaker {
  private int $failures = 0; private string $state = 'CLOSED'; private int $openedAt = 0;
  public function call(callable $fn) {
    if ($this->state === 'OPEN' && (time()-$this->openedAt) < 10) throw new RuntimeException('circuit open');
    try { $res = $fn(); $this->reset(); return $res; }
    catch(\Throwable $e){ $this->recordFailure(); throw $e; }
  }
  private function recordFailure(){ if(++$this->failures >= 5){ $this->state='OPEN'; $this->openedAt=time(); } }
  private function reset(){ $this->failures=0; $this->state='CLOSED'; }
}
$cb = new CircuitBreaker();
$resp = retry(5, fn()=> $cb->call(fn()=> Http::post($url,$payload)), sleepMilliseconds: fn($i)=>random_int(100,300)*$i);
```

**Trade-offs:** choose thresholds cẩn thận; thêm metrics để quan sát.

---

## 6) Outbox Pattern (Consistency DB → Queue)
**Khi dùng:** đảm bảo sự kiện rời DB được phát chính xác.

```php
DB::transaction(function() use ($order) {
  // 1) write domain state
  DB::table('orders')->insert($order);

  // 2) write outbox record in same tx
  DB::table('outbox')->insert([
    'type'=>'OrderCreated',
    'payload'=>json_encode($order),
    'created_at'=>now(),
  ]);
});
// A scheduled worker reads 'outbox' and publishes to SQS, marking as sent.
```

**Pitfall:** dọn dẹp outbox, idempotency khi publish → add unique key + sent_at.

---

## 7) Saga / Process Manager (Long-running Transaction)
**Khi dùng:** nhiều bước across services (reserve→charge→confirm).

- Orchestrator lưu **state** + **compensations**.
- SQS/FIFO nhóm theo `OrderId` để giữ thứ tự.

**Compensation Example**
```php
try {
  $inv = reserveInventory($order);
  $pay = chargePayment($order);
  confirmOrder($order);
} catch (\Throwable $e) {
  if(isset($pay)) refundPayment($pay);
  if(isset($inv)) releaseInventory($inv);
  throw $e;
}
```

---

## 8) CQRS (Query/Command Responsibility Segregation)
**Khi dùng:** tải đọc rất lớn, mẫu dữ liệu đọc khác ghi.

- **Command**: thay đổi trạng thái (validate, publish events).
- **Query**: đọc từ **read model** (denormalized), có thể là ClickHouse/ES.

```php
final class CreateUserCommand { public function __construct(public string $name, public string $email){} }
final class CreateUserHandler {
  public function handle(CreateUserCommand $c){
    $id = DB::table('users')->insertGetId(['name'=>$c->name,'email'=>$c->email]);
    DB::table('outbox')->insert(['type'=>'UserCreated','payload'=>json_encode(['id'=>$id])]);
  }
}
```

**Trade-offs:** phức tạp hơn; nhất quán cuối cùng (eventually consistent).

---

## 9) Repository + Specification
**Khi dùng:** tách domain filter khỏi persistence.

```php
interface Spec { public function apply($q): void; }
class ActiveSpec implements Spec { public function apply($q){ $q->where('active',1); } }
class CreatedFromSpec implements Spec { public function __construct(private string $from){} public function apply($q){ $q->where('created_at','>=',$this->from); } }

class UserRepo {
  public function queryBy(Spec ...$specs){
    $q = DB::table('users');
    foreach($specs as $s) $s->apply($q);
    return $q->get();
  }
}
```

**Trade-offs:** over-engineering nếu domain đơn giản.

---

## 10) Hexagonal (Ports & Adapters)
- **Ports:** interface cho outside world (e.g., PartnerClient, Mailer).
- **Adapters:** implement cụ thể (SamsaraAdapter).
- **Benefit:** test dễ, đổi provider nhanh, domain thuần.

---

## 11) Frontend Patterns

### 11.1 Compound Components
```tsx
function Tabs({ children }) { /* context provider */ return <div>{children}</div> }
Tabs.List = ({ children }) => <div className="flex gap-2">{children}</div>;
Tabs.Trigger = ({ value, children }) => /* read ctx, set active */ <button>{children}</button>;
Tabs.Content = ({ value, children }) => /* show if active */ <div>{children}</div>;
```

### 11.2 Custom Hooks as Strategy
```tsx
type Fetcher = (q: string) => Promise<any[]>;
function useSearch(fetcher: Fetcher) {
  const [q,setQ] = useState(''); const [data,setData]=useState<any[]>([]);
  useEffect(()=>{ /* debounce + cancel, then fetcher(q) */ },[q]);
  return { q, setQ, data };
}
// swap fetcher for different backends without changing UI
```

### 11.3 Error Boundary & Suspense
- Boundaries cho từng vùng, fallback UI; streaming với RSC.
- Data layer: TanStack Query for retries, background refresh.

---

## 12) Security Patterns (OWASP)
- Input validation, output encoding, CSRF/CORS.
- AuthZ: policies/roles/permissions.
- Secrets: env managers, rotate tokens; signed URLs.
- Logging PII: tokenize/anonymize; data retention policies.

---

## 13) ADR Template (rút gọn)
```
# ADR-001: Export Pipeline Choice
Context: Daily 50M rows, Excel requested by finance.
Decision: CSV streaming for ad-hoc; XLSX async via queue + S3 + signed URL.
Consequences: +TTFB, +stability; -more infra components; need DLQ & retries.
Metrics: TTFB<2s, p95 chunk<1.5s, DLQ<0.1%.
```