# Day 07 — Interview Prep Day 7

_Generated on 2025-10-01 03:52._

### Plan Notes
- **EN (CSV)**: Security & Auth

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

#### MySQL — EXPLAIN
Check type, rows, filtered, Extra; avoid ALL & filesort via proper index.

## Frontend (FE)

#### Suspense+Split
```tsx
import { lazy, Suspense } from 'react';
const Reports = lazy(()=> import('./Reports'));
export default function App(){ return <Suspense fallback={<div>Loading…</div>}><Reports/></Suspense>; }
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
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 7 with an emphasis on reliability and performance.*
- **2 Tech Q&A to rehearse**
  1) *Explain your approach to idempotent APIs and retries.*
  2) *How do you design pagination and prevent N+1 issues?*


### Interview Focus
- Pattern: **Factory** — why/when & trade-offs.
- MySQL: **EXPLAIN** — demonstrate with EXPLAIN or schema.
- BE deep-dive: Cache+Lock, Eloquent Perf.
- FE deep-dive: Suspense+Split, RQ Mutation Optimistic.
- **Common pitfalls**:
- Retry without jitter can thundering-herd.
- Client and server validation drift.
- Transaction scope too wide increases deadlock risk.
- Pagination without deterministic order causes duplicates.


### Practice Tasks
- Implement: **Interview Prep Day 7** feature end-to-end (BE & FE), commit with README explaining design choices.
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