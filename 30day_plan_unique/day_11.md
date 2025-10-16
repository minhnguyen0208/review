# Day 11 — Interview Prep Day 11

_Generated on 2025-10-01 03:52._

### Plan Notes
- **EN (CSV)**: Performance Lab

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

#### Design Pattern — CQRS
Separate write (normalized) and read (denormalized) models.

#### MySQL — Covering Index
Select only needed columns; align WHERE + ORDER BY with composite index.

## Frontend (FE)

#### A11y Modal
```tsx
// focus trap + ESC to close
```

#### RHF+Zod
```tsx
import { useForm } from 'react-hook-form';
import { z } from 'zod'; import { zodResolver } from '@hookform/resolvers/zod';
const schema = z.object({ email:z.string().email(), password:z.string().min(8) });
type Form = z.infer<typeof schema>;
export default function Login(){
  const { register, handleSubmit, formState:{errors} } = useForm<Form>({ resolver: zodResolver(schema) });
  return (/* form fields + errors.email?.message */);
}
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 11 with an emphasis on reliability and performance.*
- **2 Tech Q&A to rehearse**
  1) *Explain your approach to idempotent APIs and retries.*
  2) *How do you design pagination and prevent N+1 issues?*


### Interview Focus
- Pattern: **CQRS** — why/when & trade-offs.
- MySQL: **Covering Index** — demonstrate with EXPLAIN or schema.
- BE deep-dive: Testing(Feature), API+Validation.
- FE deep-dive: A11y Modal, RHF+Zod.
- **Common pitfalls**:
- Optimistic update without rollback corrupts UI state.
- Not normalizing error schema complicates FE.
- Pagination without deterministic order causes duplicates.
- Retry without jitter can thundering-herd.


### Practice Tasks
- Implement: **Interview Prep Day 11** feature end-to-end (BE & FE), commit with README explaining design choices.
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