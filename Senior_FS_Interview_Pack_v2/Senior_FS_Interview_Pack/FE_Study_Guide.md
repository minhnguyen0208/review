# Frontend Study Guide — React Hooks, RSC, Tailwind

## 1) React Hooks Essentials
### Topics
- `useState`, `useEffect` (deps chuẩn, cleanup, stale closures)
- `useMemo`, `useCallback`, `useRef`, `useReducer`
- AbortController: cancel fetch; debounce/throttle

### Code Snippet — Debounced Search with Cancel
```tsx
function SearchBox() {
  const [q, setQ] = useState('');
  const [data, setData] = useState<any[]>([]);
  const ctrlRef = useRef<AbortController | null>(null);

  useEffect(() => {
    if (!q) { setData([]); return; }
    const t = setTimeout(async () => {
      ctrlRef.current?.abort();
      const ctrl = new AbortController();
      ctrlRef.current = ctrl;
      const res = await fetch(`/api/search?q=${encodeURIComponent(q)}`, { signal: ctrl.signal });
      if (res.ok) setData(await res.json());
    }, 350);
    return () => { clearTimeout(t); ctrlRef.current?.abort(); };
  }, [q]);

  return (
    <div className="p-4">
      <input className="border p-2 w-full" value={q} onChange={e=>setQ(e.target.value)} placeholder="Search..." />
      <ul className="mt-3 space-y-2">{data.map(item => <li key={item.id} className="p-2 border rounded">{item.name}</li>)}</ul>
    </div>
  );
}
```

---

## 2) State Management & Server State
- Context vs Zustand/Redux Toolkit (selector-based to avoid re-render)
- TanStack Query: cache keys, `staleTime`, optimistic update with rollback

```tsx
// TanStack Query mutation example with optimistic update
const mutation = useMutation(updateItem, {
  onMutate: async (vars) => {
    await queryClient.cancelQueries(['items']);
    const prev = queryClient.getQueryData<Item[]>(['items']);
    queryClient.setQueryData<Item[]>(['items'], old => old!.map(i => i.id===vars.id ? {...i, ...vars} : i));
    return { prev };
  },
  onError: (_err, _vars, ctx) => { queryClient.setQueryData(['items'], ctx?.prev); },
  onSettled: () => { queryClient.invalidateQueries(['items']); },
});
```

---

## 3) Performance — Big Lists & Splitting
- Virtualization (react-window), memo rows, stable keys/handlers
- Code splitting: `React.lazy` + `Suspense`

```tsx
const Heavy = React.lazy(() => import('./Heavy'));
export default function Page() {
  return (
    <Suspense fallback={<div>Loading…</div>}>
      <Heavy />
    </Suspense>
  );
}
```

---

## 4) React Server Components (RSC) & Streaming (Next.js)
- Data fetch on server → nhỏ bundle, TTFB tốt với streaming
- Client islands chỉ cho phần tương tác

```tsx
// app/dashboard/page.tsx
export default async function Page() {
  const data = await getServerData(); // server side
  return (
    <div className="space-y-6">
      <Stats data={data} /> {/* Server Component */}
      <ClientChart data={data} /> {/* "use client" component */}
    </div>
  );
}
```

---

## 5) Tailwind Practical Patterns
- Responsive prefixes (`sm: md: lg:`), dark mode (`dark:`)
- Class composition (clsx/cva), tránh “class soup” bằng components
- A11y: focus ring, aria-* on interactive elements

```tsx
<button
  className="px-4 py-2 rounded-xl border shadow hover:shadow-md focus:outline-none focus:ring-2 focus:ring-offset-2"
  aria-label="Save changes"
>
  Save
</button>
```

---

## 6) Error Boundaries & Suspense
```tsx
function ErrorBoundary({ children }) { /* implement with componentDidCatch or libraries */ }
```

---

## 7) FE Observability
- Log client errors, web-vitals (LCP/CLS/FID), tracing IDs into API calls
- Guardrails: retry policy, backoff for fetch; user feedback (toasts/loaders)