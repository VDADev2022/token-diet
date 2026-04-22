---
name: token-diet
description: "Reduces token usage in LLM prompts through advanced compression, caching, and architectural patterns. Use for: optimizing prompt length, reducing API costs, improving latency, and managing long context."
---

# Token Diet Skill (v2.0 - 2026 Edition)

This skill provides advanced strategies for minimizing token consumption in LLM prompts. It combines linguistic, algorithmic, and architectural patterns for maximum efficiency.

## Core Principles

*   **Information Density**: Prioritize conveying essential information concisely.
*   **Context Management**: Only include relevant context, and manage it dynamically.
*   **Architectural Efficiency**: Use caching, routing, and batching to reduce redundant processing.

## Advanced Techniques

### 1. Prompt Caching (Static & Ephemeral)
Mark static system prompts or large context blocks as cacheable.
*   **Mechanism**: Use `cache_control: {"type": "ephemeral"}` (Anthropic) or similar provider features.
*   **Benefit**: ~90% cost reduction on repeat turns in a conversation.

### 2. Rolling Conversational Summarization
Instead of carrying full history, keep only the last N turns verbatim and summarize older turns.
*   **Workflow**: If history > 6000 tokens, summarize turns 1 to (N-6) into a 200-token block.
*   **Benefit**: Prevents linear cost growth in long chats.

### 3. Tool Call Batching
Design tools to accept multiple inputs in a single call.
*   **Instead of**: `get_user(id)` called 5 times.
*   **Use**: `get_users(ids: list[str])` called once.
*   **Benefit**: Reduces round-trips and redundant context re-reads.

### 4. Conditional Model Routing
Route tasks based on complexity to the most cost-effective model.
*   **SIMPLE (Extraction, Formatting)**: Route to Haiku / GPT-4o-mini / Gemini-Flash.
*   **COMPLEX (Reasoning, Synthesis)**: Route to Opus / GPT-4o / Gemini-Pro.
*   **Benefit**: Up to 80% cost reduction by avoiding "over-modeling."

### 5. Linguistic Compression (Caveman Ultra Mode)
Apply ultra-terse linguistic rules to instructions and queries for maximum token reduction.
*   **Rules**: Abbreviate (DB/auth/config/req/res/fn/impl), strip conjunctions, use arrows for causality (X → Y), one word when one word enough. Drop articles, filler, pleasantries. Use fragments.
*   **Example**: "Please analyze the following data and provide a summary" → "Data analyze. Summarize."
*   **Benefit**: Cuts token usage ~75% while retaining technical accuracy.

### 6. PCodec (Prompt-Codec) & Schema-First
Encode complex rules and schemas into short codes (R1, R2, S1). Can also integrate linguistic compression levels.
*   **Mechanism**: Gateway expands codes before the API call. Can apply Caveman Ultra compression to expanded rules/instructions.
*   **Example**: "Output JSON matching schema S1." or "R1: Factual; if unsure, state & request source." (then Caveman Ultra applied by PCodec to become "R1: Factual; unsure → state, req source.")

## Comparison: Caveman vs. Token-Diet

| Feature | Caveman Skill | Token-Diet Skill |
| :--- | :--- | :--- |
| **Focus** | Linguistic Style | Systemic Optimization |
| **Mechanism** | Text transformation | Caching, Routing, Batching |
| **Compression** | ~75% (Linguistic) | Up to 95% (Systemic) |
| **Synergy** | Use Caveman *inside* Token-Diet for maximum (20x) reduction. | |

## Usage Checklist
1.  **Cache** static prompts.
2.  **Summarize** old history.
3.  **Batch** tool calls.
4.  **Route** to cheaper models for simple tasks.
5.  **Compress** linguistically (Caveman style).

---
