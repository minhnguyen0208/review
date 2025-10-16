# Day 16 — Interview Prep Day 16

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

#### Txn+Outbox
```php
DB::transaction(function(){
  $order = Order::create([...]);
  Outbox::create(['event'=>'OrderCreated','payload'=>json_encode($order)]);
});
// worker ships Outbox to broker
```

#### Design Pattern — Unit of Work
Bundle writes in a single transaction; ensure consistency.

#### MySQL — Covering Index
Select only needed columns; align WHERE + ORDER BY with composite index.

## Frontend (FE)

#### RQ List
```tsx
import { useQuery } from '@tanstack/react-query';
async function fetchPosts(){ const r=await fetch('/api/posts'); if(!r.ok) throw new Error(); return r.json(); }
export function Posts(){ const {data,isLoading,error}=useQuery({queryKey:['posts'],queryFn:fetchPosts}); /* render */ }
```

#### InfiniteQuery
```tsx
import { useInfiniteQuery } from '@tanstack/react-query';
const fetchPage = async ({pageParam=1})=> (await fetch(`/api/posts?page=${pageParam}`)).json();
const { data, fetchNextPage, hasNextPage } = useInfiniteQuery({ queryKey:['posts'], queryFn:fetchPage, getNextPageParam:(last)=> last.next_page ?? false });
```

#### Suspense+Split
```tsx
import { lazy, Suspense } from 'react';
const Reports = lazy(()=> import('./Reports'));
export default function App(){ return <Suspense fallback={<div>Loading…</div>}><Reports/></Suspense>; }
```

### English (Interview)
- **Self-intro (30–45s)**: *Hi, I'm Minh… Today I focused on interview prep day 16 with an emphasis on reliability and performance.*
- **2 Tech Q&A to rehearse**
  1) *Explain your approach to idempotent APIs and retries.*
  2) *How do you design pagination and prevent N+1 issues?*


### Interview Focus
- Pattern: **Unit of Work** — why/when & trade-offs.
- MySQL: **Covering Index** — demonstrate with EXPLAIN or schema.
- BE deep-dive: Cache+Lock, Eloquent Perf, Txn+Outbox.
- FE deep-dive: RQ List, InfiniteQuery, Suspense+Split.
- **Common pitfalls**:
- Eager loading without constraints can explode memory.
- Not normalizing error schema complicates FE.
- Transaction scope too wide increases deadlock risk.
- Optimistic update without rollback corrupts UI state.


### Practice Tasks
- Implement: **Interview Prep Day 16** feature end-to-end (BE & FE), commit with README explaining design choices.
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