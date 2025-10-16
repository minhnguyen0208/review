# Interview Scenarios & Solutions

## 1. Massive Export CSV XLSX: Export 50M+ transactions with low memory and without timeouts
**Architecture / Flow**

```
Option A (CSV streaming, synchronous):
FE → GET /reports/export.csv → Laravel stream response → DB keyset pagination (chunkById) → fputcsv → client
Option B (XLSX queued, asynchronous):
FE → POST /reports/exports → Queue job slices → Write to object storage (S3/local) → Signed URL → FE polls progress
```
**Key Steps**

1) Prefer CSV streaming when possible (constant memory, immediate download start)
2) Use keyset pagination on monotonic key (id or (created_at, id)) instead of OFFSET
3) For XLSX: queue + chunkSize (e.g. 5k), write to disk, return signed URL
4) Cache heavy lookups; avoid joins in hot path (build lookup maps before loop)
5) Track checkpoints in Redis (last_id, row_count); resume on retry
**Code Snippet**

```php
// routes/web.php
Route::get('/export/csv', [ReportController::class, 'streamCsv']);

// app/Http/Controllers/ReportController.php
public function streamCsv() {
    $headers = [
        "Content-Type" => "text/csv",
        "Content-Disposition" => "attachment; filename=transactions.csv",
        "Cache-Control" => "no-store, no-cache",
    ];
    $callback = function() {
        $out = fopen('php://output', 'w');
        fputcsv($out, ['id','amount','created_at']);
        $q = DB::table('transactions')->select('id','amount','created_at')->orderBy('id');
        $q->chunkById(5000, function($rows) use ($out) {
            foreach ($rows as $r) {
                fputcsv($out, [(string)$r->id, $r->amount, $r->created_at]);
            }
            @ob_flush(); @flush();
        }, 'id');
        fclose($out);
    };
    return response()->stream($callback, 200, $headers);
}
```
**Failure Modes & Mitigations**

- Timeouts at proxy: use streaming, increase upstream timeouts, or switch to async XLSX
- Duplicates when resuming: store checkpoint (last_id) in Redis; ensure WHERE id > last_id
- Memory spikes: fputcsv streaming; avoid reading whole result into memory
- Slow joins: pre-build lookup map (cache) or denormalize hot fields
**Metrics / SLOs**

- TTFD (time-to-first-byte) < 2s for CSV streaming
- p95 chunk processing time (5k rows) < 1.5s
- Error rate < 0.5%, DLQ depth < 50
**Checklist**

- [ ] Keyset pagination
- [ ] Streaming response or queued XLSX
- [ ] Checkpoints in Redis
- [ ] Precomputed lookups
- [ ] Signed URL for async

---

## 2. Massive Import: Import 10M CSV rows with schema & business validation
**Architecture / Flow**

```
FE upload → Backend stores file (S3) → Light schema validation → Enqueue chunk jobs (e.g., 10k lines/job)
Worker: parse → transform → upsert in batches → record progress & errors → Final summary for download
```
**Key Steps**

1) Pre-validate headers & sample rows; reject early
2) Read stream with League\Csv; split into batches (1–5k)
3) Use DB::upsert with natural key (e.g., type+monitor_id) to ensure idempotency
4) Wrap each batch in a short transaction; retry on deadlock
5) Log per-row errors separately (S3/DB table) with line number
**Code Snippet**

```php
// Upsert example
DB::table('tankmonitors')->upsert($batch, ['type','monitor_id'], ['name','product','location','data']);
```
**Failure Modes & Mitigations**

- Deadlocks: short transactions; exponential backoff retry
- Duplicates: natural key + upsert; pre-hash for dedup
- Memory: process streaming; avoid loading entire file
- Partial failure: store bad rows separately; allow targeted re-import
**Metrics / SLOs**

- Throughput ≥ 50k rows/min/worker
- Deadlock retry success > 99%
- Error rows < 0.5%
**Checklist**

- [ ] Header/schema validation
- [ ] Batching 1–5k
- [ ] Upsert with natural key
- [ ] Retry on deadlock
- [ ] Error report file

---

## 3. SQS Idempotency Visibility: Prevent duplicate processing with at-least-once delivery
**Architecture / Flow**

```
Producer → SQS (Standard/FIFO) → Worker
Worker (before processing): acquire dedup/lock key → process → set completion marker → ack
```
**Key Steps**

1) Choose visibility timeout > p95 job time (e.g., 2x p95)
2) Use Redis SETNX or Cache::lock for job:natural_id
3) Store completion marker with TTL; operations must be idempotent (upsert/PUT semantics)
4) DLQ on repeated failures; alarm on DLQ depth
**Code Snippet**

```php
// Idempotent job guard
$key = "job:push:{$naturalId}";
$lock = Cache::lock($key, 120); // TTL >= job max seconds
if ($lock->get()) {
    try {
        // ... do work (use upsert / idempotent APIs)
        Cache::put("done:$key", true, now()->addHours(6));
    } finally {
        $lock->release();
    }
}
```
**Failure Modes & Mitigations**

- Job exceeds visibility timeout → duplicates: increase timeout, add checkpoints
- Lost lock due to crash: keep critical section short; use fencing tokens for cross-service writes
- Poison messages: maxAttempts + DLQ + replay UI
**Metrics / SLOs**

- Duplicate rate ~ 0
- p95 processing time < visibility timeout / 2
- DLQ < 0.1% of total msgs
**Checklist**

- [ ] Visibility timeout tuned
- [ ] Redis lock/dedup
- [ ] Idempotent writes
- [ ] DLQ + replay

---

## 4. Webhooks at Scale: Process partner webhooks with signing, retries, and replay
**Architecture / Flow**

```
Partner → /webhooks → Verify HMAC(timestamp, body) → Enqueue → Process → Store result
Operator UI → Replay failed with exponential backoff
```
**Key Steps**

1) Verify signature and timestamp window (e.g., 5 minutes)
2) Use idempotency key from partner event_id; dedup on receipt
3) Store raw payload and result for audit & replay
4) Retry with jitter; circuit breaker if partner unstable (for outgoing webhooks)
**Code Snippet**

```php
$sig = request()->header('X-Signature');
$ts  = request()->header('X-Timestamp');
if (abs(time() - (int)$ts) > 300) abort(401);
$calc = base64_encode(hash_hmac('sha256', $ts.'.'.request()->getContent(), $secret, true));
if (!hash_equals($sig, $calc)) abort(401);
// dedup by event_id
if (Cache::add("evt:{$eventId}", 1, 300) === false) return response()->noContent();
dispatch(new ProcessWebhook($payload));
```
**Failure Modes & Mitigations**

- Replay attacks → timestamp + nonce + HMAC
- Out-of-order → sequence watermark, store and reorder in processor
- Partner spikes → rate limit, queue, DLQ
**Metrics / SLOs**

- Verification failures rate < 1%
- Median end-to-end latency < 2s
**Checklist**

- [ ] HMAC verify
- [ ] Timestamp/nonce
- [ ] Dedup by event_id
- [ ] Replay UI

---

## 5. Backpressure Resilience: Protect downstream APIs from spikes and failures
**Architecture / Flow**

```
App → RateLimiter (token bucket Redis) → HTTP client retry+jitter → Circuit breaker
```
**Key Steps**

1) Token bucket per tenant/partner
2) Exponential backoff + jitter on 429/5xx
3) Circuit breaker (open/half-open/close)
4) Shed load or buffer via queue
**Code Snippet**

```php
use Illuminate\Cache\RateLimiting\Limit;
RateLimiter::for('partner.push', fn() => Limit::perMinute(120)->by('partner'));
$resp = retry(5, fn() => Http::post($url, $payload), sleepMilliseconds: fn($i)=>random_int(100,300)*$i);
```
**Failure Modes & Mitigations**

Retry storms → add jitter and global limiter; thundering herd → ramp-up warm start.
**Metrics / SLOs**

429 rate < 2%; stable p95 latency during spikes.
**Checklist**

- [ ] Token bucket
- [ ] Backoff + jitter
- [ ] Circuit breaker

---

## 6. Keyset Pagination Composite: Stable pagination on (created_at, id)
**Architecture / Flow**

```
Client uses cursor (created_at,id) for next page
```
**Key Steps**

ORDER BY created_at,id; WHERE (created_at,id) > (cursor); LIMIT N; return new cursor
**Code Snippet**

```php
$cursor = [$lastCreatedAt, $lastId];
$rows = DB::table('tx')->where(function($q) use ($cursor) {
    [$ts,$id] = $cursor;
    $q->where('created_at','>', $ts)
      ->orWhere(function($q) use ($ts,$id){
          $q->where('created_at','=',$ts)->where('id','>',$id);
      });
})->orderBy('created_at')->orderBy('id')->limit(101)->get();
```
**Failure Modes & Mitigations**

Clock skew → include id; prefer server timestamps.
**Metrics / SLOs**

Page p95 < 150ms for 100 rows.
**Checklist**

- [ ] Composite index (created_at,id)
- [ ] Cursor encoded

---

## 7. Distributed Lock Fencing: Only newest lock holder can write
**Architecture / Flow**

```
Redis INCR fencing token; DB compares token
```
**Key Steps**

Use INCR to get token; pass with writes; DB rejects stale tokens (compare-and-set).
**Code Snippet**

```php
$token = Redis::incr('fence:resourceX');
DB::transaction(function() use ($token) {
    $row = DB::table('res')->lockForUpdate()->where('id',1)->first();
    if ($token < $row->last_token) { throw new \RuntimeException('stale'); }
    DB::table('res')->where('id',1)->update(['data'=>'...', 'last_token'=>$token]);
});
```
**Failure Modes & Mitigations**

Stale holder resumes → DB compare prevents lost update.
**Metrics / SLOs**

Zero lost updates; minimal overhead.
**Checklist**

- [ ] INCR token
- [ ] last_token CAS

---

## 8. Near Real Time Analytics: Dashboards refreshed within 10s via CDC
**Architecture / Flow**

```
MySQL binlog → Debezium/Kafka → ClickHouse MV → FE charts
```
**Key Steps**

Capture, transform, materialize; serve from OLAP, not OLTP.
**Code Snippet**

```php
N/A (infra).
```
**Failure Modes & Mitigations**

Lag spikes → monitor; schema drift handling.
**Metrics / SLOs**

Lag p95 < 10s; OLTP unaffected.
**Checklist**

- [ ] CDC connector
- [ ] MV
- [ ] Lag alerts

---

## 9. Multi Tenant SaaS: Shared DB with tenant_id isolation
**Architecture / Flow**

```
Global scope/policy; composite indexes; namespaced caches
```
**Key Steps**

Filter by tenant_id, enforce in repo; tenant namespaces; quotas.
**Code Snippet**

```php
static::addGlobalScope('tenant', function(Builder $b){
    $b->where('tenant_id', Tenant::current());
});
```
**Failure Modes & Mitigations**

Missing scope → leaks; add tests & linters.
**Metrics / SLOs**

Zero cross-tenant incidents.
**Checklist**

- [ ] Scope/policy
- [ ] Indexes
- [ ] Namespaced cache

---

## 10. React RSC Streaming: Server components + client islands
**Architecture / Flow**

```
Server fetch; Suspense; stream to client
```
**Key Steps**

Use RSC for data; small client bundles; Suspense boundaries.
**Code Snippet**

```php
export default async function Page() {
  const data = await getServerData();
  return (<>
    <Stats data={data} /> {/* server */}
    <ClientChart data={data} /> {/* use client */}
  </>);
}
```
**Failure Modes & Mitigations**

Boundary mistakes; auth/session handling.
**Metrics / SLOs**

Bundle ↓ >=30%; TTFB improved.
**Checklist**

- [ ] Server data
- [ ] Suspense
- [ ] Client islands

---
