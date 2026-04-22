# 🥗 Token-Diet: Advanced LLM Prompt Optimization Framework

**Token-Diet** is a comprehensive framework designed to minimize token consumption in Large Language Model (LLM) prompts without sacrificing output quality. By integrating linguistic, algorithmic, and architectural patterns, Token-Diet enables up to **20x reduction** in token usage, significantly lowering API costs and improving inference latency.

---

## 🚀 Key Features

### 1. ⚡ Prompt Caching (Static & Ephemeral)
Leverage provider-level caching (e.g., Anthropic's `cache_control`) to mark static system prompts or large context blocks.
*   **Impact**: Reduces repeat turn costs by up to **90%**.
*   **Best for**: Multi-turn agentic workflows and stable system instructions.

### 2. 🔄 Rolling Conversational Summarization
Prevents linear context bloat by maintaining a fixed window of recent history while summarizing older turns.
*   **Impact**: Maintains performance in long-running sessions without hitting context limits.
*   **Mechanism**: Automatically condenses turns 1 to (N-6) into a concise summary block.

### 3. 📦 Tool Call Batching
Optimizes agentic loops by grouping multiple tool requests into a single round-trip.
*   **Impact**: Minimizes redundant context re-reads and reduces total input tokens.
*   **Example**: `get_users(ids: list[str])` instead of multiple `get_user(id)` calls.

### 4. 🚦 Conditional Model Routing
Intelligently dispatches tasks to the most cost-effective model based on complexity.
*   **Simple Tasks**: Routed to lightweight models (e.g., Haiku, GPT-4o-mini).
*   **Complex Tasks**: Routed to high-reasoning models (e.g., Opus, GPT-4o).

### 5. 🦴 Linguistic Compression (Caveman Ultra)
Applies extreme linguistic pruning to instructions and queries.
*   **Impact**: Cuts token usage by **~75%** while retaining 100% technical accuracy.
*   **Rules**: Strips articles, fillers, and conjunctions; uses abbreviations and causal arrows (X → Y).

### 6. 🛠️ PCodec (Prompt-Codec) & Schema-First
Encodes complex rules and JSON schemas into short, referenceable codes (R1, R2, S1).
*   **Mechanism**: Gateway-level expansion ensures the model receives full context only when necessary.

---

## 📊 Effectiveness Comparison

| Feature | Token-Diet v2.0 | Standard RAG | LLMLingua-2 |
| :--- | :--- | :--- | :--- |
| **Compression** | **Ultra (20x+)** | Low (1.2x) | High (10x) |
| **Latency** | **Lowest** | High | Mid |
| **Cost** | **Lowest** | High | Mid |
| **Scope** | Full Stack | Context Only | Token Pruning |

---

## 🛠️ Usage Checklist

1.  **Cache** static system prompts to save on repeat turns.
2.  **Summarize** conversation history once it exceeds token thresholds.
3.  **Batch** tool calls to reduce API round-trips.
4.  **Route** simple extraction/formatting tasks to cheaper models.
5.  **Compress** instructions using **Caveman Ultra** linguistic rules.

---

<img width="3600" height="2100" alt="image" src="https://github.com/user-attachments/assets/66f26d37-7238-4105-8c92-5be0443d460a" />


## 🧑‍💻 Author

*   **Name**: Vijay Dixon
*   **Email**: [vijaydixon2026+aitools@gmail.com](mailto:vijaydixon2026+aitools@gmail.com)

---

## 📜 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

#LLMOptimization #TokenReduction #PromptEngineering #AICostSavings #LLMEfficiency #AgenticAI #GenerativeAI #AI #MachineLearning #DeepLearning #NLP #PromptCompression #ContextManagement #AIDevelopment
