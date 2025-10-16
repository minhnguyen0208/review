# Day 13 — Interview Prep Day 13

_Generated on 2025-10-01 03:52._

### Plan Notes
- **EN (CSV)**: Coding Test

## Backend (BE)

#### Auth+Sanctum
```php
$token = $user->createToken('web')->plainTextToken;
Route::middleware('auth:sanctum')->get('/profile', fn(Request $r)=>$r->user());
```

#### S3 Multipart
```php
// Server: issue multipart + presigned part URLs; Client uploads parts; Server completes.
```

#### Design Pattern — Factory
Use Factory to isolate object creation.
Pros: testability, SRP. Cons: extra indirection.

#### MySQL — Partition by Date
Range partition on date to prune scans; ensure partition columns in PK/unique.

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
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 13 with an emphasis on reliability and performance.*
- **2 Tech Q&A to rehearse**
  1) *Explain your approach to idempotent APIs and retries.*
  2) *How do you design pagination and prevent N+1 issues?*


### Interview Focus
- Pattern: **Factory** — why/when & trade-offs.
- MySQL: **Partition by Date** — demonstrate with EXPLAIN or schema.
- BE deep-dive: Auth+Sanctum, S3 Multipart.
- FE deep-dive: ErrorBoundary, Suspense+Split.
- **Common pitfalls**:
- Lock TTL too short causes duplicate work.
- Client and server validation drift.
- Eager loading without constraints can explode memory.
- Optimistic update without rollback corrupts UI state.


### Practice Tasks
- Implement: **Interview Prep Day 13** feature end-to-end (BE & FE), commit with README explaining design choices.
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