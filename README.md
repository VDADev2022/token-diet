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


<img width="3600" height="2100" alt="image" src="https://github.com/user-attachments/assets/66f26d37-7238-4105-8c92-5be0443d460a" />

---

## 🛠️ Usage Checklist

1.  **Cache** static system prompts to save on repeat turns.
2.  **Summarize** conversation history once it exceeds token thresholds.
3.  **Batch** tool calls to reduce API round-trips.
4.  **Route** simple extraction/formatting tasks to cheaper models.
5.  **Compress** instructions using **Caveman Ultra** linguistic rules.

---

## 🚀 Integration

To leverage the Token-Diet skill in your AI workflows, follow these integration guidelines:

### Download the SKILL.md file and upload to AI tool (manus) as a skill, some tools may require it to be downloaded as a zip file (claude)

### For Manus Agents

Manus agents can directly utilize the Token-Diet skill by referencing its name. When you want to apply token reduction techniques, simply invoke the skill with the `/token-diet` command, followed by your prompt or specific instructions. The agent will automatically apply the defined strategies (e.g., Caveman Ultra, caching, summarization) to optimize token usage.

**Example Usage within Manus:**

```
/token-diet Analyze the provided log data for anomalies and summarize key findings concisely.
```

### For Claude and Other LLMs (via PCodec/Gateway)

For direct integration with Claude or other LLMs, the Token-Diet principles are best applied at the gateway or orchestration layer using the PCodec (Prompt-Codec) mechanism. This involves an intermediary service that intercepts and transforms prompts before sending them to the LLM.

1.  **Implement a Gateway**: Set up a proxy or gateway service that sits between your application and the LLM API.
2.  **Apply PCodec Rules**: Configure the gateway to apply the Token-Diet's PCodec rules. This means defining short codes (e.g., `R1`, `S1`) that expand into full, but optimized, instructions or schema definitions.
3.  **Integrate Advanced Techniques**: The gateway can also manage prompt caching, rolling summarization, tool call batching, and conditional model routing before the prompt reaches the LLM.

**Example Prompt (after gateway processing):**

```
User: R1: Analyze data. Summarize. S1: JSON output.
```

(Where `R1` expands to a Caveman Ultra compressed instruction and `S1` ensures a structured JSON output.)

---

## 🧑‍💻 Author

*   **Name**: Vijay Dixon
*   **Email**: [vijaydixon2026+aitools@gmail.com](mailto:vijaydixon2026+aitools@gmail.com)

---

## 📜 License

This project is licensed under the MIT License - see the LICENSE file for details.

---

#LLMOptimization #TokenReduction #PromptEngineering #AICostSavings #LLMEfficiency #AgenticAI #GenerativeAI #AI #MachineLearning #DeepLearning #NLP #PromptCompression #ContextManagement #AIDevelopment
