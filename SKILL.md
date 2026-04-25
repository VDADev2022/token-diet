---
name: token-diet
description: "Production-ready token optimization: reduce costs 40–75% through retrieval pruning, smart caching, and model routing. Use whenever optimizing API costs, latency, or managing long context—especially for RAG pipelines, high-volume systems, multi-turn conversations, or when context exceeds 2K tokens."
---

# Token Diet v3.0 (Production-Ready)

Deployment-grade token optimization with execution order, ROI metrics, guardrails, and measurable outcomes.

## Execution Flow (Order Matters)

```
Q → Retrieve → Prune → Cache → Route → Build Prompt → Compress → Call LLM → Measure → Update State
```

**Why this order**: Early pruning eliminates waste before caching/routing decisions. Compression happens last (post safety checks). Measurement feeds back into next iteration.

---

## 1. Retrieval Pruning (Highest ROI)

**Goal**: Never send unnecessary tokens.

**Pipeline**:
```
query → embed → topK(20) → dedupe → sentence-trim → topK(5–8)
```

**Concrete rules**:
* Drop near-duplicate chunks (>85% similarity)
* Keep only sentences containing query terms or answer-bearing context
* Hard cap: ≤1200 tokens total context
* If retrieval yields <5 relevant chunks, fetch 5 more before pruning

**Win**: 30–60% reduction immediately (no quality loss if done right).

---

## 2. Smart Caching (Implement Early)

**Split prompt into 3 blocks**:
```
[STATIC_SYS]   → cache (reuse across requests)
[DYNAMIC_CTX]  → no cache (changes per request)
[USER_INPUT]   → no cache (one-off query)
```

**Cache candidates**:
* System instructions
* Schemas (JSON, SQL, output format)
* Tool definitions
* Few-shot examples (if static)

**Decision rule**:
```
if block_reused_in_next_N_requests → cache
else → inline (no caching overhead)
```

**Implementation**: Use API cache headers (`cache_control: {"type": "ephemeral"}` for Anthropic; similar for other providers).

---

## 3. Conditional Routing (Cheap-First Model Selection)

**Decision tree**:
```
if task ∈ {format, extract, classify, summarize}
    → small model (Haiku, GPT-4o-mini, Gemini-Flash)
else if task ∈ {reason, synthesize, creative}
    → large model (Sonnet, GPT-4o, Gemini-Pro)
```

**Safety guard**: If confidence low or error detected → retry on stronger model (don't downgrade past first attempt).

**Result**: 60–80% cost reduction on simple tasks without quality hit.

---

## 4. Tool Batching (Avoid Round-Trips)

**Bad** (5 calls + reread context 5x):
```
get_user(1) → get_user(2) → get_user(3)
```

**Good** (1 call):
```
get_users([1, 2, 3])
```

**Also batch**:
* Embedding calls (vector DBs)
* Database reads
* Validation checks

---

## 5. Stateful Memory (Replace Naive Summarization)

**Structure state as JSON**:
```json
{
  "facts": ["user_id=42", "role=admin"],
  "constraints": ["budget_limit=$1000", "deadline=2026-04-30"],
  "open_tasks": ["approval_pending", "docs_missing"],
  "summary": "(optional) Human-readable 150-token version"
}
```

**When to regenerate**:
* If contradiction detected between new facts and old state
* After ≥5 turns
* If summary token cost > fresh state token cost

**Why**: Structured state lets LLM reason precisely. Optional summary is for readability/debugging, not required.

---

## 6. Linguistic Compression (Last-Mile Only)

**Apply only right before the API call**:

```
"Please analyze the dataset and provide a concise summary"
→ "Analyze data. Summarize concise."
```

**DO NOT compress**:
* Code (breaks syntax)
* JSON schema (breaks parsing)
* Legal/safety text (liability risk)
* URLs, identifiers

**Benefit**: ~15% token reduction on prose prompts; safe only if applied surgically.

---

## 7. Prompt Assembly Template

**Minimal structure** (reorder as needed):
```
[SYS — cached]

Context:
{pruned_context_1200_tokens_max}

State:
{facts + constraints (JSON)}

Task:
{compressed_user_input}
```

**Why minimal**: Every line you don't send saves tokens. No filler, no explanations that don't move the needle.

---

## 8. Measurement (Non-Optional)

**Track per request**:
```
tokens_in
tokens_out
latency_ms
cache_hit_rate (%)
retrieval_tokens_before / after
cost_usd
```

**Add alerts**:
* Context >1500 tokens → investigate retrieval
* Cache hit rate <70% → check static block design
* Cost spike >20% vs. baseline → disable routing/compression, debug

**Why**: Without measurement, you're flying blind. Guardrails only work if you see violations early.

---

## 9. Safety Guards (Prevent Silent Failure)

| Problem | Signal | Action |
|---------|--------|--------|
| Answer quality drops | User feedback or eval score ↓ | Disable compression first, then increase context |
| Hallucination risk rises | Repeated factual errors | Increase context, not summary; rebuild state from raw history |
| Silent drift | State contradicts history | Rebuild state from scratch; check if constraints stale |
| Over-pruning | Query unmatched to results | Fetch more chunks before pruning; lower similarity threshold |

**Golden rule**: If unsure whether compression/routing broke something, **disable it and rerun**. Measure the difference. Only re-enable if safe.

---

## Ultra-Compressed (Caveman Runtime Spec)

```
Retrieve → prune hard
Cache static
Route cheap→expensive
Batch all
State > summary
Compress last
Measure always
Guard drift
```

---

## Net Effect (Realistic)

| Metric | Target | Conditions |
|--------|--------|------------|
| Token reduction | 50–80% | All 9 steps applied in order |
| Latency | ↓20–50% | Caching + routing active |
| Cost | ↓40–75% | Routing to smaller models + cache hits >70% |
| Quality | Stable | Guards enforced; no compression on code/schema/legal |

**Typical breakdown**:
* Pruning: 30–40% reduction
* Caching: 20–30% reduction (if reuse >50%)
* Routing: 30–50% cost reduction (simple tasks to cheap model)
* Compression: 10–15% reduction (prose only)
* **Combined**: Multiplicative, not additive.

---

## Common Mistakes

| Mistake | Result | Fix |
|---------|--------|-----|
| Skip measurement | Invisible failures | Instrument from day 1 |
| Compress code/JSON | Syntax errors, parsing fails | Never compress structured data |
| Prune too aggressively | Relevant context lost → hallucinations | Keep ≥1200 tokens context; test quality on pruned versions |
| Cache everything | No room for dynamic context | Cache only truly static blocks |
| Summarize too early | Miss important detail → wrong answer | Use structured state; regenerate if contradiction |
| Ignore routing guardrails | Quality collapse on hard tasks | Always allow retry on stronger model |

---

## Author

*   **Name**: Vijay Dixon
*   **Email**: vijaydixon2026+aitools@gmail.com
