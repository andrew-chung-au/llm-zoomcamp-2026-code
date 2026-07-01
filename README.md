# LLM Zoomcamp 2026 — Learning in Public Log

This repository is a learning-in-public artifact tracking my technical implementations and architectural takeaways through DataTalksClub's **LLM Zoomcamp (2026 Edition)**.

---

## 🏗 Repository Structure

The code and notes are organized by module as I move through the curriculum:

*   **`01-agentic-rag/`** — Foundations of agentic loops, context boundaries, and managing non-deterministic reasoning steps.
*   **`02-vector-search/`** — Data engineering pipelines for retrieval, text embeddings, and custom search indexes.
*   **`03-orchestration/`** — Orchestration harnesses, state isolation, secrets hygiene, and event-driven control planes using open-source platforms like Kestra.

---

## 🏗 Repository Structure

The code and notes are organized by module as I move through the curriculum:

*   **`01-agentic-rag/`** — Foundations of agentic loops, context boundaries, and managing non-deterministic reasoning steps.
*   **`02-vector-search/`** — Data engineering pipelines for retrieval, text embeddings, and custom search indexes.
*   **`03-orchestration/`** — Orchestration harnesses, state isolation, secrets hygiene, and event-driven control planes using open-source platforms like Kestra.

---

## 🛠 Tech Stack & Tooling

*   **Core Toolkit:** Python, SQL
*   **Focus Areas:** Context Engineering, RAG Pipelines, Multi-Agent Coordination, Orchestration Harnesses.
*   **Dependency Management:** Managed via `uv` (tracked in `uv.lock` and `pyproject.toml`). To initialize and synchronize the environment locally, run:
    ```bash
    uv sync
    ```