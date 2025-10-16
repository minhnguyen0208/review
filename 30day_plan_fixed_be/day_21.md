# Day 21 — Interview Prep Day 21

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Migrations & Seeders
```php
Schema::table('posts', function(Blueprint $t){ $t->softDeletes(); });
Post::factory()->count(50)->create();
```

#### Testing (Pest)
```php
test('api returns resource shape', function() {
  $res = $this->getJson('/api/posts')->assertOk()->json();
  expect($res)->toHaveKeys(['data','links','meta']);
});
```

#### Query Builder vs Eloquent
```php
DB::table('posts')->where('published',1)->count(); // faster aggregate
```

#### Design Pattern — Repository
```php
interface PostRepository { public function paginate(array $filters,int $perPage=20): \Illuminate\Contracts\Pagination\LengthAwarePaginator; }
class EloquentPostRepository implements PostRepository {
  public function paginate(array $filters, int $perPage=20){
    return Post::query()
      ->when($filters['q']??null, fn($q,$v)=>$q->whereFullText('title',$v))
      ->when($filters['author_id']??null, fn($q,$v)=>$q->where('author_id',$v))
      ->orderByDesc('created_at')->paginate($perPage);
  }
}
class PostController extends Controller {
  public function __construct(private PostRepository $repo){}
  public function index(Request $r){ return PostResource::collection($this->repo->paginate($r->all())); }
}
```

#### MySQL — Covering Index
```sql
-- Bad: SELECT *
SELECT * FROM posts WHERE author_id = ? ORDER BY created_at DESC LIMIT 20;

-- Good: only needed cols + matching composite index
CREATE INDEX idx_posts_author_created ON posts(author_id, created_at DESC);
SELECT id, title, created_at FROM posts WHERE author_id=? ORDER BY created_at DESC LIMIT 20;
```
**Reasoning**: "Using index" in EXPLAIN => avoid back-to-table lookup → lower IO/latency.


## Frontend (FE)

#### React Query List
```tsx
import { useQuery } from '@tanstack/react-query';
async function fetchPosts(){ const r=await fetch('/api/posts'); if(!r.ok) throw new Error('Network'); return r.json(); }
export function Posts(){
  const { data, isLoading, error } = useQuery({ queryKey:['posts'], queryFn: fetchPosts, staleTime: 30_000 });
  if(isLoading) return <p>Loading…</p>;
  if(error) return <p>Failed to load.</p>;
  return <ul>{data.data.map((p:any)=> <li key={p.id}>{p.title}</li>)}</ul>;
}
```

### English (Interview)
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 21 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: backpressure, consistency, debounce, denormalization, isolation level, optimistic UI, rate limiting, throughput
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