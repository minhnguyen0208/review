# Day 22 — Interview Prep Day 22

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Testing (Pest)
```php
test('api returns resource shape', function() {
  $res = $this->getJson('/api/posts')->assertOk()->json();
  expect($res)->toHaveKeys(['data','links','meta']);
});
```

#### Feature Flags
```php
if(Feature::enabled('new-header')){ /* render new */ } else { /* old */ }
```

#### API Versioning
```php
Route::prefix('v1')->group(fn()=> Route::apiResource('posts', V1\PostController::class));
```

#### Design Pattern — Unit of Work
```php
class UserOnboarding {
  public function run(array $data){
    DB::transaction(function() use ($data){
      $user = User::create($data['user']);
      Profile::create(['user_id'=>$user->id] + $data['profile']);
      event(new UserOnboarded($user->id));
    });
  }
}
```

#### MySQL — EXPLAIN
```sql
EXPLAIN SELECT id,title,created_at FROM posts
WHERE author_id = 42
ORDER BY created_at DESC
LIMIT 20;
```
**Read fields**  
- `type`: prefer `ref`/`range`; avoid `ALL`  
- `rows`: est. rows to examine (lower is better)  
- `filtered`: % rows passing filter (higher is better)  
- `Extra`: "Using index" (covering), "Using filesort" (needs better index)


## Frontend (FE)

#### React Query Mutation
```tsx
import { useMutation, useQueryClient } from '@tanstack/react-query';
function useUpdatePost(){
  const qc = useQueryClient();
  return useMutation({
    mutationFn: (p:any)=> fetch('/api/posts/'+p.id,{method:'PUT',headers:{'Content-Type':'application/json'},body:JSON.stringify(p)}).then(r=>r.json()),
    onMutate: async (p)=>{ await qc.cancelQueries({queryKey:['posts']}); const prev=qc.getQueryData<any>(['posts']); qc.setQueryData(['posts'], (old:any)=> ({...old, data: old.data.map((x:any)=> x.id===p.id?{...x,...p}:x)})); return {prev}; },
    onError: (_e,_p,ctx)=> ctx?.prev && qc.setQueryData(['posts'], ctx.prev),
    onSettled: ()=> qc.invalidateQueries({queryKey:['posts']})
  });
}
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 22 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: debounce, idempotency, isolation level, memoization, observability, partitioning, retries, rollback
- **Q&A**: 1) *How do you ensure idempotency for create endpoints?*  2) *When do you choose client caching vs server caching?*
- **Behavioral (STAR)**: *Tell me about a production incident you owned end‑to‑end.*


### Practice Tasks
- Build feature end-to-end; capture EXPLAIN & tests.
- Record 60s answer on a topic from today.

### Acceptance Criteria
- BE schema/status correct + tests; FE handles loading/error/empty; DB EXPLAIN + index note.

### Docs & References
- [Database](https://dev.mysql.com/doc/)
- [Laravel](https://laravel.com/docs)
- [React Query](https://tanstack.com/query/latest)

### Reflection
- What trade-off today will you highlight in an interview?