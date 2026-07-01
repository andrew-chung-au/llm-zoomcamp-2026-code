# Module 3: AI Orchestration with Kestra — Homework Submission

This repository contains the configuration files, flows, and deployment scripts used to complete the Module 3 homework on AI Orchestration with Kestra.

---

## Repository Structure

The directory is structured as follows to ensure repeatable execution and clear tracking:

*   **`03-orchestration/flows/`**: Contains the YAML orchestration pipelines imported into the Kestra UI to run the example flows.
    *   `1_chat_without_rag.yaml`: Used to observe baseline LLM responses lacking localized context[cite: 1].
    *   `2_chat_with_rag.yaml`: Implements document ingestion and a localized RAG pipeline via Kestra's KV store.
    *   `3_rag_with_websearch.yaml`: Leverages real-time web search capabilities for context augmentation[cite: 1]. *Note: The provider block in this file was updated to use Gemini instead of OpenAI to align with the local stack.*
    *   `4_simple_agent.yaml`: Used extensively to test token usage parameters between short and long summaries.
    *   `5_web_research_agent.yaml`: Runs an autonomous tool-calling loop utilizing web research and filesystem tasks.
    *   `6_multi_agent_research.yaml`: Orchestrates a multi-agent system using specialized sub-agents working in tandem.
*   **`docker-compose.yml`**: The container configuration script used to orchestrate and run the local Kestra instance[cite: 1].
*   **`03-orchestration-notes.md`**: Contains personal documentation.

---

## Homework Answers:

### Question 1: Context Engineering

Try the following experiment:

1. Open ChatGPT in a private browser window: [https://chatgpt.com](https://chatgpt.com/)
2. Enter this prompt: "Create a Kestra flow that loads NYC taxi data from CSV to BigQuery"
3. Then, use Kestra's AI Copilot with the same prompt

After trying the same prompt in ChatGPT vs Kestra's AI Copilot, what is the primary reason AI Copilot generates better Kestra flows?

- AI Copilot uses a more powerful model
- AI Copilot has access to current Kestra plugin documentation
- AI Copilot uses more tokens
- AI Copilot has internet access

**Answer:** AI Copilot has access to current Kestra plugin documentation

---

### Question 2: RAG vs No RAG

Run both `1_chat_without_rag.yaml` and `2_chat_with_rag.yaml` in the Kestra UI. Read the execution logs for each.

The non-RAG response about Kestra 1.1 features is best described as:

- Accurate and specific, matching the actual release notes
- Vague, generic, or fabricated — the model guesses from training data
- Empty — the model refuses to answer without context
- Identical to the RAG version

**Answer:** Vague, generic, or fabricated — the model guesses from training data

---

### Question 3: Token Usage — Short Summary

Run `4_simple_agent.yaml` with `summary_length = short` (leave the other inputs as defaults).

Open the execution logs and find the token usage logged by the `log_token_usage` task.

What is the approximate output token count for `multilingual_agent`?

- 5-15 tokens
- 60-100 tokens
- 200-400 tokens
- 500+ tokens

```
2026-07-01 21:25:08.069
📊 Token Usage Summary:

Multilingual Agent:
- Input tokens: 282
- Output tokens: 61
- Total tokens: 343

English Brevity Agent:
- Input tokens: 76
- Output tokens: 31
- Total tokens: 107

💡 Tip: Monitor token usage to understand costs and optimize prompts!
```

**Answer:** 60-100 tokens

---

### Question 4: Token Usage — Long Summary

Run `4_simple_agent.yaml` again with `summary_length = long`.

Compare the `multilingual_agent` output token count to your result from Question 3. Roughly how many times more output tokens does the long summary use?

- About the same (within 20%)
- 2-5x more
- 10-20x more
- 50x more

```
2026-07-01 21:33:19.480
📊 Token Usage Summary:

Multilingual Agent:
- Input tokens: 282
- Output tokens: 176
- Total tokens: 458

English Brevity Agent:
- Input tokens: 191
- Output tokens: 39
- Total tokens: 230

💡 Tip: Monitor token usage to understand costs and optimize prompts!
```

**Answer:** 2-5x more

---

### Question 5: Modifying a Flow

Open `4_simple_agent.yaml` in the Kestra flow editor. Find the `english_brevity` task and change its prompt from asking for exactly 1 sentence to asking for exactly 3 sentences.

Save the flow, then run it with `summary_length = long`.

Compare the `english_brevity` output token count to the original 1-sentence version (also with `summary_length = long`). How do they compare?

- About the same (within 20%)
- 2-4x more
- 5-10x more
- 10x+ more

```
  - id: english_brevity
    type: io.kestra.plugin.ai.agent.AIAgent
    prompt: |
      Generate exactly 1 sentence English summary of the following:
      "{{ outputs.multilingual_agent.textOutput }}"
```

```
2026-07-01 21:37:52.242
📊 Token Usage Summary:

Multilingual Agent:
- Input tokens: 282
- Output tokens: 165
- Total tokens: 447

English Brevity Agent:
- Input tokens: 180
- Output tokens: 34
- Total tokens: 214

💡 Tip: Monitor token usage to understand costs and optimize prompts!
```

```
  - id: english_brevity
    type: io.kestra.plugin.ai.agent.AIAgent
    prompt: |
      Generate exactly 3 sentences English summary of the following:
      "{{ outputs.multilingual_agent.textOutput }}"
```

```
2026-07-01 21:39:12.112
📊 Token Usage Summary:

Multilingual Agent:
- Input tokens: 282
- Output tokens: 151
- Total tokens: 433

English Brevity Agent:
- Input tokens: 166
- Output tokens: 86
- Total tokens: 252

💡 Tip: Monitor token usage to understand costs and optimize prompts!
```

**Answer:** 2-4x more

---

### Question 6: Best Practices

Based on what you learned in this module, for production workflows requiring deterministic, repeatable results with strict compliance requirements (e.g., financial reporting, workflows in highly regulated industries), which approach is most appropriate?

- Always use AI agents for maximum flexibility and adaptation
- Use traditional task-based workflows for predictability and auditability
- Use only RAG without agents for better performance
- Use web search tools exclusively to ensure current data

**Answer:** Use traditional task-based workflows for predictability and auditability
