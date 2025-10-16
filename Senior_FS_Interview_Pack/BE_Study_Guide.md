# Backend Study Guide — Laravel, MySQL, Redis, SQS

## 1) Laravel Core & Patterns
### Topics
- Eloquent nâng cao: eager loading (`with`, `loadMissing`), subquery select, scopes, `when()`
- Pagination hiệu năng: `chunkById`, keyset pagination (seek method), `cursor`
- Validation & FormRequest; DTO light (request->validated())
- Auth: Sanctum/JWT, policies, guards
- API design: versioning, error envelope, pagination, rate limit
- Testing (Pest): unit vs feature, database transactions, factories

### Code Snippet — Keyset Pagination (composite)
```php
$cursorTs = $request->query('cursor_ts');
$cursorId = $request->query('cursor_id');

$query = DB::table('transactions')
    ->when($cursorTs && $cursorId, function($q) use ($cursorTs, $cursorId) {
        $q->where(function($w) use ($cursorTs, $cursorId) {
            $w->where('created_at', '>', $cursorTs)
              ->orWhere(function($w) use ($cursorTs, $cursorId) {
                  $w->where('created_at', '=', $cursorTs)->where('id', '>', $cursorId);
              });
        });
    })
    ->orderBy('created_at')->orderBy('id')
    ->limit(101);

$rows = $query->get();
$next = null;
if ($rows->count() > 100) {
    $last = $rows->pop();
    $next = ['cursor_ts' => $last->created_at, 'cursor_id' => $last->id];
}
return response()->json(['data' => $rows, 'next' => $next]);
```

---

## 2) MySQL 8 Performance
### Topics
- Composite & covering indexes; `EXPLAIN`; avoid `SELECT *`
- Window functions (LAG/ROW_NUMBER) cho báo cáo
- JSON + generated columns (persisted) để index JSON path
- Transactions & isolation level (RR vs RC); deadlock retry
- Histograms & invisible indexes để testing

### Code Snippet — Deadlock-safe Batch Upsert
```php
function upsertBatch(string $table, array $rows, array $keys, array $updates) {
    retry(5, function() use ($table,$rows,$keys,$updates) {
        DB::transaction(function() use ($table,$rows,$keys,$updates) {
            DB::table($table)->upsert($rows, $keys, $updates);
        }, 3); // 3s timeout, short transactions
    }, sleepMilliseconds: fn($a) => 100 * (2 ** $a)); // exp backoff
}
```

---

## 3) Redis — Cache, Locks, Rate Limits
### Topics
- Cache-aside vs write-through; cache tags; TTL & stale-while-revalidate
- Distributed locks: `Cache::lock` + fencing tokens
- Token bucket rate limit per tenant/partner

### Code Snippet — Lock + Fencing Token
```php
$token = Redis::incr('fence:resource:42');
Cache::lock('lock:resource:42', 10)->block(5, function() use ($token) {
    DB::transaction(function() use ($token) {
        $row = DB::table('resource')->lockForUpdate()->where('id', 42)->first();
        if ($token < $row->last_token) {
            throw new RuntimeException('stale-writer');
        }
        DB::table('resource')->where('id',42)->update(['value'=>'new','last_token'=>$token]);
    });
});
```

---

## 4) Queues — SQS, Idempotency, DLQ
### Topics
- Visibility timeout > p95 job time (2x)
- Idempotency keys (natural id) bằng Redis/DB unique + upsert
- DLQ + replay UI; exponential backoff + jitter
- Batch processing & concurrency control

### Code Snippet — Idempotent Job Guard
```php
public function handle() {
    $key = "job:push:{$this->naturalId}";
    $lock = Cache::lock($key, 120);
    if (!$lock->get()) return;

    try {
        // Idempotent write (PUT semantics)
        DB::table('partner_payloads')->upsert([
            'natural_id' => $this->naturalId,
            'payload' => json_encode($this->data),
            'pushed_at' => now(),
        ], ['natural_id'], ['payload','pushed_at']);
    } finally {
        optional($lock)->release();
    }
}
```

---

## 5) Massive Export/Import
### CSV Streaming Export
```php
Route::get('/export/csv', function () {
    $headers = [
        "Content-Type" => "text/csv",
        "Content-Disposition" => "attachment; filename=transactions.csv",
    ];
    $cb = function() {
        $out = fopen('php://output', 'w');
        fputcsv($out, ['id','amount','created_at']);
        DB::table('transactions')->select('id','amount','created_at')
            ->orderBy('id')
            ->chunkById(5000, function($rows) use ($out) {
                foreach ($rows as $r) {
                    fputcsv($out, [$r->id, $r->amount, $r->created_at]);
                }
                @ob_flush(); @flush();
            }, 'id');
        fclose($out);
    };
    return response()->stream($cb, 200, $headers);
});
```

### Import with Upsert & Error Log
```php
use League\Csv\Reader;
$csv = Reader::createFromPath($path, 'r'); $csv->setHeaderOffset(0);
$batch = [];
foreach ($csv->getRecords() as $i => $row) {
    if (!preg_match('/^[a-zA-Z0-9_\\-@.:\\/]+$/', $row['tank_id'] ?? '')) {
        Storage::append('errors.log', "line=$i invalid tank_id");
        continue;
    }
    $batch[] = [
        'type' => 'anova', 'monitor_id' => $row['tank_id'],
        'name' => $row['tank_name'] ?? '', 'data' => json_encode($row, JSON_UNESCAPED_UNICODE)
    ];
    if (count($batch) >= 2000) {
        upsertBatch('tankmonitors', $batch, ['type','monitor_id'], ['name','data']);
        $batch = [];
    }
}
if ($batch) upsertBatch('tankmonitors', $batch, ['type','monitor_id'], ['name','data']);
```

---

## 6) Webhooks at Scale (Verify + Replay)
```php
$sig = request()->header('X-Signature');
$ts  = (int) request()->header('X-Timestamp');
abort_if(abs(time()-$ts) > 300, 401);
$calc = base64_encode(hash_hmac('sha256', $ts.'.'.request()->getContent(), $secret, true));
abort_if(!hash_equals($sig,$calc), 401);
// Dedup
$eventId = data_get(request()->json(), 'event_id');
if (!Cache::add("evt:$eventId", 1, 300)) return response()->noContent();
dispatch(new ProcessWebhook(request()->all()));
```

---

## 7) Observability
- Structured logs (trace_id), metrics (p95 latency, retries, DLQ depth), APM traces
- Health checks: DB, Redis, queue lag, rate limiter saturation