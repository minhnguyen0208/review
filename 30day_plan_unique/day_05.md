# Day 05 — Interview Prep Day 5

_Generated on 2025-10-01 03:52._

### Plan Notes
- **EN (CSV)**: Caching & Redis

## Backend (BE)

#### S3 Multipart
```php
// Server: issue multipart + presigned part URLs; Client uploads parts; Server completes.
```

#### Txn+Outbox
```php
DB::transaction(function(){
  $order = Order::create([...]);
  Outbox::create(['event'=>'OrderCreated','payload'=>json_encode($order)]);
});
// worker ships Outbox to broker
```

#### Cache+Lock
```php
$value = Cache::remember('stats:daily', now()->addMinutes(15), fn()=>computeStats());
$result = Cache::lock('refresh_token_api_'.$companyId, 60)->block(10, function () use ($refreshToken) {
  return refreshTokenFlow($refreshToken);
});
```

#### Design Pattern — CQRS
Separate write (normalized) and read (denormalized) models.

#### MySQL — Idempotency Key
Persist request hash + response to dedupe retries safely.

## Frontend (FE)

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

#### ErrorBoundary
```tsx
class ErrorBoundary extends React.Component<{children:React.ReactNode},{hasError:boolean}>{/* ... */} 
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 5 with an emphasis on reliability and performance.*
- **2 Tech Q&A to rehearse**
  1) *Explain your approach to idempotent APIs and retries.*
  2) *How do you design pagination and prevent N+1 issues?*


### Interview Focus
- Pattern: **CQRS** — why/when & trade-offs.
- MySQL: **Idempotency Key** — demonstrate with EXPLAIN or schema.
- BE deep-dive: S3 Multipart, Txn+Outbox, Cache+Lock.
- FE deep-dive: RHF+Zod, ErrorBoundary.
- **Common pitfalls**:
- Optimistic update without rollback corrupts UI state.
- Client and server validation drift.
- Transaction scope too wide increases deadlock risk.
- Pagination without deterministic order causes duplicates.


### Practice Tasks
- Implement: **Interview Prep Day 5** feature end-to-end (BE & FE), commit with README explaining design choices.
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