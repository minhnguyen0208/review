# Day 08 — Interview Prep Day 8

_Generated on 2025-10-01 04:22._

### Plan Notes
_(no CSV notes)_

## Backend (BE)

#### Middleware
```php
// app/Http/Middleware/ForceJson.php
public function handle($request, Closure $next){
  $request->headers->set('Accept','application/json');
  return $next($request);
}
// Kernel registration & when to use global vs route middleware
```

#### Service Container & DI
```php
$this->app->bind(PricingStrategy::class, PremiumPricing::class);
$svc = app(PricingStrategy::class); // resolved via container
```

#### Design Pattern — Strategy
```php
interface PricingStrategy { public function price(float $base, array $ctx=[]): float; }
class DefaultPricing implements PricingStrategy { public function price(float $b, array $ctx=[]): float { return $b; } }
class PremiumPricing implements PricingStrategy { public function price(float $b, array $ctx=[]): float { return round($b*0.95,2); } }
class CheckoutService {
  public function __construct(private PricingStrategy $strategy){}
  public function total(float $base, array $ctx=[]): float { return $this->strategy->price($base,$ctx); }
}
// Binding
$this->app->bind(PricingStrategy::class, fn()=> new PremiumPricing());
$total = (new CheckoutService(app(PricingStrategy::class)))->total(100.0);
```

#### MySQL — Partition by Date
```sql
ALTER TABLE logs
PARTITION BY RANGE (TO_DAYS(created_at))(
  PARTITION p2025 VALUES LESS THAN (TO_DAYS('2026-01-01')),
  PARTITION pmax  VALUES LESS THAN MAXVALUE
);
```
**Gotcha**: partitioning column must appear in every UNIQUE/PRIMARY key (MySQL 8.0).


## Frontend (FE)

#### useState
```tsx
const [count,setCount] = useState(0);
```

#### useEffect
```tsx
useEffect(()=>{ document.title = `Count ${count}`; return () => {/* cleanup */}; }, [count]);
```

#### useMemo
```tsx
const expensive = useMemo(()=> heavy(data), [data]); // memo only if heavy & stable
```

#### useCallback
```tsx
const onClick = useCallback(()=> doThing(id), [id]); // stable ref for deps
```

#### useRef
```tsx
const inputRef = useRef<HTMLInputElement>(null); useEffect(()=>{ inputRef.current?.focus(); },[]);
```

#### useReducer
```tsx
function reducer(s, a){ switch(a.type){ case 'inc': return {...s, n:s.n+1}; default:return s; } }
const [state, dispatch] = useReducer(reducer, { n:0 });
```

#### Custom Hook
```tsx
function useDebounce<T>(value:T, delay=300){ const [v,setV] = useState(value); useEffect(()=>{ const id=setTimeout(()=>setV(value),delay); return ()=>clearTimeout(id);},[value,delay]); return v; }
```

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
- **Elevator pitch**: *Hi, I'm Minh, a backend-leaning full‑stack engineer. Today I focused on interview prep day 8 and captured the trade‑offs I'd present in interviews.*
- **Vocabulary**: backpressure, consistency, debounce, denormalization, isolation level, memoization, rate limiting, retries
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
- [React](https://react.dev/learn)
- [React Query](https://tanstack.com/query/latest)

### Reflection
- What trade-off today will you highlight in an interview?