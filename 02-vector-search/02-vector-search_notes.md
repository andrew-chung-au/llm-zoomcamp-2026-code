# 01 intro (Vector Search)

## Inverted Index & Keyword Search

---

### 1. Core Concept: What is an Inverted Index?
*   **Definition:** A data structure that maps **content** (words/tokens) to its **locations** (documents or database rows). 
*   **Analogy:** It functions exactly like the index at the back of a textbook. Instead of scanning the entire book page-by-page to find a topic, you look up the word to find the exact page numbers instantly.
*   **Purpose:** It eliminates the need for slow, linear sequential scans (O(N) time complexity) across millions of documents, enabling sub-second search retrieval.

---

### 2. Structural Mechanics: Forward vs. Inverted Index

#### Forward Index (Traditional Database)
Maps a **Document** to its **Tokens**.
*   **Doc 1:** `"The smart robot"`
*   **Doc 2:** `"The fast robot"`

#### Inverted Index (Search Engine Database)
Flips the mapping to group by **Token** to its **Document IDs**.
*   `"fast"` -> `[Doc 2]`
*   `"robot"` -> `[Doc 1, Doc 2]`
*   `"smart"` -> `[Doc 1]`
*   `"the"` -> `[Doc 1, Doc 2]`

> **Postings List:** The list of document IDs associated with a specific term (e.g., the postings list for `"robot"` is `[Doc 1, Doc 2]`).

---

### 3. Metadata for Ranking & Scoring
In real-world applications (like Elasticsearch or Lucene), a basic list of Document IDs isn't enough. The postings list in an inverted index also stores metadata used by ranking algorithms:
*   **Term Frequency (TF):** How many times the token appears within that specific document.
*   **Positions:** The exact character or word offset (crucial for exact phrase matching and proximity searches).

---

### 4. Interaction with Search Algorithms
While the **inverted index** retrieves *which* documents match a query, scoring algorithms determine *how relevant* those documents are to rank them.

#### A. TF-IDF (Term Frequency-Inverse Document Frequency)
Evaluates term importance by balancing local abundance against global rarity.
*   **TF (Term Frequency):** High frequency in a single document = high relevance.
*   **IDF (Inverse Document Frequency):** High frequency across the entire database (e.g., "the", "and") = low relevance weight.

Formula: TF-IDF = TF(t, d) * log(N / DF(t))
*(Where N is total documents and DF(t) is the number of documents containing the term t.)*

#### B. BM25 (Best Matching 25)
The industry standard evolution of TF-IDF (used by default in modern keyword search engines). It optimizes the score using two adjustments:
1.  **Term Frequency Saturation:** Limits the impact of repetitive words. Mentioning a keyword 20 times doesn't make a document 20 times more relevant than mentioning it twice.
2.  **Document Length Normalization:** Penalizes overly long documents. A keyword match in a short headline is weighted heavier than the same keyword buried deep in a 100-page manual.

### Vector Embeddings & Distance Metrics Notes

---

### 1. Core Concept: What are Embeddings and Distance?
* **Embeddings:** Neural networks convert text into numerical arrays called **vectors**. The model is trained so that texts with similar meanings land close to each other in this multidimensional space.
* **Distance Metrics:** Mathematical formulas used to calculate exactly *how close* or *far apart* two vectors are, which determines their semantic similarity.

---

#### 2. Cosine Similarity (The Industry Standard for Text)

The `all-MiniLM-L6-v2` model produces embeddings that are typically treated as normalized vectors, meaning each vector has unit length. When vectors are normalized, the dot product and cosine similarity are equivalent, which is why the model documentation refers to cosine similarity [web:351][web:356].

Cosine similarity measures the angle between two vectors, ignoring their length. It is usually interpreted like this:

- `1.0` = same direction, so the texts are very similar.
- `0.0` = perpendicular, so the texts are unrelated.
- `-1.0` = opposite direction, so the texts are maximally dissimilar.

Formally, cosine similarity is the cosine of the angle between two vectors. That means `cos(0) = 1`, `cos(90) = 0`, and `cos(180) = -1` [web:351][web:368].

Because these embeddings are normalized, we can use the dot product directly to compare texts. That is why `v1.dot(dv)` works as a similarity score for our examples [web:351][web:376].

In practice, negative cosine scores are uncommon for text embeddings. These models usually map text into a region of vector space where most comparisons are near zero or above, so very negative values are rare [web:368][web:376].

#### Why this matters

This is the basic idea behind vector search: texts with similar meaning end up with similar vectors, and the dot product gives us a fast way to compare them [web:351][web:377].

---

#### 3. Dot Product (Inner Product)
The raw mathematical engine behind cosine similarity. Instead of just looking at the angle, it combines both the **angle** and the **magnitude** (length) of the vectors.
* **How it works:** Multiplies the corresponding dimensions of two vectors together and sums the results.
* **The Normalization Trick:** If you "normalize" your embeddings (scale them so every vector has an exact length of 1), **Dot Product and Cosine Similarity become mathematically identical**. 
* **Use Case:** Highly preferred by vector databases (like Pinecone, Qdrant, Milvus) for normalized vectors because hardware computes the Dot Product significantly faster than Cosine Similarity.

---

#### 4. Euclidean Distance (L2 Distance)
Measures the physical, straight-line distance between the endpoints of two vectors (similar to the Pythagorean theorem).
* **How it works:** Calculates the spatial gap between Point A and Point B.
* **Use Case:** Rarely used for text. It is highly useful when the *magnitude* of the vector actually matters to the meaning—for example, in recommendation systems where magnitude might represent "how strongly" a user likes a specific movie or genre.

---

#### 5. Manhattan Distance (L1 Distance)
Measures distance as if navigating a city grid; you cannot travel diagonally in a straight line, but must move step-by-step strictly along the axes.
* **How it works:** Sums the absolute differences across all dimensions.
* **Use Case:** Useful in incredibly high-dimensional, sparse datasets. It is less sensitive to extreme outliers than Euclidean distance, making it mathematically robust against the "curse of dimensionality" (a phenomenon where traditional spatial distances start to lose meaning in massive mathematical spaces).

# 02 Embeddings

### 1. Choosing a model

Sentence-transformers supports many models, and the right choice depends on your task, your language, and the resources available. Larger models are usually slower, so for our FAQ dataset of short English texts, a small model is enough. In practice, it’s worth trying a few models on your own data and keeping the one that works best.

We’ll use `all-MiniLM-L6-v2` because it offers:

- 384-dimensional vectors, which are compact.
- Fast performance on CPU.
- Good quality for general English text.
- Cosine similarity, which we’ll explain below.

# 03 Embedding Our Dataset

### 1. `model.encode(q1)` shape

`model.encode(q1)` returns a 1D NumPy array with shape `(384,)`. The trailing comma is just tuple syntax for a one-element shape, not an error.

# 04 Vector Search

### 1. Floating-point comparisons

`np.allclose(scores, scores_loop)` can return `False` because of tiny floating-point differences, even when the matrix version and the Python loop are doing effectively the same math. Using `atol=1e-5` makes the comparison more tolerant.

### 2. Best match selection

`np.argmax(scores)` gives the index of the best match, while `scores.max()` gives only the score value. You need `argmax` when you want to retrieve the actual document.

### 3. Sorting the scores

`np.argsort(scores)` returns indices in ascending order using a temporary array for sorting. The reversed slice or `np.argsort(-scores)` is just a sorting trick on the temporary array; it does not change the original scores.

### 4. Display formatting

`:.3f` in `print(f"Best match score: {scores[idx]:.3f}")` is only display formatting. It rounds the score for readability without changing the underlying value.

### 5. Why Return Top 5 (Not Just Best Match)?

Key reasons for returning multiple results:
1. Answers may span multiple documents (part in one, rest in another)
2. Top result might not be perfect; second/third could be better
3. Send all 5 to LLM to combine and synthesize

Number 5 is a starting point (gut feeling). Later evaluation
can test whether 3 or 10 works better for this dataset.

For larger datasets, use a vector search library (filtering + ranking).

# 05 Vector Search with minsearch

### minsearch Vector Retrieval

---

#### 1. Core Concept: In-Memory Vector Search
* **Definition:** `minsearch` provides a simple in-memory vector search index for comparing embeddings.
* **Purpose:** It is useful for experiments, notebooks, and quick prototypes where you want a lightweight vector search setup.
* **Tradeoff:** It is easy to use, but it does not persist the index to disk.

---

#### 2. Building the Index
`VectorSearch` stores the embedding matrix and the document payload together.

```python
from minsearch import VectorSearch

vindex = VectorSearch(keyword_fields=["course"])
vindex.fit(X, documents)
```

* `X` is the NumPy matrix of document embeddings.
* `documents` is the list of FAQ dictionaries.
* `keyword_fields=["course"]` keeps the course filter available during search.

---

#### 3. Querying the Index
Before searching, the user query must be embedded into the same vector space as the documents.

```python
queryvector = model.encode(query)
results = vindex.search(queryvector, num_results=5)
```

* The query is encoded first.
* `vindex.search(...)` compares the query vector against document vectors.
* The search returns the most similar documents by score.

---

#### 4. Filtering by Course
Vector search still supports filtering, so search can stay restricted to one course.

```python
results = vindex.search(
    queryvector,
    filter_dict={"course": "llm-zoomcamp"},
    num_results=5,
)
```

* Filtering happens before ranking.
* This prevents unrelated courses from appearing in the results.
* The final ranking still uses vector similarity.

---

#### 5. Exact Nearest Neighbor Search
`minsearch` performs exact nearest-neighbor search for the whole dataset.

* Every document vector is compared with the query vector.
* The best matches are selected after scoring all candidates.
* This is fine for small datasets, but it becomes expensive as data grows.

---

#### 6. Why This Matters
This lesson shows the bridge from NumPy vector search to a reusable search library.

* NumPy demonstrates the math.
* `minsearch` packages that math into a search interface.
* The same retrieval pattern can later be swapped into RAG.

---

# 06 RAG with Vector Search

### Swapping Retrieval in RAG

---

#### 1. Core Idea
The RAG pipeline stays modular, so only the retrieval step changes.

* `search` changes from keyword search to vector search.
* `build_prompt` stays the same.
* `llm` stays the same.

---

#### 2. Using `RAGBase`
`RAGBase` already contains the main RAG flow.

```python
assistant = RAGBase(index=index, llm_client=openai_client)
```

* The base class handles the reusable RAG steps.
* The child class only needs to override retrieval.
* This keeps the code clean and easier to extend.

---

#### 3. Why `RAGVector` Needs an Embedder
Vector search needs the query embedded before it can search.

```python
class RAGVector(RAGBase):
    def __init__(self, embedder, **kwargs):
        self.embedder = embedder
        super().__init__(**kwargs)
```

* `embedder` turns the user question into a vector.
* `**kwargs` forwards the remaining setup values to `RAGBase`.
* This means the parent class still initializes the shared RAG pieces.

---

#### 4. Overriding Search
The subclass changes only the retrieval method.

```python
def search(self, query, num_results=5):
    queryvector = self.embedder.encode(query)
    filter_dict = {"course": self.course}
    return self.index.search(
        queryvector,
        num_results=num_results,
        filter_dict=filter_dict,
    )
```

* The query is embedded first.
* The course filter is still applied.
* The vector index searches using the query embedding.

---

#### 5. Using the Vector Assistant
Once the subclass is built, usage stays simple.

```python
vector_response = vector_assistant.rag("the program has already begun, can I still sign up?")
print("Vector RAG answer:")
print(vector_response)
```

* The assistant now uses vector retrieval instead of keyword retrieval.
* Rephrased questions can still match the right FAQ entry.
* Only the search step changed.

---

#### 6. Why This Is Useful
This lesson shows that RAG can swap retrieval methods without rewriting everything.

* The same structure supports keyword search, vector search, and later even hybrid search.
* The query flow becomes more robust to paraphrases.
* The design makes future changes easier, including swapping the model provider later.

---

#### 7. Key Comparison
* `RAGBase`: sends the raw query into the existing search flow.
* `RAGVector`: embeds the query first, then searches by vector similarity.
* Both keep the rest of the RAG pipeline unchanged.

---

#### 8. Practical Takeaway
Vector search is more flexible than keyword search, but it adds overhead.

* You need an embedding model.
* You need to encode the query before search.
* You need to store and manage vectors.
* The gain is better matching for paraphrased questions.

# 07 Vector Search with sqlitesearch

### Persistence and ANN Search

---

#### 1. Why `sqlitesearch` Exists
`minsearch` is fine for experiments, but it has two practical limits:
* the index is rebuilt every startup.
* everything stays in memory.

`sqlitesearch` keeps the index on disk, so you can build it once and reuse it later.

---

#### 2. Why Vector Search Is More Expensive
Vector search has an extra cost that text search does not:
* every document must be embedded first,
* the embeddings must be stored,
* and the query must still be compared against many vectors.

So the expensive part is not just search, but the whole indexing pipeline.

---

#### 3. The Query Does Not Change the Dataset
The document embeddings stay fixed in the vector space.
When a query arrives, it is embedded into the same space and compared against those fixed document vectors.

So the query does not “reframe” the dataset.
It simply enters the same space and searches from there.

---

#### 4. Why a Distance Cutoff Is Not Enough
It is tempting to think ANN is just “only check vectors within some radius.”
But that would still require checking everything first to know which ones are inside the radius.

ANN works because the index is organized ahead of time.
That structure lets the system skip most comparisons at query time.

---

#### 5. What ANN Is Actually Doing
ANN does not change similarity itself.
It reduces the number of candidate vectors that need to be scored.

* Exact NN scores every document.
* ANN first narrows the search to a promising region.
* Then it runs exact scoring only inside that smaller set.

That is why ANN is faster, but slightly approximate.

---

#### 6. Common ANN Structures
ANN systems usually build one of these kinds of structure during indexing:

##### Clustering / IVF (Inverted File Index)
* Group vectors into clusters.
* Compare the query to cluster centers first.
* Search the most promising cluster or clusters.

##### Trees (Annoy - Approximate Nearest Neighbors Oh Yeah)
* Split the space into branches.
* Follow the most promising path.
* Search only the resulting bucket.

##### Graphs / HNSW (Hierarchical Navigable Small World)
* Link each vector to nearby vectors.
* Walk from one nearby node to the next at query time.
* Stop when the search reaches a local best match.

My “nearest-neighbor-of-neighbors” intuition was basically the graph idea.

`sqlitesearch` supports several ANN modes:
* `lsh` is the default and uses random hyperplane projections.
* `ivf` uses K-means clustering.
* `hnsw` uses a proximity graph and gives the highest recall.

All modes use two-phase search: approximate candidate retrieval first, then exact cosine similarity reranking.

---

#### 7. Why This Matters for `sqlitesearch`
`sqlitesearch` is the persistent, on-disk version of `minsearch`: it stores vector indexes in SQLite, supports ANN retrieval, and lets one process build the index while another can query it later.

That makes the setup more realistic:
* the index survives restarts,
* the search is not brute-force only,
* and the whole pipeline is closer to production use.

`uv add sqlitesearch`

---

#### 8. `minsearch` vs `sqlitesearch`
* `minsearch` is in-memory, uses NumPy, and is good for experiments and notebooks.
* `sqlitesearch` is persistent, stores its index in a SQLite `.db` file, and is better for projects and persistence.
* `minsearch` uses exact cosine similarity over all vectors.
* `sqlitesearch` uses ANN modes like `lsh`, `ivf`, and `hnsw`, with exact reranking after candidate retrieval.
* `sqlitesearch` is lightweight because it only depends on SQLite and NumPy.

---

#### 9. Why `sqlitesearch` Is Mostly a Teaching Tool
* It was built to show the ingestion-then-deployment split clearly.
* For most real projects, you would eventually move to a more fully featured vector database.
* It can still run anywhere SQLite is available, which makes it convenient when you want persistence without the overhead of a dedicated vector database.

---

# 08 Vector Search with PGVector

---

### 1. Core Concepts: Traditional vs. Vector Search

Traditional SQL Search: Relies on exact matches or text patterns (e.g., `WHERE title LIKE '%Data%'`). It cannot understand meaning.

Vector Search: Searches by semantic meaning. Machine learning models convert text, images, or audio into long lists of numbers called embeddings (or vectors).

The Goal: High-dimensional vectors that are close to each other numerically share similar concepts (e.g., "king" and "queen" or "cat" and "kitten" sit near each other in vector space).

Why pgvector over Dedicated Vector DBs?

While dedicated vector stores (like Chroma or Qdrant) exist, pgvector allows you to keep your structured application data (users, orders) and your AI embeddings in the exact same Postgres database. This eliminates the architectural complexity of managing, syncing, and backing up two separate databases.

### 2. Docker Command Breakdown

The command spins up a production-ready PostgreSQL instance with pgvector pre-installed from Docker Hub (`hub.docker.com/r/pgvector/pgvector`).

```bash
docker run -it \
    --name pgvector \
    -e POSTGRES_USER=user \
    -e POSTGRES_PASSWORD=pswd \
    -e POSTGRES_DB=faq \
    -v pgvector_data:/var/lib/postgresql/data \
    -p 5432:5432 \
    pgvector/pgvector:pg17
```

| Parameter / Flag | Purpose |
|---|---|
| `docker run -it` | Runs the container interactively and keeps the terminal process open so you can see live database logs. |
| `--name pgvector` | Assigns a friendly local name (pgvector) so you can easily stop/start it via `docker stop pgvector`. |
| `-e POSTGRES_USER=user` | Environment variable setting the database superuser name. |
| `-e POSTGRES_PASSWORD=pswd` | Environment variable setting the superuser password. |
| `-e POSTGRES_DB=faq` | Automatically creates a default database named faq upon startup. |
| `-v pgvector_data:...` | Data Persistence: Maps a Docker volume to the container's internal data directory. This ensures your data isn't deleted when the container stops. |
| `-p 5432:5432` | Forwards port 5432 from your host machine to the container, letting external Python apps connect to `localhost:5432`. |
| `pgvector/pgvector:pg17` | The image address. pgvector/pgvector is the official organization/repo on Docker Hub; `:pg17` specifies PostgreSQL version 17. |

### 3. Python Environment & Connection (psycopg v3)

Package Manager (uv): The command `uv add psycopg[binary]` uses uv, a modern, high-speed replacement for pip.

The Driver (psycopg v3): This is a complete rewrite of the older psycopg2 driver.

The `[binary]` tag: Installs pre-compiled C extensions so you don't need local database development tools installed on your operating system to compile the driver.

Key Improvement (Direct Connection Execution): You can run `conn.execute()` directly on the connection object. You no longer need to manually spin up and manage a separate cursor object (`cur = conn.cursor()`) for simple queries.

#### 💡 Deep Dive: What is a Cursor & Why This Matters
To understand this improvement, you have to look at how database drivers traditionally handle data:

What is a Cursor? A cursor is a control structure used to navigate, manipulate, and fetch the rows of a database result set.

The Analogy: If the database is a massive warehouse and your SQL query is the order form, the cursor is a forklift operator. It goes into the warehouse, points to the specific data, and brings it back to your Python app line by line so your computer's memory doesn't get overwhelmed.

The Traditional Boilerplate (psycopg2): Historically, you couldn't run queries directly on a connection. Even for a simple command like creating a table, you had to write extra lines of code just to manage the "forklift":

```python
conn = psycopg2.connect(...)
cur = conn.cursor()          # 1. Create cursor manually
cur.execute("SELECT ...")    # 2. Run query
data = cur.fetchall()        # 3. Fetch data
cur.close()                  # 4. Clean up cursor manually
```

How psycopg v3 Fixed This: When you call `conn.execute()`, the driver handles the boring stuff for you. Under the hood, it still creates a temporary cursor, executes your SQL, and cleans it up automatically. Your code becomes shorter and cleaner without sacrificing safety.

When do you still need a manual cursor? > Only when dealing with massive datasets. If you are querying millions of vector rows and want to stream them in small batches (e.g., fetching 100 rows at a time using `.fetchmany()`) to prevent your computer's memory from crashing, you will still manually spin up a cursor. For creating tables, inserting rows, or running simple vector searches, `conn.execute()` is all you need.

### 4. Implementation SQL Operators

Once connected, pgvector introduces new mathematical operators to calculate distance in SQL queries:

- `<=>` : Cosine Distance (Most common for text/LLM embeddings; measures the angle between vectors).
- `<->` : Euclidean Distance (Measures straight-line distance between points).
- `<#>` : Negative Inner Product (Used for dot-product similarity).

Note on Querying: When sorting by distance (e.g., `ORDER BY embedding <=> %s LIMIT 1`), smaller numbers mean the vectors are closer together, meaning the results are more semantically similar.

---

## 5. Why the embedding string exists

The embedding is generated in Python as a numeric vector, but Postgres expects a value it can parse into its `vector` column type. That is why the vector is converted into a string like `"[0.1,0.2,...]"` before insert or query time.

The string is not the real embedding format for storage in your notebook logic; it is just the transport format passed into SQL so pgvector can cast it back into a vector.

---

## 6. Why the insert loop is in Python, not SQL

SQL is good at set-based operations, but the embedding generation happens in Python because the model lives there. So the workflow is split:

- Python loads documents.
- Python creates embeddings.
- Python loops over document/vector pairs.
- Postgres stores the final rows.

That separation is useful to remember because it explains why the ingestion script is not “just SQL”.

---

## 7. Why `zip()` appears in the insert loop

`zip(documents, vectors)` pairs each document with its matching embedding so the loop processes them in sync. It is not creating a database structure; it is just making sure the `n`th document is inserted with the `n`th vector.

This is a classic Python pattern for parallel iteration and is much safer than manually tracking indexes.

---

## 8. Why PGVector is the production-oriented step

PGVector is not just a vector store; it is vector search inside Postgres. That means your structured application data, metadata, transactions, and embeddings can all live in one system.

This is what makes it more production-oriented than `minsearch` or `sqlitesearch`:
- persistent storage inside a real database,
- concurrent reads and writes,
- transactional consistency,
- integration with existing Postgres applications,
- a clearer path to larger datasets.

So PGVector sits between notebook-friendly tools and dedicated vector databases. It is a practical next step when you want semantic search to behave more like part of a real application than a standalone experiment.

---

# 09 Using ONNX Runtime instead of PyTorch

### Using ONNX Runtime instead of PyTorch

For production, the main benefit is lower overhead. `sentence-transformers` pulls in PyTorch and many extra dependencies, while ONNX Runtime can serve the same embedding model with a much smaller deployment footprint.

A quick comparison from the lesson:
- `sentence-transformers`: 4.8 GB, 58 packages
- ONNX Runtime: 147 MB, 27 packages
- That is about 33x smaller for the same embeddings and the same results

For experiments and development, `sentence-transformers` is still fine. For production, ONNX Runtime is the lighter and cleaner option.

#### Project setup

Create a separate project (or child module folder) for the ONNX version so the dependencies stay isolated.

```bash
mkdir llm-zoomcamp-onnx && cd llm-zoomcamp-onnx
uv init --no-workspace
uv add onnxruntime tokenizers numpy tqdm minsearch
uv add --dev huggingface-hub jupyter
```

`huggingface-hub` is only needed to download the model. At runtime, the important packages are `onnxruntime`, `tokenizers`, and `numpy`.

Register a notebook kernel for the project so Jupyter uses this exact environment:

```bash
uv run python -m ipykernel install --user --name llm-zoomcamp-onnx --display-name "llm-zoomcamp-onnx"
```

#### Downloading the model

Copy the `download.py` script from the `embed/` folder to the onnx project root, and use it to fetch the ONNX model from Hugging Face.

```bash
uv run python download.py
```

This creates the local model files:

```text
models/
  Xenova/
    all-MiniLM-L6-v2/
      tokenizer.json
      model.onnx
```

You only need to do this once. After that, the model files are stored locally.

Add the model directory to `.gitignore`:

```text
models/
```

#### The Embedder class

Copy the Use the `embedder.py` script from the `embed/` folder to the onnx project root, and use it to generate embeddings.

Under the hood, it does four things:
- Tokenize: convert text into token IDs and attention masks
- Run the ONNX model: execute the model graph on CPU
- Mean pooling: average token embeddings using the attention mask
- Normalize: divide by the L2 norm so vectors can be compared with dot product

The key point is that it still gives the same `encode()` interface as before, but without the heavy PyTorch dependency stack.

#### Why this matters

The lesson highlights a practical deployment tradeoff:
- PyTorch-based embeddings are convenient for learning and prototyping
- ONNX Runtime is better when you want a smaller, lighter production setup

So the real idea is not just switching libraries. It is keeping the same embedding workflow while cutting runtime size and dependency overhead.

### Available models

The ONNX setup can use several different embedding models with the same basic code path. In practice, you change the model name in `download.py` and update the path passed to `Embedder()`.

The key difference between models is usually:
- **quality**,
- **speed**,
- **language coverage**,
- **embedding size**.

Typical tradeoff:
- smaller models are faster and lighter,
- larger models usually give better retrieval quality,
- multilingual models work across more languages,
- retrieval-focused models are often better for search than general sentence similarity models.

Examples:

- `Xenova/all-MiniLM-L6-v2` — 384 dimensions, good small general-purpose model.
- `Xenova/all-MiniLM-L12-v2` — 384 dimensions, a bit better quality, but slower.
- `Xenova/paraphrase-MiniLM-L6-v2` — 384 dimensions, good for paraphrase-style similarity.
- `Xenova/paraphrase-multilingual-MiniLM-L12-v2` — 384 dimensions, multilingual paraphrase use.
- `Xenova/multilingual-e5-small` — 384 dimensions, multilingual retrieval.
- `Xenova/multilingual-e5-base` — 768 dimensions, stronger multilingual retrieval.
- `Xenova/bge-small-en-v1.5` — 384 dimensions, strong retrieval performance.
- `Xenova/bge-base-en-v1.5` — 768 dimensions, stronger retrieval performance.
- `Xenova/gte-small` — 384 dimensions, lightweight modern model.
- `Xenova/gte-base` — 768 dimensions, stronger GTE model.

#### How to switch models

To use a different model:
1. Add the model name to `download.py`.
2. Run the download step again.
3. Update the model path in `Embedder()`.

Example:

```python
embed = Embedder("models/Xenova/bge-base-en-v1.5") # If we leave it as `embed = Embedder()` it uses the default argument in __init__:
vectors = embed.encode("your text here") # Check for embed.encode in the code
print(vectors.shape)
```

The `print(vectors.shape)` line is just a quick sanity check. It confirms the embedding size, which should match the model’s expected dimensionality.

#### Why the runtime stays light

Even though the model choice can change, the runtime stays minimal because it only depends on:
- `onnxruntime`,
- `tokenizers`,
- `numpy`.

That matters because it makes the setup easier to deploy in:
- small Docker images,
- serverless functions,
- edge devices.

In other words, the model files can be swapped, but the runtime stack stays lean.

# 10 Next Steps

The main takeaway is that vector search is useful, but not always the first choice. It adds extra overhead because documents and queries must be turned into embeddings before search can happen.

A simple way to think about the progression is:
- **Text search first**: fast, simple, and often enough.
- **Vector search next**: useful when users phrase the same idea in different words.
- **Hybrid search last**: combines both methods and often gives better results.

Key terms:
- **Embedding**: a numeric vector that represents meaning.
- **Vector search**: finds documents by comparing embeddings, not exact words.
- **Hybrid search**: combines text search and vector search.
- **Reranking**: a second pass that re-scores retrieved documents.

The best next step is to start with text search, then add vector search only when it clearly improves results.

# 11 Homework: Hybrid Search

Vector search and text search have different strengths. Vector search matches by meaning, so it helps when the query and the document use different words. Text search matches exact words, so it is better for names, codes, and rare terms. Hybrid search combines both methods.

However, text search and vector search produce different scores, and those scores are not directly comparable. Instead of merging the raw scores, the homework hybrid search merges the ranking lists using **Reciprocal Rank Fusion (RRF)**. RRF gives each document a small score based on its position in each ranked list, then adds those scores together. A document that appears in both lists gets a contribution from both searches, while a document that appears in only one list gets just one contribution.

The RRF formula is:

\[
RRF(d) = \sum \frac{1}{k + rank(d)}
\]

where:
- `rank(d)` is the position of the chunk in a ranked list, starting at 0.
- `k` is a constant that controls how much rank affects the final score. A smaller `k` makes top positions matter more, while a larger `k` makes the ranking flatter and rewards agreement across lists more evenly. The value 60 is the standard default from the original RRF paper, and it is commonly used because it gives a good balance without overemphasizing tiny rank differences.

A useful way to think about RRF is that it rewards agreement between the two searches. A chunk does not need to be first in either search on its own. If it ranks well in both, it can rise to the top after fusion.

Code:

```python
def rrf(result_lists, k=60, num_results=5):
    scores = {}
    docs = {}

    for results in result_lists:
        for rank, doc in enumerate(results):
            key = (doc["filename"], doc["start"])
            scores[key] = scores.get(key, 0) + 1 / (k + rank)
            docs[key] = doc

    ranked = sorted(scores, key=scores.get, reverse=True)
    return [docs[key] for key in ranked[:num_results]]
```

Example:

```python
query = "How do I give the model access to tools?"

# Get text search and vector search results/rankings
text_results = text_search(query, num_results=5)
vector_results = vector_search(query, num_results=5)

# Fuse results
results = rrf([vector_results, text_results], k=60, num_results=5)
```

The important detail is that `rrf()` does not compare raw scores. It only looks at ranks. That makes it easier to combine different search methods without worrying about score scale.

## What this shows

The main insight is that hybrid search can recover useful documents that one method alone might miss. If a chunk is a semantic match for the query and also contains an exact keyword, it is more likely to survive the fusion step.

In the homework example, the top fused result may not be the top result from text search or vector search separately. It wins because it ranks strongly in both lists, which is exactly what hybrid search is designed to reward.

Key terms:
- **Hybrid search**: combines text search and vector search.
- **Reciprocal Rank Fusion (RRF)**: merges ranked lists using rank-based scoring.
- **Rank**: the position of a result in a search list.
- **Fusion**: combining multiple ranked lists into one final ranking.