# Day 25 — Interview Prep Day 25

_Generated on 2025-10-01 03:52._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Cache+Lock
```php
$value = Cache::remember('stats:daily', now()->addMinutes(15), fn()=>computeStats());
$result = Cache::lock('refresh_token_api_'.$companyId, 60)->block(10, function () use ($refreshToken) {
  return refreshTokenFlow($refreshToken);
});
```

#### Eloquent Perf
```php
// Eager load with constraints
$posts = Post::with(['comments'=>fn($q)=>$q->latest()->limit(5)])->paginate(20);

// Subquery for latest comment timestamp
$posts = Post::addSelect(['latest_comment_at'=>Comment::select('created_at')
  ->whereColumn('comments.post_id','posts.id')->latest()->limit(1)])->get();

// Counts
$top = Post::withCount(['comments','likes'])->orderByDesc('comments_count')->limit(50)->get();
```

#### Design Pattern — Factory
Use Factory to isolate object creation.
Pros: testability, SRP. Cons: extra indirection.

#### MySQL — Idempotency Key
Persist request hash + response to dedupe retries safely.

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
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 25 with an emphasis on reliability and performance.*
- **2 Tech Q&A to rehearse**
  1) *Explain your approach to idempotent APIs and retries.*
  2) *How do you design pagination and prevent N+1 issues?*


### Interview Focus
- Pattern: **Factory** — why/when & trade-offs.
- MySQL: **Idempotency Key** — demonstrate with EXPLAIN or schema.
- BE deep-dive: Cache+Lock, Eloquent Perf.
- FE deep-dive: A11y Modal, RHF+Zod.
- **Common pitfalls**:
- Transaction scope too wide increases deadlock risk.
- Lock TTL too short causes duplicate work.
- Missing error boundaries hides failures.
- SELECT * prevents covering index.


### Practice Tasks
- Implement: **Interview Prep Day 25** feature end-to-end (BE & FE), commit with README explaining design choices.
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