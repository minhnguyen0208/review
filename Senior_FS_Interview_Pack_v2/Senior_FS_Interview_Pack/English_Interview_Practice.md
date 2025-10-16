# English Interview Practice — Senior PHP/React

## 1) Technical Answer Structure
- Open with context: “**The key constraints are…**”
- Outline approach: “**First… Then… Finally…**”
- Close with trade-offs & validation: “**Pros/Cons, metrics, next steps**”

### Template
```
First, I'd clarify the requirements (data size, latency targets).
Then, I'd choose keyset pagination with streaming to avoid memory spikes.
Finally, I'd validate with p95 latency and add checkpoints for retries.
```

---

## 2) Ready-to-use Answers (1–2 minutes)

### Export Millions of Rows
> “For multi-million row exports, I avoid OFFSET and use keyset pagination. I stream CSV directly to the client or generate XLSX asynchronously with workers and store it in object storage with a signed URL. The pipeline is idempotent via Redis/DB upsert; we track checkpoints so retries resume where they left off. This reduces memory usage and improves TTFB.”

### Idempotent Pipelines
> “We embrace at-least-once delivery with idempotency keys and database upserts. For consistency, we use the outbox pattern so DB writes and messages are bound within a transaction. Retries use exponential backoff; failures go to a DLQ with replay tooling.”

### React Performance
> “We keep data fetching in server components to reduce bundle size and use client islands for interactions. For big lists, we use virtualization and selector-based state to avoid re-renders. Mutations use optimistic updates with rollback.”

---

## 3) Behavior — STAR Stories

### Optimization Story
- **S**: Reports timing out for 40M+ rows.
- **T**: Deliver export without OOM/timeouts.
- **A**: Keyset + streaming, cached lookups, SQS workers for XLSX.
- **R**: TTFB < 2s; 0 OOM; stable under load.

### Incident Story
- **S**: Duplicate pushes due to low visibility timeout.
- **T**: Remove duplicates.
- **A**: Idempotency keys; tune visibility; DLQ + replay.
- **R**: Duplicates ~0; improved MTTR.

---

## 4) Quick Phrases & Connectors
- “**In my experience**, the trade-off is…"
- “**The key point is** minimizing tail latency (p95)…"
- “**For example**, we replaced OFFSET with keyset pagination…”

---

## 5) Mock Q&A (Practice)
- *How do you export 50M rows safely?*
- *How do you make SQS jobs idempotent?*
- *Explain useEffect vs useMemo, and when to use each.*
- *Design a webhook system with retries and replay.*