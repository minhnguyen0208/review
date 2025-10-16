# Day 08 — Interview Prep Day 8

_Generated on 2025-10-01 03:52._

### Plan Notes
- **EN (CSV)**: System Design 1

## Backend (BE)

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

#### Indexing+EXPLAIN
```sql
CREATE INDEX idx_posts_author_created ON posts(author_id, created_at DESC);
EXPLAIN SELECT id,title,created_at FROM posts WHERE author_id=? ORDER BY created_at DESC LIMIT 20;
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

#### MySQL — Partition by Date
Range partition on date to prune scans; ensure partition columns in PK/unique.

## Frontend (FE)

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

#### RQ List
```tsx
import { useQuery } from '@tanstack/react-query';
async function fetchPosts(){ const r=await fetch('/api/posts'); if(!r.ok) throw new Error(); return r.json(); }
export function Posts(){ const {data,isLoading,error}=useQuery({queryKey:['posts'],queryFn:fetchPosts}); /* render */ }
```

#### Suspense+Split
```tsx
import { lazy, Suspense } from 'react';
const Reports = lazy(()=> import('./Reports'));
export default function App(){ return <Suspense fallback={<div>Loading…</div>}><Reports/></Suspense>; }
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 8 with an emphasis on reliability and performance.*
- **2 Tech Q&A to rehearse**
  1) *Explain your approach to idempotent APIs and retries.*
  2) *How do you design pagination and prevent N+1 issues?*


### Interview Focus
- Pattern: **Strategy** — why/when & trade-offs.
- MySQL: **Partition by Date** — demonstrate with EXPLAIN or schema.
- BE deep-dive: Eloquent Perf, Indexing+EXPLAIN, Txn+Outbox.
- FE deep-dive: RQ Mutation Optimistic, RQ List, Suspense+Split.
- **Common pitfalls**:
- Pagination without deterministic order causes duplicates.
- Optimistic update without rollback corrupts UI state.
- SELECT * prevents covering index.
- Missing error boundaries hides failures.


### Practice Tasks
- Implement: **Interview Prep Day 8** feature end-to-end (BE & FE), commit with README explaining design choices.
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