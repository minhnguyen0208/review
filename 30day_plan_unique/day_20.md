# Day 20 — Interview Prep Day 20

_Generated on 2025-10-01 03:52._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Testing(Feature)
```php
public function test_user_can_login(){
  $u=User::factory()->create(['password'=>bcrypt('secret')]);
  $this->post('/login',['email'=>$u->email,'password'=>'secret'])->assertRedirect('/dashboard');
}
```

#### API+Validation
```php
// routes/api.php
Route::apiResource('posts', PostController::class);

// App/Http/Controllers/PostController.php
class PostController extends Controller {
  public function index(Request $r){
    $q = Post::query()
      ->when($r->q, fn($q,$v)=>$q->where('title','like',"%$v%"))
      ->when($r->author_id, fn($q,$v)=>$q->where('author_id',$v))
      ->orderByDesc('created_at');
    return PostResource::collection($q->paginate($r->integer('per_page',20)));
  }
  public function store(StorePostRequest $req){
    $post = Post::create($req->validated());
    return (new PostResource($post))->response()->setStatusCode(201);
  }
}
```

#### Txn+Outbox
```php
DB::transaction(function(){
  $order = Order::create([...]);
  Outbox::create(['event'=>'OrderCreated','payload'=>json_encode($order)]);
});
// worker ships Outbox to broker
```

#### Design Pattern — Strategy
Swap algorithms at runtime via interface.
Pros: open/closed. Cons: more classes.

#### MySQL — Idempotency Key
Persist request hash + response to dedupe retries safely.

## Frontend (FE)

#### ErrorBoundary
```tsx
class ErrorBoundary extends React.Component<{children:React.ReactNode},{hasError:boolean}>{/* ... */} 
```

#### Suspense+Split
```tsx
import { lazy, Suspense } from 'react';
const Reports = lazy(()=> import('./Reports'));
export default function App(){ return <Suspense fallback={<div>Loading…</div>}><Reports/></Suspense>; }
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 20 with an emphasis on reliability and performance.*
- **2 Tech Q&A to rehearse**
  1) *Explain your approach to idempotent APIs and retries.*
  2) *How do you design pagination and prevent N+1 issues?*


### Interview Focus
- Pattern: **Strategy** — why/when & trade-offs.
- MySQL: **Idempotency Key** — demonstrate with EXPLAIN or schema.
- BE deep-dive: Testing(Feature), API+Validation, Txn+Outbox.
- FE deep-dive: ErrorBoundary, Suspense+Split.
- **Common pitfalls**:
- Pagination without deterministic order causes duplicates.
- Missing error boundaries hides failures.
- Lock TTL too short causes duplicate work.
- Eager loading without constraints can explode memory.


### Practice Tasks
- Implement: **Interview Prep Day 20** feature end-to-end (BE & FE), commit with README explaining design choices.
- Add EXPLAIN notes + index/partition proposal if needed.
- Write 1 Feature test (BE) + 1 RTL or Playwright test (FE).


### Acceptance Criteria
- BE endpoints return correct schema & status (201/204/404/422) with tests.
- FE states: loading/error/empty handled; mutation has optimistic update + rollback.
- Query hot-path has EXPLAIN screenshot/notes + index plan.


### Docs & References
- [Cache](https://laravel.com/docs/cache)
- [Database](https://dev.mysql.com/doc/)
- [Docker](https://docs.docker.com/)
- [Eloquent](https://laravel.com/docs/eloquent)
- [Jest](https://jestjs.io/docs/getting-started)
- [Laravel](https://laravel.com/docs)
- [OWASP](https://owasp.org/www-project-top-ten/)
- [Playwright](https://playwright.dev/docs/intro)
- [Queues](https://laravel.com/docs/queues)
- [React](https://react.dev/learn)
- [React Query](https://tanstack.com/query/latest)
- [React Router](https://reactrouter.com/en/main)
- [Sanctum](https://laravel.com/docs/sanctum)
- [Tailwind](https://tailwindcss.com/docs)
- [TypeScript](https://www.typescriptlang.org/docs/)
- [Validation](https://laravel.com/docs/validation)

### Checklist
- [ ] BE implemented & tested (Feature/Pest)
- [ ] FE implemented & tested (RTL/Playwright)
- [ ] Performance notes (EXPLAIN/metrics) committed
- [ ] 2 interview answers recorded (EN)


### Reflection
- What would you change if you had to scale this 10x?
- Which trade-off was most impactful today?