# 05 Search

### Core Search Concepts (Lucene/Elasticsearch/MinSearch)

*   **Text Fields:** For natural language (e.g., document bodies). Text is analyzed, lowercased, and broken into tokens for flexible, partial matching.
*   **Keyword Fields:** For structured strings (e.g., IDs, tags). Text is stored as a single literal block requiring an exact, case-sensitive match.
*   **Filtering:** Binary inclusion/exclusion (Yes/No). Used to instantly isolate documents (e.g., matching a specific course ID) without calculating a relevance score.
*   **Boosting:** Weight multipliers applied to specific fields during a query. For example, boosting a `title` field ensures that matches in the header score higher than matches buried in the page text.

# 07 The LLM

### Pydantic Objects

A Pydantic object is a structured Python object created from a schema. Instead of returning a raw string or an unstructured dictionary, the data is parsed into a model with defined fields and types.

#### What that means

When the course says “the response is a Pydantic object,” it means the LLM output has been turned into a predictable object that follows a blueprint. You can access its fields with dot notation, like `response.output_text` or `response.output[0].content[0].text`, depending on the API.

#### Why it matters

Pydantic validates the data against the schema you defined. If a field is missing or has the wrong type, it raises a `ValidationError` instead of letting broken data spread through your program.

#### What developers do with errors

If validation fails, you can:
- Ask the LLM to fix the output and try again.
- Ignore the bad record if the data is not critical.
- Use optional fields or defaults to make the schema more forgiving.

#### Why it is useful in AI apps

Pydantic makes LLM output easier to trust, easier to read, and easier to use in code. It is especially helpful when you need the model to return structured data instead of free-form text.

### Pydantic Validation and LLM Self-Repair

Pydantic helps turn messy LLM output into structured Python objects. If the output does not match the expected schema, it raises a `ValidationError` instead of letting bad data flow through the app.

#### What happens on mismatch

If the LLM forgets a required field or returns the wrong type, Pydantic fails validation immediately. That protects the rest of your code from crashes or silent bugs.

#### Two common responses to errors

##### 1. Reask the LLM
You can catch the validation error and send the error message back to the model so it can try again with corrected output. This is often called self-repair or reasking.

##### 2. Ignore or relax the rule
If the missing data is not critical, you can make fields optional, give them defaults, or drop the bad record and keep the rest. This avoids extra latency and token cost.

#### Why this is useful

Pydantic makes LLM outputs predictable, typed, and easier to work with. In practice, it acts like a schema checker at the boundary between the model and your application.

### Exploring the Response

The API response is a structured Pydantic object, not just a plain text answer. The final answer is only one field inside a larger response object.

#### Where the answer is

- `response.output` is a list of output items.
- `response.output[0]` is the first message object.
- `response.output[0].content[0].text` is the actual text answer.
- `response.output_text` is a shortcut that returns the final answer directly.

#### What else the response can contain

The response object can also include:
- nested message/content structure
- token usage counts
- model name
- IDs and status fields
- tool-call or intermediate output details

#### Why this matters

The extra fields are useful for:
- **cost tracking**, using token counts
- **monitoring**, such as latency, failures, or abnormal responses
- **debugging**, by inspecting response structure and metadata
- **evaluation**, by logging more than just the final answer

#### Simple way to think about it

The full response is a container with the answer plus extra structured information. `response.output_text` gives you just the final text, while the full object gives you additional details when you need them.

### Advanced Cost Optimization: Prompt Caching

When identical context prefixes (such as an embedded FAQ knowledge base or a long system instruction) are sent repeatedly across sequential API requests, the gateway reads them from cache rather than re-tokenizing them.

#### Key Economics:
* **Performance Gains:** Drastically reduces Time-to-First-Token (TTFT) latency.
* **Cost Reductions:** Cached tokens typically receive a **50% discount** compared to standard input tokens.
* **Tracking Attribute:** Look inside `response.usage.prompt_tokens_details.cached_tokens` to extract the exact volume of cached text processed.

### API Token Usage: OpenAI vs Gemini

Both OpenAI and Gemini report the same core token metrics: input tokens, output tokens, and total tokens. The main difference is the field names used in each SDK response.

#### Example: OpenAI usage output

This was the OpenAI-style usage object returned in the response:

```python
CompletionUsage(
    completion_tokens=20,
    prompt_tokens=8,
    total_tokens=28,
    completion_tokens_details=None,
    prompt_tokens_details=None
)
```

#### Field mapping

| Token metric | OpenAI-style field | Native Gemini field |
|---|---|---|
| Input tokens | `prompt_tokens` | `prompt_token_count` |
| Output tokens | `completion_tokens` | `candidates_token_count` |
| Total tokens | `total_tokens` | `total_token_count` |

#### Important difference

If you call Gemini through Google’s **OpenAI-compatible endpoint**, the response uses the **OpenAI-style** `usage` object and field names like `prompt_tokens` and `completion_tokens` [web:933][web:917]. If you use Gemini through the **native Gemini SDK**, the response uses `usage_metadata` with fields like `prompt_token_count` and `candidates_token_count` [web:936][web:940].

#### Notes

`completion_tokens_details` and `prompt_tokens_details` may be `None`, but those fields exist to hold more detailed token breakdowns when available [web:944]. In native Gemini responses, extra fields can include things like `thoughts_token_count` and cached-content counts, depending on the SDK and model [web:936][web:940].

# 08 RAG Helper

### Why `rag_helper.py` is a class

`rag_helper.py` is written as a class because the RAG pipeline depends on shared state, especially the search index and the LLM client. In the notebook version, these existed as global variables, but once the code is moved into a separate file, relying on globals makes it harder to reuse and harder to adapt.

### Why a class helps

A class lets these dependencies be passed in through the constructor and stored on the object. That means the same RAG logic can be reused with different search backends, different LLM clients, different models, or different prompt templates without changing the rest of the code.

### Why `ingest.py` stays as functions

`ingest.py` mainly contains utility functions like loading FAQ data and building an index. These tasks do not need to keep shared state across multiple method calls, so plain functions are simpler and more appropriate.

### Key distinction

The difference is not about whether the code is used from different files, because both files can be imported elsewhere. The real difference is that `rag_helper.py` needs to carry configuration and shared dependencies, while `ingest.py` mostly just takes input, performs a task, and returns output.

# 09 Data Ingestion

### What is `sqlitesearch`

`sqlitesearch` is a wrapper for SQLite to make its lower-level search features easier to use. Instead of working with SQLite directly, the library gives you a simpler, higher-level API for indexing and searching documents.

### Why it is useful

The main benefit is convenience. It hides a lot of the SQLite-specific setup and lets you work with a search interface that feels closer to `minsearch`, so it is easier to plug into the rest of the course code.

### What it adds

`sqlitesearch` provides a more practical interface for:
- adding documents to an index,
- searching text with a simple API,
- using a persistent search backend without changing the rest of the RAG pipeline,
- keeping the ingestion and query code clean and consistent.

### More information
https://alexeyondata.substack.com/p/how-i-built-sqlitesearch-a-lightweight

### Note on the ingestion demo

In the lesson, documents are added one by one with preview prints and a short time delay for teaching purposes. This makes the ingestion process easier to observe and helps show that one process can write to the index while another process reads from it.

### What would happen in normal practice

In a real ingestion job, the time delay would be removed because the goal is to load data as quickly as possible. You also would not usually print every document preview unless you were debugging or checking that the data looked correct.

### Typical real-world approach

A more normal approach is to ingest documents as fast as possible and print only occasional progress updates, such as every 100 or 1,000 records. This keeps the process fast while still giving enough visibility to confirm that ingestion is working.

```Python
for i, doc in enumerate(docs_llm, start=1):
    index.add(doc)
    if i % 100 == 0:
        print(f"Added {i} documents...")
```

### Modular design and compatibility

The pipeline is modular, so the RAG logic stays the same while the search backend can be swapped out. That works here because `sqlitesearch` follows the same API as `minsearch`, including the same `search` method shape.

This means some experiments are easy to try without changing `RAGBase`, especially:
- changing chunk sizes,
- changing overlap ratios,
- swapping the LLM client,
- testing retrieval quality by inspecting returned documents before generation.

But some experiment options are more likely to require code changes:

- **Keyword search (`minsearch`, `sqlite` full-text)** is usually easy to swap if the API stays the same.
- **Vector search** may require embeddings, a different index type, and a different result format.
- **Hybrid search** may require combining two retrieval pipelines and merging results.
- **Switching to Elasticsearch, OpenSearch, or Qdrant** may require a wrapper or a `RAGBase` subclass if the search method or filters do not match what the pipeline expects.

### Why this matters for experimentation

The main retrieval experiments are usually about improving recall and latency, not rewriting the whole pipeline. If the backend stays API-compatible, I can test retrieval quality independently by checking the documents returned before passing them to the LLM. If the correct answer is not in the top results, changing the prompt will not help.

So the rule is: keep the modular structure, but expect integration work if the retrieval backend changes its interface.

### Choosing an approach

Pick the right tool for your data:

- `minsearch`: single process, in-memory only, re-indexes on every startup. Use when the dataset is small and indexing is fast.
- `sqlitesearch`: separate ingestion and query, file-based SQLite, opens an existing index. Use when ingestion is slow or when you need the index to survive restarts.

Use `minsearch` when you can load and index the data on startup without noticeable delay. Switch to a persistent backend when ingestion takes too long or when you need the index to survive restarts.

For larger production systems, use the same pattern with a different backend:
- Elasticsearch
- OpenSearch
- Qdrant
- Weaviate

The architecture stays the same: one process ingests, another queries.

# 10 Wrap-up of Part 1 (RAG next steps)

### Where to go next

- **Agents:** Let the LLM control the search loop so it can recover from bad retrieval, run multiple searches, and adapt the query when needed.
- **Vector search:** Helps when users ask the same thing in different words by matching meaning rather than exact keywords.
- **Elasticsearch / OpenSearch:** Better for larger systems because they scale well and support more advanced search features.
- **Fine-tuning:** Possible, but less flexible than RAG and harder to update when data changes. Not covered in this course.

# 13 Function Calling

## Inspecting JSON

When an API response is a structured object, `.model_dump()` can convert it into a plain Python dictionary. This makes the nested response easier to inspect in a notebook.

### Display as formatted JSON

```python
from IPython.display import JSON

# Turn the structured response object into a plain dictionary,
# then display it as readable JSON in the notebook.
JSON(answer.model_dump())
```

### Print as indented JSON text

```python
import json

# Convert the response object to a dictionary and print it with indentation
# so the nested structure is easier to read.
print(json.dumps(answer.model_dump(), indent=2))
```

### Tip

- Use `JSON(...)` for a cleaner notebook view.
- Use `json.dumps(..., indent=2)` when plain printed text is enough.
- `.model_dump()` is useful when the API response is a structured model object rather than a regular dictionary.

# 14 The Agentic Loop

Writing the agent loop by hand makes the underlying process visible before any framework hides it. It shows when the model decides a tool is needed, when the tool runs, how the result is added back into the conversation, and how the next model call continues from that updated state.

Frameworks like LangChain, PydanticAI, and the OpenAI Agents SDK use the same core pattern. They reduce boilerplate, but they also hide the step-by-step flow. A handwritten loop is easier to debug and makes repeated tool calls much easier to understand.

### Core pattern

The agent loop follows the same basic cycle each time:

1. Send the current conversation to the model.
2. Check whether the model requested a tool.
3. Run the tool if needed.
4. Append the tool result to the conversation history.
5. Call the model again.
6. Stop when the model returns a final answer without more tool calls.

### Why this matters

Understanding the manual loop makes it easier to reason about:
- why conversation history must be preserved,
- how tool calls are linked to tool results,
- why multiple iterations may be needed,
- and what frameworks are doing behind the scenes.

# 15 ToyAIKit

### Registering a tool

In this context, **registering** a tool means adding a function to the framework's list of available tools so the agent can use it later. Once a tool is registered, the framework knows that the tool exists and can expose it to the model as something it is allowed to call.

A useful way to think about it is:

- **Writing the function** creates the tool.
- **Registering the function** makes the tool available to the agent.

For example:

```python
agent_tools = Tools()
agent_tools.add_tool(search, search_tool)
```

This tells the framework to include `search` in the agent's tool set, along with the schema that describes how the tool should be called.

#### Mental model

Registering a tool is a bit like adding a book to a library catalog or adding a command to a menu. The tool already exists as a Python function, but registration is what makes it visible and usable inside the agent system.

#### What registration may do

Depending on the framework, registering a tool may also:

- store the function in an internal tool registry,
- attach its schema or generate one automatically,
- inspect the function signature and docstring,
- and do some light validation to make sure the tool can be exposed properly.

This is different from full runtime validation. Registration mainly prepares the tool for use; it does not guarantee that every future tool call will be correct.

#### Scope of registration

Registering a tool usually makes it available only within the current project, application, or agent setup. It does **not** normally make the tool available to every other person who downloads the framework.

For other people to use that tool, it would need to be shared as part of the codebase, package, plugin, or extension that they install. So registration is usually local to your implementation, not global to all users of the framework.

#### Why it matters

Without registration, the function exists only in Python code. The agent framework would not know that the tool is available, and the model would not be able to call it as part of the agent loop.

# 16 Other Frameworks

### From ToyAIKit to other agent frameworks

Lesson 15 showed the core pattern behind an agent: define tools, pass messages into a loop, let the model decide whether to call a function, and repeat until the response is complete. That same pattern shows up in many other frameworks, even if the APIs look different.

The important idea is that the framework is mostly a wrapper around the same loop:

- send the messages,
- run any tool calls the model requests,
- add the tool output back into the conversation,
- repeat until the model is done.

Once this loop is understood, other frameworks become much easier to read. The task is no longer to memorize a new magic system — it is to recognize the same basic pattern in a different wrapper.

### Frameworks worth knowing

#### OpenAI Agents SDK

This is the official agent SDK from OpenAI. It is a good fit if OpenAI is already part of the stack and a maintained official framework is preferred. It supports tools, multi-turn conversations, and agent handoffs.

#### PydanticAI

This is a type-safe framework that works with multiple model providers. Tools are plain Python functions with type hints, which makes it feel natural for anyone already comfortable with Python type checking. It is a strong choice for clarity, structure, and multi-provider support.

#### LangChain and LangGraph

LangChain is one of the most widely used agent and LLM frameworks, with many integrations. LangGraph adds graph-based workflows for more complex agent behavior. It is useful when lots of connectors, document loaders, or vector store integrations are needed.

#### Google ADK

Google’s Agent Development Kit is a good match for Gemini models. It exposes familiar building blocks like tools, instructions, and sessions, and it fits naturally into the Google Cloud ecosystem.

#### Other frameworks

A few other names are worth recognizing:

- CrewAI for multi-agent orchestration.
- AutoGen for multi-agent conversations.
- Semantic Kernel for C# and Python.
- Smolagents for a lightweight approach.
- Anthropic’s tool use APIs for native Anthropic workflows.

The specific framework matters less than the quality of the tools and prompts. The loop is usually the same even when the packaging changes.

### Picking a framework

The best framework is usually the one that fits the stack and the constraints. If OpenAI’s ecosystem is already in use, the official SDK is a natural starting point. If type safety and flexibility across providers are the priority, PydanticAI is appealing. If lots of integrations are needed, LangChain or LangGraph may be the better fit. If the stack is already on Google Cloud or Gemini, Google ADK is worth a close look.

### When not to use an agent

It is tempting to reach for an agent whenever an LLM is involved, but that is not always the best choice. Agents add cost, latency, more moving parts, and less predictable behavior because the model can take different paths on different runs.

A simpler approach is often enough:

- one RAG search and one answer,
- one LLM call with no tools,
- parsing or transforming a document into another form.

The best rule is to start simple. If a plain prompt, a single model call, or a basic retrieval flow solves the problem, ship that first. Use an agent only when the simpler approach breaks down and the extra loop is genuinely needed.

### Final takeaway

The real lesson is not tied to one framework. It is the loop itself: tools, messages, execution, and repetition. Once that clicks, moving between frameworks becomes much easier because the same pattern is visible under the hood.