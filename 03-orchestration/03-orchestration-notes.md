# 01 intro (AI Orchestration with Kestra)

## Orchestrating AI Workflows

---

### 1. Core Concept: What is Kestra?
* **Definition:** Kestra is an open-source, event-driven orchestration platform used to build, run, scale, and monitor data pipelines and workflows.
* **Analogy:** Think of it like a highly structured "conductor" for your code and data tasks. Instead of running Python scripts manually or relying on basic cron jobs, Kestra manages the execution order, handles retries, and securely passes data between steps. 
* **Format:** Workflows (called "Flows") are defined using declarative YAML. This allows you to outline the sequence of tasks without writing the underlying execution engine logic from scratch.

---

### 2. Ecosystem Context: Where Kestra Fits
* **The AWS Mental Model:** Architecturally, Kestra behaves a lot like AWS Step Functions. Both orchestrate discrete tasks, manage state passing, and handle retries so you don't have to write the execution loop yourself (using declarative YAML instead of Amazon States Language JSON). However, while Step Functions is proprietary and tightly coupled to AWS (gluing together Lambda, ECS, DynamoDB), Kestra is open-source and infrastructure-agnostic. 
* **The Data Engineering Heavyweights:** Incumbent industry standards for ETL/ELT pipelines like Apache Airflow, Prefect, and Dagster fundamentally require workflows to be defined as Python Directed Acyclic Graphs (DAGs). Kestra challenges this by separating the orchestration layer from the execution layer.
* **Key Platform Traits:**
    * **Polyglot:** Because the orchestration is YAML, the individual tasks can run existing Python scripts, SQL queries, or Bash commands natively. This allows someone to execute a Python ingestion script and a BigQuery SQL script within the same workflow without needing to wrap everything in a Python framework.
    * **Event-Driven:** Instead of just running on a schedule, it natively responds to events like a file dropping into an S3 bucket, a webhook trigger, or a database change.

---

### 3. The Context Problem: AI Assistants vs. Workflow AI
* **The Limitation:** Traditional AI assistants (like ChatGPT or Gemini in a browser) do not have context about your specific codebase, real-time system data, or the latest API documentation.
* **The Result:** Without this context, generic LLMs tend to hallucinate outdated plugin syntax or incorrect properties that cannot be trusted in production.
* **The Solution:** By embedding AI directly into the Kestra platform, we can provide the model with the exact context it needs (valid properties, current docs) to get reliable results.

---

### 4. Why Use AI for Workflows?
Integrating AI into your orchestration saves time on boilerplate code and documentation searches. It helps to:
* **Generate Workflows Faster:** You can describe the pipeline you want in natural language instead of writing the YAML from scratch.
* **Avoid Errors:** AI tools that are grounded in the right context output syntax-correct, up-to-date code that follows best practices.
* **Automate Decisions:** AI Agents can dynamically orchestrate tasks based on changing conditions, rather than forcing you to map out every possible sequence in advance.
* **Ground Responses in Data:** Implementing Retrieval Augmented Generation (RAG) ensures that when the AI answers questions or makes decisions, it is using accurate, real data.

---

### 5. Key Techniques Covered in this Module
This module focuses on three distinct approaches to utilizing AI within workflows:
* **AI Copilot:** Generating and refining flows.
* **RAG Workflows:** Grounding AI responses in real data.
* **AI Agents:** Designing autonomous task execution and multi-agent systems.

---

### 6. Agent Frameworks vs. Orchestrators (The "Brain" vs. The "Nervous System")
A common pitfall when transitioning AI projects from proof-of-concept to production is relying entirely on Python scripts to manage reliability. 
* **The "Brain" (Agent Frameworks):** Frameworks like Google ADK, LangChain, or CrewAI are designed to manage the internal cognitive logic of the LLM. They handle prompt templating, memory, and tool binding. However, an agent framework is ultimately just an application script; if an underlying network tunnel drops or an API rate limit is hit, the script crashes.
* **The "Nervous System" (Orchestrators):** Orchestrators like Kestra act as the control plane. They do not dictate *how* the agent thinks; they dictate *when* and *where* it runs, and how to recover from failure.
    * **Infrastructure Prerequisites:** Kestra handles pulling secure secrets and establishing necessary connections before the agent executes.
    * **System-Level Retries:** If a network connection drops mid-execution, the orchestrator catches the failure at the infrastructure level, triggering an exponential backoff retry without requiring custom while-loops in the Python logic.
    * **Separation of Concerns:** By moving the "plumbing" (scheduling, state persistence, error retries) to the orchestrator, the Python code remains strictly focused on the AI logic.

---

### 7. Access, Cost, and Deployment
* **Open-Source (Free):** Kestra is fundamentally an open-source platform (Apache 2.0 license). It can be run entirely for free by deploying it locally via Docker or hosting it on your own infrastructure (e.g., a Kubernetes cluster). This grants full access to the orchestration engine, declarative YAML flows, and all standard plugins (including the AI tools).
* **Enterprise / Cloud (Paid):** For organizations moving into large-scale production, Kestra offers a paid Enterprise edition and a fully managed Cloud platform. These tiers introduce enterprise-grade security features like Single Sign-On (SSO), Role-Based Access Control (RBAC), High Availability (HA), and dedicated support SLAs, typically billed on a usage-based model.

---

### 8. Industry Adoption & Portfolio Impact
* **Wide Adoption:** Kestra is used in mission-critical environments by organizations including Apple, JPMorgan Chase, Bloomberg, and Leroy Merlin, with over 20,000 stars on GitHub.
* **Employer Value:** Using Kestra in a project signals professional "GitOps" discipline, infrastructure-agnostic engineering, and a focus on production-grade observability. It demonstrates that you can build resilient, observable systems rather than just simple scripts.
* **Automation Engineering:** Unlike no-code tools (Zapier, Power Automate, n8n) focused on SaaS app integration, Kestra is a production-grade orchestrator. For an automation engineering role, Kestra is superior for handling complex branching, step-level checkpointing, and version-controlled, production-ready backend automation.

---

# 02 Context Engineering

## Bridging the Gap Between Generic LLMs and Production Workflows

---

### 1. The Context Problem
* **The Failure Mode:** When you ask a generic AI to write a complex workflow, it often produces code that is "syntactically correct" but functionally flawed or non-idiomatic.
* **Common Pitfalls:**
    * **Outdated Syntax:** Generating task types that have been deprecated or renamed.
    * **Hallucinated Features:** Inventing triggers or properties that never existed.
    * **Misconfiguration:** Placing properties in the wrong hierarchy (e.g., top-level vs. nested under `csvOptions`).

---

### 2. Why Hallucination Happens
* **Training Cutoff:** LLMs rely on historical data and lack "live" awareness of the latest software updates, API changes, or internal organizational best practices.
* **Lack of Grounding:** Without access to the specific documentation or the codebase context, the model "guesses" based on general patterns rather than evaluating against the current environment.

---

### 3. Context Engineering: The Solution
* **Definition:** The systematic design of the information environment surrounding AI models. It involves structuring the data and instructions so the model has the right knowledge, tools, and constraints before it even begins to respond.
* **The Principle:** AI is only as good as the context you provide. Providing valid property names, current documentation, and environment-specific standards forces the model to work within known, valid constraints.
* **Architecture-Centric Thinking:** Unlike prompt engineering, which is user-facing and aims for a single "perfect" prompt, context engineering is developer-facing. It focuses on building the infrastructure (RAG, structured knowledge, tool definitions) that supplies the model with relevant data at the right time.

---

# 03 Setting Up Kestra

## Reviewing the Setup

This module explains how to run Kestra locally and use it to execute the example flows for this orchestration module.

---

### 1. Prerequisites
- **Docker** with Docker Compose is required to run Kestra locally.
- Docker Desktop is the easiest option on Mac and Windows because it includes both.
- Docker should be installed before proceeding.

---

### 2. Start Kestra
- The module includes a pre-configured `docker-compose.yml` for Kestra.
- From the `03-orchestration` directory, run:

```bash
cd 03-orchestration
docker compose up -d
```

- Once the container starts, access the Kestra UI at [http://localhost:8080](http://localhost:8080/).
- To stop Kestra, run:

```bash
docker compose down
```

---

### 3. Log In to Kestra
- Open the Kestra UI at [http://localhost:8080](http://localhost:8080/).
- Sign in with the local Kestra credentials unless you changed them.
- Local OSS login `admin@kestra.io` / `Admin1234!`.
- You need to be logged in before running flows.

---

### 4. Get API Keys
- A **Gemini API key** is required for the module.
- It can be created in [Google AI Studio](https://aistudio.google.com/app/apikey).
- The free tier is enough for light use, but quota limits may be reached if the agent and multi-agent flows are run repeatedly.
- If `429 Resource Exhausted` appears, wait briefly before retrying, or use a paid tier.
- Flow 3 now also uses **Gemini** instead of OpenAI, so no OpenAI API key is needed for this setup.
- **Gemini 3.1 Flash Lite** (gemini-3.1-flash-lite) has been used in the flows due to more generous free rate limits.

- A **Tavily API key** is required for web search in flows 3, 5, and 6.
- It can be created at [Tavily](https://tavily.com/).
- The free tier includes 1,000 searches per month.

---

### 5. Configure Secrets in Kestra
- Kestra reads secrets from environment variables that use the `SECRET_` prefix.
- Secret values must be base64-encoded before Kestra can use them.
- Before starting Kestra, export the keys like this:

```bash
export GEMINI_API_KEY="your-gemini-api-key-here"
export SECRET_GEMINI_API_KEY=$(echo -n $GEMINI_API_KEY | base64)
export SECRET_OPENAI_API_KEY=$(echo -n "your-openai-api-key-here" | base64)
export SECRET_TAVILY_API_KEY=$(echo -n "your-tavily-api-key-here" | base64)
```

- Then restart Kestra:

```bash
docker compose up -d
```

- In the flows, reference secrets like this:

```yaml
{{ secret('GEMINI_API_KEY') }}
```

- The `SECRET_` prefix is omitted when calling `secret()`.
- API keys should never be committed to Git.

---

### 6. Using a `.env` File Safely
- A local `.env` file can be used for convenience during setup.
- The file should stay on the local machine and should not be committed.
- Add `.env` to `.gitignore` so Git does not track it.
- The `.env` file is useful when keys need to be reused across sessions.

#### Example `.env`
```env
SECRET_GEMINI_API_KEY=base64-encoded-gemini-key
SECRET_OPENAI_API_KEY=base64-encoded-openai-key
SECRET_TAVILY_API_KEY=base64-encoded-tavily-key
```

#### Load the `.env` file
- If all variables should be loaded into the current terminal session:

```bash
set -a
source .env
set +a
```

- If only a subset is needed, export only the required variables instead of loading everything.

#### Load only a subset
```bash
export SECRET_GEMINI_API_KEY=$(grep '^SECRET_GEMINI_API_KEY=' .env | cut -d= -f2-)
export SECRET_OPENAI_API_KEY=$(grep '^SECRET_OPENAI_API_KEY=' .env | cut -d= -f2-)
docker compose up -d
```

#### Unset unneeded values
```bash
unset SECRET_TAVILY_API_KEY
```

#### Safety notes
- Never paste real keys into a shared notebook, README, or GitHub commit.
- Restart Kestra after changing environment values so it picks them up.
- A `.env` file is fine for local course work, but a proper secrets manager is safer for production.

---

### 7. Import Example Flows
- The example flow YAML files are already included in `03-orchestration/flows/`.
- From the `03-orchestration` directory, upload them into Kestra with `curl`.
- Update the username and password if the Kestra setup is different.

```bash
cd 03-orchestration

curl -X POST -u 'admin@kestra.io:Admin1234!' http://localhost:8080/api/v1/flows/import -F fileUpload=@flows/1_chat_without_rag.yaml
curl -X POST -u 'admin@kestra.io:Admin1234!' http://localhost:8080/api/v1/flows/import -F fileUpload=@flows/2_chat_with_rag.yaml
curl -X POST -u 'admin@kestra.io:Admin1234!' http://localhost:8080/api/v1/flows/import -F fileUpload=@flows/3_rag_with_websearch.yaml
curl -X POST -u 'admin@kestra.io:Admin1234!' http://localhost:8080/api/v1/flows/import -F fileUpload=@flows/4_simple_agent.yaml
curl -X POST -u 'admin@kestra.io:Admin1234!' http://localhost:8080/api/v1/flows/import -F fileUpload=@flows/5_web_research_agent.yaml
curl -X POST -u 'admin@kestra.io:Admin1234!' http://localhost:8080/api/v1/flows/import -F fileUpload=@flows/6_multi_agent_research.yaml
```

- The YAML can also be pasted directly into the Kestra UI.

---

### 8. Run the First Agent
- Open the Kestra UI at [http://localhost:8080](http://localhost:8080/).
- Login (local OSS login: `admin@kestra.io` / `Admin1234!`).
- Navigate to the `zoomcamp` namespace.
- Find the `4_simple_agent` flow and click **Execute**.
- Leave the default inputs or customize them.
- Watch the execution and review the outputs.
- Then run `5_web_research_agent` and `6_multi_agent_research` and compare the logs and outputs.

# 04 AI Copilot & Workflow Generation

## Accelerating Pipeline Development with Grounded AI

---

### 1. The Challenge of Manual Workflow Construction
* **High Cognitive Load:** Building workflows manually requires deep knowledge of specific plugin types, exact property naming conventions, and correct YAML syntax.
* **Efficiency Bottlenecks:** Time spent looking up documentation and connecting tasks one-by-one is significant before the flow is even executable.
* **The "Boilerplate" Problem:** Much of development time is spent on repetitive task configuration rather than focusing on the core business logic.

---

### 2. AI Copilot: The "Grounded" Difference
* **Architectural Awareness:** Unlike standard chat where an AI is "blind" to your current work, the Kestra AI Copilot injects your live YAML flow into its prompt context. It sees exact task IDs, plugin names, and variable structures in real-time, ensuring refinements are syntactically valid within your specific file.
* **Grounded Generation:** Kestra's AI Copilot is grounded in the live documentation of your specific Kestra deployment. It does not "guess" syntax based on stale training data; it references the active schema definition of the plugins you are actually using.
* **Pre-Engineered System Instructions:** Kestra’s engineers have hard-coded instructions into the Copilot’s system message, ensuring the model acts as an expert that preserves existing YAML structure and only outputs valid plugin types, removing the need for you to constantly remind the AI to "keep the namespace."

---

### 3. Setup and Access
* **Provider Note:** In Kestra’s Open Source edition, the AI Copilot currently supports Gemini as its primary AI provider.
* **Configuration:** You must configure Gemini API access as a secret within your Kestra instance to enable the Copilot.
* **Interface:** Access is built directly into the Kestra UI (at http://localhost:8080) via the sparkle icon (✨) located in the top-right corner of the Flow Editor.

---

### 4. The "5%" Collaboration Rule
* **The Bulk Work:** Copilot automates 95% of the boilerplate structure, which is the most time-consuming part of flow creation.
* **The Human Gap:** You are responsible for the final 5%—the specific environmental details (secrets, variables, environment-specific SLAs) that the AI cannot know without direct access to your setup.
* **Iterative Refinement:** Because the conversation is cumulative, you can refine flows incrementally (e.g., adding triggers or error handling) without discarding your existing work.

---

### 5. Integration Beyond the UI
* **Agent Skills:** For developers working in external IDEs like Cursor or Claude, Kestra provides an `agent-skills` repository. This brings Kestra-specific "grounding" (up-to-date plugin docs, valid properties, best practices) to external coding agents, enabling reliable flow generation without needing to switch to the Kestra UI.

---

# 05 Retrieval Augmented Generation (RAG)

### Overview: What is RAG?
**Retrieval Augmented Generation (RAG)** is a technique that retrieves relevant information from external data sources, augments the AI prompt with that context, and generates a response grounded in real data. 

* **The Problem:** LLMs often suffer from hallucinations, lack internal data access, or possess outdated information.
* **The Solution:** RAG solves this by ensuring the AI has access to current, accurate, and private information at query time.

---

### How RAG Works in Kestra
While production environments typically schedule these phases separately (**ingest on a cadence, query on demand**), demo workflows often run them back-to-back.

#### 1. Ingest Phase
*Run once, or on a schedule when your data changes.*
1.  **Fetch Documents:** Load external documentation, release notes, or other data sources.
2.  **Create Embeddings:** Convert the text into vectors using an embedding model.
3.  **Store Embeddings:** Save vectors in Kestra's KV Store.

> ⚠️ **Production Warning:** Storing embeddings in Kestra's KV Store is designed for simplicity, learning, and small-scale demos. It is **not** a replacement for a proper vector database. For serious workloads (large document sets, low-latency retrieval), use a dedicated vector store.

#### 2. Query Phase
*Runs every time a question is asked.*
1.  **Retrieve Context:** Find the stored embeddings most similar to the user's question.
2.  **Augment Prompt:** Inject the retrieved content/context directly into the LLM prompt.
3.  **Generate Response:** The LLM answers using the real, grounded context provided.

---

### Implementation Examples

#### Step 1: Without RAG
* **Flow:** `1_chat_without_rag.yaml`
* **Prompt:** *"Which features were released in Kestra 1.1?"*
* **Result:** **Inaccurate.** Without RAG, the model is prone to hallucinating features, providing outdated information, or giving vague, generic answers.

#### Step 2: Static RAG
* **Flow:** `2_chat_with_rag.yaml`
* **Process:** Ingests the Kestra 1.1 release blog post from GitHub $\rightarrow$ Creates embeddings via Gemini $\rightarrow$ Stores them in Kestra's KV Store $\rightarrow$ Queries the LLM with context enabled.
* **Result:** **Accurate.** Returns real features grounded explicitly in the release notes.

#### Step 3: Web Search RAG
* **Flow:** `3_rag_with_websearch.yaml`
* **Process:** Uses the `TavilyWebSearch` retriever to fetch live web results at query time and inject them as context. **No ingestion step required.**

---

### Static RAG vs. Web Search RAG

| Feature | Static RAG | Web Search RAG |
| :--- | :--- | :--- |
| **Data Source** | Documents you manually ingested | Live web results |
| **Best For** | Internal docs, policies, fixed knowledge bases | Time-sensitive or frequently changing info |
| **Ingestion Step**| Required | Not required |
| **Example Query** | *"What does our refund policy say?"* | *"What is the latest release of Kestra?"* |

> 📌 **Rule of Thumb:** Use **Static RAG** when you control the source material. Use **Web Search RAG** when the answer depends on information changing faster than your ingestion pipelines can handle.

---

### Best Practices
* **Keep documents updated:** Re-ingest regularly so your vector store/KV store reflects current information.
* **Chunk appropriately:** Break large documents down into smaller, meaningful sections before ingesting to improve search precision.
* **Test retrieval quality:** Verify that the correct documents are actually being pulled for your queries.
* **Choose the right retriever:** Match your use case—static RAG for controlled knowledge bases, web search for live, dynamic data.

---

# 06 AI Agents

## The Core Paradigm: Model + Harness = Agent

* **The Model:** The core intelligence layer (the LLM itself), which excels at reasoning but is inherently non-deterministic.
* **The Harness:** The runtime scaffolding and infrastructure surrounding the model (state, memory, tools, and execution loops).
* **The Agent:** The model operating *inside* this harness to dynamically achieve a declarative goal.

In Module 1, you built the agentic loop by hand: a manual `while` loop that called the LLM, executed any tool calls it returned, sent the results back, and stopped when the model produced a final answer.

In Kestra, the **`AIAgent` plugin** abstracts this entire runtime engine into a production-grade infrastructure component. You define the goal, the tools, and optionally a system message. Kestra drives the loop, manages conversation history, and surfaces the result as a task output.

---

## Traditional vs. AI Agentic Workflows

| Approach | Architecture & Control Plane | Best Used For | Trade-offs |
| :--- | :--- | :--- | :--- |
| **Traditional Workflows** | **Imperative (Deterministic):** Fixed sequence, predetermined logic. The developer pre-defines the exact execution sequence, parameters, and error handling. | Repeating ETL/ELT pipelines, strict compliance processes, predictable data syncs. | Minimum latency, completely predictable costs, explicit auditability. |
| **AI Agents** | **Declarative (Goal-Oriented):** The developer provides a goal and tools; the LLM dynamically evaluates state and decides the pathing at runtime. | Unstructured web research, dynamic alert routing, data patching, adaptive failure remediation. | Non-deterministic pathing, compounding token costs, variable latency. |

### Code Comparison

**Traditional Workflow**

```yaml
tasks:
  - id: step1
    type: Task1
  - id: step2
    type: Task2
```

**AI Agent Workflow**

```yaml
tasks:
  - id: agent
    type: io.kestra.plugin.ai.agent.AIAgent
    prompt: "Research data engineering trends and create a report"
    tools:
      - WebSearch
      - TaskExecution
```

---

## Anatomy of an AI Agent

Here is how a complete agent is structured in Kestra.

> **Note:** The example below uses Google Gemini (`io.kestra.plugin.ai.provider.GoogleGemini`), but the provider block supports any major AI provider (OpenAI, Anthropic, etc.). Ensure you have configured your platform vault secrets (e.g., `GEMINI_API_KEY`) before executing.

```yaml
id: example_agent
namespace: zoomcamp
tasks:
  - id: agent
    type: io.kestra.plugin.ai.agent.AIAgent

    # System Message: Establishes persona, boundaries, and formatting constraints
    systemMessage: |
      You are a data analyst. Analyze data and provide insights.

    # Prompt: The core unstructured directive/goal
    prompt: "What are the top 3 trends in this data?"

    # Provider: Abstraction layer for the LLM
    provider:
      type: io.kestra.plugin.ai.provider.GoogleGemini
      modelName: gemini-2.5-flash
      apiKey: "{{ secret('GEMINI_API_KEY') }}" # Secrets hygiene managed via vault

    # Tools: Explicitly exposed capabilities the model can choose to call
    tools:
      - type: io.kestra.plugin.ai.tool.TavilyWebSearch
        apiKey: "{{ secret('TAVILY_API_KEY') }}"

    # Memory: Persistent KV state tracking context across separate workflow executions
    memory:
      type: io.kestra.plugin.ai.memory.KestraKVStore
      memoryId: analyst_001
```

---

## Execution Mechanics: Retrievers vs. Tools

Understanding how context gets inside the harness is critical to keeping token usage efficient:

* **Content Retrievers:** These run unconditionally up front to inject fresh context into the prompt before the model evaluates its goal.
* **Tools:** These are conditional and non-deterministic. They are invoked only if the LLM analyzes the prompt and decides it lacks information or capabilities to proceed.

### The Execution Loop in Practice

In an advanced web research flow (e.g., `5_web_research_agent.yaml`), the agent autonomously decides when to use tools inside a strict **Search → Evaluate → Pivot/Loop** cycle:

1. Receives a research prompt.
2. Decides to use the web search tool to gather information.
3. Evaluates search results and determines if more searches/queries are needed to clear its confidence threshold.
4. Synthesizes findings into a structured markdown report.
5. Saves the report to a file using the filesystem tool.

> **Note:** For basic implementations, flows like `4_simple_agent.yaml` demonstrate text summarization, chaining agent tasks, and using `pluginDefaults` to avoid repetition.

---

## Tooling & Extensibility

Kestra abstracts tool interoperability through specialized plugins and modern open-source protocols.

### Key Extensibility Concepts

* **Deterministic Compute:** LLMs struggle with precision logic (e.g., statistical calculations, SHA-256 generation). Kestra handles this via `CodeExecution`, running model-generated snippets inside an isolated Judge0 sandbox to ensure exact math.
* **Model Context Protocol (MCP):** Connects the agent directly to external environments and containerized microservices via standard clients.
* **Multi-Agent Networks:** Employs nesting logic where an orchestrating agent calls a secondary, highly-specialized sub-agent (`AIAgent`) as a standard tool.

### Agent Tools Available in Kestra

| Tool | Purpose | Example Use |
| :--- | :--- | :--- |
| `TavilyWebSearch` | Search the web for current information | Market research, news monitoring |
| `GoogleCustomWebSearch` | Search with Google Custom Search API | General web scraping and search |
| `CodeExecution` | Run code safely via Judge0 | Math calculations, data manipulation/validation |
| `KestraTask` | Execute any Kestra task | Run tasks based on 1000+ Kestra plugins |
| `KestraFlow` | Trigger other Kestra flows | Call other flows for modularity |
| `StreamableHttpMcpClient` | Use MCP servers via HTTP/SSE | Connect to remote MCP servers |
| `DockerMcpClient` | Use MCP servers in Docker | MCP servers spun up on-demand via Docker |
| `StdioMcpClient` | Use MCP servers via stdio | Integration with external/local systems |
| `AIAgent` | Use another agent as a tool | Multi-agent systems, separating concerns |

---

## Production-Grade Agent Observability

Because agents make highly dynamic, multi-turn LLM requests, real-time logging is a core requirement to avoid cascading failures or token drift. Kestra provides full observability, including execution times, token counters, and raw JSON outputs.

Enable detailed logging via the configuration property:

```yaml
tasks:
  - id: research_agent
    type: io.kestra.plugin.ai.agent.AIAgent
    description: Autonomous research agent with web search capabilities
    provider:
      type: io.kestra.plugin.ai.provider.GoogleGemini
      apiKey: "{{ secret('GEMINI_API_KEY') }}"
      modelName: gemini-2.5-flash
    configuration:
      logRequests: true  # Traces prompt states and exact tool parameters sent to the LLM
      logResponses: true # Captures the model's raw JSON outputs and token counters
```

> **Review Takeaway:** Productionizing agents is a distribution and infrastructure problem. By configuring explicit request/response logging, you ensure strict tracking of token ceilings, request latency, and raw execution paths across your autonomous loops.

---

# 07 Multi-Agent Systems

## Why Multi-Agent?

A single agent with too many responsibilities (searching, analyzing, formatting, validating) tends to get unreliable — its system prompt grows bloated, it gets confused about which tool to use when, and failures are hard to trace back to a root cause.

Multi-agent systems solve this by splitting a complex task across **specialized agents**, each with a narrow, well-defined job. One agent can call another **as a tool**, exactly like it would call a web search or a database query. This turns "agent orchestration" into just another instance of the tool-calling pattern you already know from single-agent flows — nothing structurally new, just a different kind of tool.

### Core Benefits

* **Separation of concerns:** each agent focuses on one thing (research vs. analysis vs. formatting), so prompts stay simple and predictable.
* **Easier debugging:** when something breaks, you can isolate the failure to a specific agent's logs rather than untangling one giant prompt.
* **Reusability:** a well-scoped agent (e.g. a "web research agent") can be reused across multiple higher-level flows as a plug-in capability.

---

## Example: Company Research

Flow: [`6_multi_agent_research.yaml`](https://github.com/DataTalksClub/llm-zoomcamp/blob/main/03-orchestration/flows/6_multi_agent_research.yaml)

This flow implements a two-agent system for competitor research.

| Agent | Specialization | Tools | Responsibility |
| :--- | :--- | :--- | :--- |
| **Research Agent** | Web research and data gathering | Tavily web search | Find factual, current information |
| **Main Analyst Agent** | Analysis and synthesis | Research Agent (used as a tool) | Create structured reports |

### How It Works

1. **Input:** a company name (e.g., `"kestra.io"`).
2. The **main agent** receives the prompt: *"Research this company."*
3. The main agent calls the **research agent** as a tool: *"Find information about kestra.io."*
4. The research agent uses Tavily to gather data from the live web.
5. The research agent returns its findings to the main agent.
6. The main agent synthesizes the findings into a final structured **JSON output**.

### The Key Pattern: AIAgent as a Tool

The main agent doesn't know (or care) that the research agent is itself an LLM running its own reasoning loop — it just sees a callable tool that takes a query and returns results. This is the same abstraction Kestra uses for `TavilyWebSearch` or `CodeExecution`. That consistency is what makes nesting agents so composable: you can chain or nest them arbitrarily deep without changing how the orchestrating agent is designed.

```yaml
tools:
  - type: io.kestra.plugin.ai.agent.AIAgent
    id: research_agent
    prompt: "Find information about {{ inputs.company }}"
    tools:
      - type: io.kestra.plugin.ai.tool.TavilyWebSearch
        apiKey: "{{ secret('TAVILY_API_KEY') }}"
```

---

## Best Practices

* **Define clear responsibilities:** each agent should have a specific role and stay within it — avoid letting one agent "help out" with another's job, as this defeats the separation-of-concerns benefit.
* **Monitor token usage:** every agent hop is its own set of LLM calls, so costs and latency compound quickly in nested systems. Use `logRequests`/`logResponses` (from the observability lesson) on each agent to track this.
* **Document agent purposes:** describe each agent's role directly in flow and task `description` 

---

# 08 Best Practices for Production AI Workflows

This lesson consolidates the decision-making, cost, security, and operational practices you need before running any AI-driven flow (RAG, agents, or multi-agent systems) in production.

## When to Use What

Choosing the right tool for the job is the single biggest lever for keeping AI workflows cheap, reliable, and maintainable. As a rule of thumb: reach for AI only when the task genuinely requires judgment or adaptability — default to deterministic tools otherwise.

| Scenario | Use This | Why |
| :--- | :--- | :--- |
| Creating/editing flows | AI Copilot | Fastest way to generate YAML flow code |
| Answering questions about your data | RAG | Grounds responses in real data |
| Fixed, repeatable ETL pipelines | Traditional workflows | Deterministic, predictable, compliant |
| Research and analysis tasks | AI Agents | Can adapt to findings and make decisions |
| Complex, multi-step objectives | Multi-agent systems | Specialized agents working together |

---

## Cost Considerations

AI features run on LLM APIs, and every request/response consumes billable **tokens** — chunks of text roughly a few characters each. Costs scale with both input tokens (your prompt + context) and output tokens (the model's response), and output tokens are typically far more expensive.

Pricing per 1M tokens ([full pricing page](https://ai.google.dev/gemini-api/docs/pricing)):

| Model | Tier | Input | Output |
| :--- | :--- | :--- | :--- |
| Gemini 2.5 Flash | Free | $0.00 | $0.00 |
| Gemini 2.5 Flash | Batch / Flex | $0.15 | $1.25 |
| Gemini 3.5 Flash | Free | $0.00 | $0.00 |
| Gemini 3.5 Flash | Standard | $1.50 | $9.00 |
| Gemini 3.5 Flash | Batch / Flex | $0.75 | $4.50 |
| Gemini 3.5 Flash | Priority | $2.70 | $16.20 |

**Guideline:** use Gemini 2.5 Flash for most workflows — it's cheaper and free for standard inference. Step up to Gemini 3.5 Flash only when you need stronger reasoning for complex agent tasks, since the cost jump is significant (e.g., standard-tier output is 7.2x more expensive).

### Cost-Saving Tips

* Start with the free tier for learning and development.
* Use smaller/cheaper models for simple tasks — check the [pricing page](https://ai.google.dev/gemini-api/docs/pricing).
* Set `maxOutputTokens` to cap response size and prevent runaway generation costs.
* Monitor token usage in execution outputs to catch inefficient prompts early.
* Use traditional workflows when determinism is needed — don't pay for reasoning you don't require.

---

## Security

Treat API keys the same way you'd treat AWS credentials or a database password — never hardcode them, and never commit them to Git.

```yaml
# ❌ Wrong
apiKey: "sk-abc123def456"

# ✅ Correct
apiKey: "{{ secret('GEMINI_API_KEY') }}"
```

Export base64-encoded keys as `SECRET_`-prefixed environment variables before starting Kestra — this is how Kestra's secret manager picks them up at runtime without ever exposing raw values in your flow YAML. Rotate keys regularly (e.g., every 90 days) and monitor usage for anomalies that might indicate a leaked key. Read more in the [Kestra secrets documentation](https://kestra.io/docs/concepts/secret).

---

## Observability and Debugging

You've already seen `logRequests`/`logResponses` in the agent observability lesson — this section is about using them systematically as a debugging workflow, not just a one-off toggle.

```yaml
- id: my_agent_task
  type: io.kestra.plugin.ai.agent.AIAgent
  provider:
    # ...
    # provider settings
    # ...
  configuration:
    logRequests: true
    logResponses: true
```

What to monitor:

* Token usage per execution (cost tracking)
* Agent tool calls and decisions (did it pick the right tool, and why?)
* Execution time and costs (latency budgeting)
* Output quality (is the model actually solving the task well?)

### Debugging Tips

* Start with simple prompts and iterate — don't debug a complex multi-agent chain from scratch.
* Check logs for LLM reasoning — the raw request/response traces often reveal *why* a wrong tool was chosen.
* Verify tool execution outputs independently — a failure can originate in the tool itself, not the model's decision to call it.

---

## Production Readiness

Before deploying AI workflows to production, run through this checklist. Unlike traditional pipelines, AI workflows need extra scrutiny because their non-deterministic nature means "it worked once" isn't sufficient proof of reliability.

* **Test thoroughly** — run multiple times with different inputs; verify outputs are consistent and accurate across variations, not just on your happy-path example.
* **Add fallbacks** — handle API failures with retries and configure alerts on failure, since LLM APIs can have transient outages or rate limits.
* **Set limits** — cap `maxOutputTokens` to control costs and prevent runaway generations.
* **Document behavior** — explain what the agent does in your flow and task descriptions, so the next person (or future you) can maintain it without re-reverse-engineering the prompt logic.

# 09 Module Recap & Next Steps

## Module Recap

Across this module, you moved from foundational concepts to production-ready practices for building AI into Kestra workflows:

* **Context engineering** — why generic AI assistants fail without it, and how grounding matters for reliable output.
* **AI Copilot** — generating and refining flows by describing inputs and goals in natural language, rather than hand-writing YAML from scratch.
* **RAG (Retrieval-Augmented Generation)** — building AI responses grounded in your own data sources, reducing hallucination risk versus relying on a model's parametric knowledge alone.
* **AI Agents** — autonomous systems that use tools and make dynamic decisions, extending the manual agentic loop from Module 1 into Kestra's managed `AIAgent` runtime.
* **Multi-agent systems** — specialized agents collaborating on complex, multi-step objectives via the "agent-as-tool" pattern.
* **Best practices** — cost control, secrets management, observability, and production-readiness checklists.

Together, these give you the building blocks to use AI across the **full workflow lifecycle**: generating flows faster, answering questions from your own data, and automating tasks that don't follow a fixed sequence of steps. That last point is the real shift this module represents — moving from purely imperative pipelines to workflows that can adapt at runtime.

---

## Where to Go From Here

* **Experiment with different LLM providers.** The flows in this module use Gemini, but Kestra's AI plugin supports other providers too (OpenAI, Anthropic, etc.). Swapping providers is usually just a change to the `provider` block — try comparing output quality, latency, and cost across a couple of models for the same task.
* **Build custom agents for your data pipelines.** Take a workflow you already run and ask: *which parts involve decisions that depend on external data?* Those are good agent candidates — anywhere you'd currently write a big conditional (`if/else`) tree based on unpredictable inputs is a strong signal.
* **Explore Kestra Blueprints.** The [Blueprints library](https://kestra.io/blueprints) has pre-built workflow examples you can import and adapt, including AI and agent patterns — a fast way to see production-shaped examples beyond this course's flows.
* **Share your learnings.** Join the [Kestra Slack community](https://kestra.io/slack) to ask questions, share what you've built, and see what others are working on. Given you're already documenting your LLM Zoomcamp journey publicly, this is a natural extension — sharing a working multi-agent flow there could also be good networking material for your AI engineering transition.

---

## Kestra Documentation

* [AI Tools Overview](https://kestra.io/docs/ai-tools)
* [AI Copilot](https://kestra.io/docs/ai-tools/ai-copilot)
* [AI Agents](https://kestra.io/docs/ai-tools/ai-agents)
* [RAG Workflows](https://kestra.io/docs/ai-tools/ai-rag-workflows)
* [AI Plugin](https://kestra.io/plugins/plugin-ai)
* [AI Agent Task](https://kestra.io/plugins/plugin-ai/agent)
* [RAG Tasks](https://kestra.io/plugins/plugin-ai/rag)

## External Resources

* [Google Gemini API](https://ai.google.dev/docs)
* [Google AI Studio](https://aistudio.google.com/)
* [Tavily Web Search](https://docs.tavily.com/)
* [Kestra GitHub](https://github.com/kestra-io/kestra)