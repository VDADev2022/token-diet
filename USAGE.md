# 📚 Token-Diet v3.0: Detailed Usage Guide

This guide provides comprehensive instructions on how to implement and leverage the **Token-Diet v3.0** framework to optimize your Large Language Model (LLM) workflows. By following these steps, you can significantly reduce token usage, lower API costs, improve latency, and enhance the reliability of your AI applications.

---

## 1. Understanding the Execution Flow (Order Matters)

Token-Diet v3.0 operates on a strict, optimized pipeline. Adhering to this order is crucial for maximizing efficiency and preventing unintended side effects.

`Query → Retrieve → Prune → Cache → Route → Build Prompt → Compress → Call LLM → Measure → Update State`

**Key Principle**: Eliminate unnecessary tokens as early as possible. Compression is a final, surgical step applied only after safety checks.

---

## 2. Step-by-Step Implementation

### 2.1. Query (Q)

This is the initial user request or system trigger that kicks off the LLM interaction. It can be a natural language query, a structured command, or an event.

### 2.2. Retrieve

Fetch relevant information from your knowledge base (e.g., vector database, internal documents, web search) based on the `Query`.

*   **Action**: Perform your standard RAG (Retrieval Augmented Generation) retrieval process.
*   **Output**: A collection of raw text chunks or documents.

### 2.3. Prune (Retrieval Pruning - Highest ROI)

Before sending retrieved context to the LLM, aggressively filter out irrelevant or redundant information. This is the most impactful step for token reduction.

*   **Pipeline**: `query → embed → topK(20) → dedupe → sentence-trim → topK(5–8)`
    *   **Dedupe**: Remove near-duplicate chunks (e.g., >85% similarity using embedding comparison).
    *   **Sentence-Trim**: Keep only sentences within chunks that contain query terms or are highly relevant to the answer.
    *   **Hard Cap**: Ensure the total pruned context is ≤1200 tokens.
    *   **Fallback**: If pruning yields <5 relevant chunks, fetch 5 more before re-pruning.
*   **Benefit**: 30–60% token reduction without quality loss.

### 2.4. Cache (Smart Caching - Implement Early)

Identify and cache static or frequently reused prompt components. This avoids sending the same tokens repeatedly.

*   **Split Prompt**: Conceptually divide your prompt into:
    *   `[STATIC_SYS]`: System instructions, personas, global rules.
    *   `[DYNAMIC_CTX]`: Pruned context, current state.
    *   `[USER_INPUT]`: The immediate user query.
*   **Cache Candidates**: System instructions, JSON schemas, tool definitions, static few-shot examples.
*   **Decision Rule**: Cache a block if it's likely to be reused in the next `N` requests (e.g., `N=5`). Otherwise, inline it.
*   **Implementation**: Utilize API-level caching features (e.g., Anthropic's `cache_control: {"type": "ephemeral"}`).

### 2.5. Route (Conditional Routing - Cheap-First Model Selection)

Dynamically select the most cost-effective LLM based on the complexity and type of the current task.

*   **Decision Tree**: 
    *   If task is `format`, `extract`, `classify`, `summarize` → use a small, fast model (e.g., Haiku, GPT-4o-mini, Gemini-Flash).
    *   If task is `reason`, `synthesize`, `creative` → use a large, powerful model (e.g., Sonnet, GPT-4o, Gemini-Pro).
*   **Safety Guard**: If the chosen small model returns low confidence or an error, automatically retry the request with a stronger model. Never silently downgrade quality.
*   **Benefit**: 60–80% cost reduction on simple tasks.

### 2.6. Build Prompt (Prompt Assembly Template)

Construct the final prompt using a minimal, structured template. Avoid filler or unnecessary explanations.

*   **Minimal Structure Example**:
    ```
    [SYS — cached]
    
    Context:
    {pruned_context_1200_tokens_max}
    
    State:
    {facts + constraints (JSON)}
    
    Task:
    {compressed_user_input}
    ```
*   **Principle**: Every line you don't send saves tokens. Only include components that directly contribute to the LLM's understanding and task execution.

### 2.7. Compress (Linguistic Compression - Last-Mile Only)

Apply linguistic compression (like Caveman Ultra) only to the `Task` portion of the prompt, and only right before the API call.

*   **Rules**: Abbreviate, strip conjunctions, use arrows for causality (X → Y), use fragments. (Refer to `SKILL.md` for full Caveman Ultra rules).
*   **Strict Exclusions**: **NEVER** compress:
    *   Code (breaks syntax)
    *   JSON schema (breaks parsing)
    *   Legal/safety text (liability risk)
    *   URLs, identifiers
*   **Benefit**: ~15% token reduction on prose prompts, applied surgically to prevent errors.

### 2.8. Call LLM

Send the optimized prompt to the selected LLM via its API.

### 2.9. Measure (Non-Optional)

Implement robust tracking and alerting for every LLM interaction. Without measurement, optimizations are blind.

*   **Track per request**: `tokens_in`, `tokens_out`, `latency_ms`, `cache_hit_rate (%)`, `retrieval_tokens_before / after`, `cost_usd`.
*   **Alerts**: Set up alerts for:
    *   Context >1500 tokens (investigate retrieval).
    *   Cache hit rate <70% (check static block design).
    *   Cost spike >20% vs. baseline (debug compression/routing).

### 2.10. Update State (Stateful Memory)

After the LLM response, update the `State` based on new information or task progression.

*   **When to regenerate**: If contradiction detected, after ≥5 turns, or if summary token cost > fresh state token cost.
*   **Why**: Maintain an accurate, structured representation of the conversation's context for future turns.

---

## 3. Integration with Manus Agents

Manus agents can directly utilize the Token-Diet skill. When you invoke the skill, the agent will apply the defined strategies to optimize your prompt.

**Example Usage within Manus:**

```
/token-diet Analyze the provided log data for anomalies and summarize key findings concisely.
```

Behind the scenes, Manus will apply the v3.0 pipeline: pruning context, checking cache, routing to an appropriate model, building the prompt, and compressing the task instruction before calling the LLM.

---

## 4. Integration with Claude and Other LLMs (via PCodec/Gateway)

For direct integration with LLMs like Claude, implement a gateway or orchestration layer that sits between your application and the LLM API. This gateway will execute the Token-Diet v3.0 pipeline.

1.  **Implement a Gateway**: Develop a proxy service to intercept and transform prompts.
2.  **Apply PCodec Rules**: Define short codes (e.g., `R1` for a rule, `S1` for a schema) that the gateway expands into full, optimized instructions or JSON schemas before sending to the LLM.
3.  **Orchestrate Techniques**: The gateway manages caching, pruning, routing, and stateful memory before the prompt reaches the LLM.

**Example Prompt (sent from your application to the gateway):**

```
User: R1: Analyze data. Summarize. S1: JSON output.
```

(The gateway expands `R1` into a Caveman Ultra compressed instruction and `S1` into a full JSON schema, then applies other Token-Diet steps before forwarding to the LLM.)

---

## 5. Common Mistakes to Avoid

*   **Skipping Measurement**: You can't optimize what you don't measure. Instrument everything from day one.
*   **Compressing Structured Data**: Never apply linguistic compression to code, JSON, or legal text; it leads to syntax errors or semantic loss.
*   **Over-Pruning**: Aggressive pruning without quality checks can lead to hallucinations. Always keep sufficient context (e.g., ≥1200 tokens) and test quality.
*   **Caching Everything**: Cache only truly static blocks; dynamic context needs to be fresh.
*   **Ignoring Guardrails**: Always allow for retries on stronger models if a cheaper model fails, and rebuild state if contradictions arise.

By diligently applying Token-Diet v3.0, you can achieve a highly efficient, cost-effective, and robust LLM system.
