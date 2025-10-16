# Day 18 — Interview Prep Day 18

_Generated on 2025-10-01 03:52._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Indexing+EXPLAIN
```sql
CREATE INDEX idx_posts_author_created ON posts(author_id, created_at DESC);
EXPLAIN SELECT id,title,created_at FROM posts WHERE author_id=? ORDER BY created_at DESC LIMIT 20;
```

#### Queue+Retry+DLQ
```php
class SyncJob implements ShouldQueue {
  use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;
  public $tries=3; public $backoff=[60,300,900];
  public function handle(){ Partner::push(); }
}
// DLQ/Horizon to inspect failures
```

#### Design Pattern — Event Sourcing
Persist events; derive state via projections; great for audit.

#### MySQL — Partition by Date
Range partition on date to prune scans; ensure partition columns in PK/unique.

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

#### RQ Mutation Optimistic
```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';
function useUpdatePost(){
  const qc = useQueryClient();
  return useMutation({
    mutationFn: (p:any)=> fetch('/api/posts/'+p.id,{method:'PUT',headers:{'Content-Type':'application/json'},body:JSON.stringify(p)}).then(r=>r.json()),
    onMutate: async (p)=>{ await qc.cancelQueries({queryKey:['posts']}); const prev=qc.getQueryData<any[]>(['posts']);
      qc.setQueryData<any[]>(['posts'], old=> old?.map(x=>x.id===p.id?{...x,...p}:x) ?? []); return {prev}; },
    onError: (_e,_p,ctx)=> ctx?.prev && qc.setQueryData(['posts'], ctx.prev),
    onSettled: ()=> qc.invalidateQueries({queryKey:['posts']})
  });
}
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 18 with an emphasis on reliability and performance.*
- **2 Tech Q&A to rehearse**
  1) *Explain your approach to idempotent APIs and retries.*
  2) *How do you design pagination and prevent N+1 issues?*


### Interview Focus
- Pattern: **Event Sourcing** — why/when & trade-offs.
- MySQL: **Partition by Date** — demonstrate with EXPLAIN or schema.
- BE deep-dive: Indexing+EXPLAIN, Queue+Retry+DLQ.
- FE deep-dive: A11y Modal, RHF+Zod, RQ Mutation Optimistic.
- **Common pitfalls**:
- Transaction scope too wide increases deadlock risk.
- Eager loading without constraints can explode memory.
- Not normalizing error schema complicates FE.
- Client and server validation drift.


### Practice Tasks
- Implement: **Interview Prep Day 18** feature end-to-end (BE & FE), commit with README explaining design choices.
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