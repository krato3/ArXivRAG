# ArXivRAG
Docs for project understanding
# HELP.md — Research Paper RAG Project

## 1) What this project does

This project builds a research-paper retrieval-augmented generation (RAG) system.

The workflow is:

1. Download arXiv research papers
2. Parse PDF files into page-level text
3. Clean the extracted text
4. Split papers into section-aware chunks
5. Embed chunks using a sentence-transformer model
6. Store dense vectors in Qdrant
7. Store BM25 text indexes in OpenSearch
8. Retrieve relevant chunks using hybrid search
9. Rerank results with a cross-encoder
10. Route the query and generate an answer using Ollama + LangGraph
11. Expose the workflow through a FastAPI API

This is not a toy demo. It is closer to a production-style document RAG pipeline with evaluation, retrieval tuning, and API serving.

---

## 2) High-level architecture

The project has 4 major layers:

- Ingestion layer: downloads and prepares paper data
- Retrieval layer: Qdrant + OpenSearch + hybrid retrieval
- RAG orchestration layer: LangGraph nodes and state
- API + evaluation layer: REST endpoint + quality checks

Flow:

Research paper PDFs
   ↓
PDF parsing
   ↓
Text cleaning
   ↓
Section-based chunking
   ↓
Embedding generation
   ↓
Qdrant + OpenSearch indexing
   ↓
Hybrid retrieval (dense + BM25)
   ↓
Reranking
   ↓
Query routing / rewrite / intent classification
   ↓
LLM answer generation
   ↓
FastAPI response

---

## 3) Root-level files

### main.py
This is a tiny entry script, not the actual app logic.

It prints “Hello from research-rag!” and is mainly a placeholder.

### pyproject.toml
Defines the Python project and dependencies.

Important packages include:
- FastAPI
- LangGraph
- LangChain
- Qdrant client
- OpenSearch Python client
- sentence-transformers
- PyMuPDF
- arxiv
- pytest

This tells you the stack is:
- Python
- FastAPI
- LangChain/LangGraph
- local LLM (Ollama)
- vector DB (Qdrant)
- keyword search (OpenSearch)
- Hugging Face embedding models

### docker-compose.yml
Sets up the infrastructure for Qdrant.

The project expects a Qdrant service at localhost:6333.

### README.md
This is the main project overview and explains the architecture, evaluation strategy, and retrieval design.

If you are preparing for an interview, this is the best “story” document for the project.

---

## 4) app/ folder structure

## app/api/

This is the external API layer.

### app/api/main.py
This is the main FastAPI app.

Important responsibilities:
- Defines the FastAPI app object
- Exposes health check at /health
- Validates incoming query input
- Calls the RAG graph using get_rag_graph().invoke(...)
- Returns answer + sources
- Handles service errors with HTTP 503

Key models:
- QueryRequest: validates the user question
- Source: defines citation metadata
- QueryResponse: returns the final answer and sources

Important behavior:
- Query cannot be empty or whitespace-only
- RAG backend errors produce a 503 response
- The answer is grounded in retrieved document chunks

This is the interface users hit in production.

---

## app/rag/

This is the core intelligent orchestration layer.

### app/rag/state.py
Defines the LangGraph shared state.

State fields include:
- query
- rewritten_query
- route
- retrieved_documents
- reranked_documents
- relevance_grade
- answer
- sources

This is the memory/context container passed between nodes.

### app/rag/graph.py
This is the main orchestration graph.

It defines a LangGraph pipeline:
- query_router
- rewrite
- retrieve
- rerank
- query_intent
- generate_answer
- abstain

Important logic:
- If query is short, route to rewrite
- Else route to retrieval
- After retrieval and reranking, decide whether enough evidence exists
- If confidence is too low, ask the intent layer or abstain
- Otherwise answer with the LLM

This is the heart of the project.

---

## app/rag/nodes/

This folder contains every LangGraph node.

### query_router.py
Decides whether the query needs rewriting before retrieval.

Logic:
- Very short queries are considered weak and routed to rewrite
- Longer queries go straight to retrieval

This is a lightweight routing step.

### query_rewriter.py
Rewrites the user’s original question into a cleaner academic search query.

Uses:
- local Ollama
- HTTP request to localhost:11434/api/generate

Why it matters:
- Search quality improves when the question is reframed for research-paper retrieval

### retrieve.py
This is the retrieval node.

It does:
- dense search using Qdrant
- BM25 search using OpenSearch
- reciprocal rank fusion (RRF)
- reference filtering
- stores retrieved_documents in state

Important points:
- It uses BAAI/bge-small-en-v1.5 embedding model
- Dense retrieval uses cosine similarity
- BM25 retrieval uses lexical search
- Fusion merges dense and lexical signals
- Reference-only chunks are filtered out

### rerank.py
Takes the hybrid results and reranks them.

Uses:
- BAAI/bge-reranker-base
- Cross-encoder scoring
- Keeps only final top 5

This improves retrieval quality before answer generation.

### query_intent.py
Classifies the user query as:
- research_question
- ambiguous
- outside_scope

This prevents the system from answering irrelevant or weak queries.

### generate_answer.py
This is the final answer generation node.

It:
- takes reranked documents
- builds evidence context
- calls Ollama Qwen model
- ensures answer uses only evidence
- attaches source references like [Source 1], [Source 2]

Important interview point:
- The system is designed to be grounded and citation-aware
- It avoids outside knowledge unless supported by retrieved evidence

### abstain.py
This node likely handles cases where evidence is too weak or the topic is outside scope.

The final behavior is that the system either:
- answers with evidence
- or says it cannot answer reliably

---

## 5) app/ingestion/

This folder handles the full paper ingestion pipeline.

### download_papers.py
Downloads research papers from arXiv.

Important actions:
- Uses arxiv.Client
- Searches multiple arXiv queries
- Downloads PDFs to data/raw/papers
- Saves metadata to data/raw/metadata.json
- Avoids duplicates
- Sleeps between requests to respect rate limits

This is where the corpus is created.

### parse_pdfs.py
Uses PyMuPDF (fitz) to extract text from downloaded PDFs.

It:
- reads each PDF
- extracts page-by-page text
- stores a JSON object per paper
- preserves metadata like title, authors, abstract, categories

Output location:
- data/processed/

### clean_text.py
Cleans OCR-style and PDF-text artifacts.

Examples of fixes:
- broken encoding characters like â€™
- hyphenation across lines
- excessive whitespace
- line breaks inserted by PDFs

This step is crucial because extracted PDF text is often noisy.

### section_chunker.py
This is one of the most important files in the project.

It:
- detects academic section headings
- groups text under sections like Introduction, Method, Experiments
- creates chunks around each section
- stores chunk metadata like paper_id, title, section, page_numbers, text

Why it matters:
- The retrieval system works on chunk-level evidence, not whole paper text
- Section-aware chunking makes source references more useful

### embed_chunks.py
Generates embeddings for each chunk.

Uses:
- SentenceTransformer
- BAAI/bge-small-en-v1.5
- normalizes embeddings
- stores them in JSON

Output:
- data/processed_clean/embeddings.json

This creates the vector representation used for dense retrieval.

### verify_embeddings.py
Validates the generated embedding file.

Checks:
- number of records
- embedding dimension
- required fields
- invalid records

This is a data-quality validation script.

### redownload_failed.py
A utility script to retry a failed download or refresh specific papers.

Useful for recovery when some paper downloads fail.

---

## 6) app/retrieval/

This folder is focused on search infrastructure and evaluation.

### setup_qdrant.py
Creates the Qdrant collection for vector search.

Important settings:
- collection name: research_papers
- vector size: 384
- distance metric: cosine similarity

This is the dense retrieval index.

### load_qdrant.py
Loads vectors into Qdrant.

It usually reads the processed embeddings and pushes them into the collection.

This is how the system becomes searchable semantically.

### setup_opensearch.py
Creates the OpenSearch index for BM25 retrieval.

It:
- connects to localhost:9200
- creates the research_papers index
- maps chunk fields
- bulk-indexes all chunks

This is the lexical retrieval index.

### reference_filter.py
Filters out bibliography/reference-heavy chunks.

Why needed:
- reference sections can dominate retrieval and hurt answer quality
- it detects patterns like numbered references, DOI links, citation lists

This improves evidence quality.

### retrieval_metrics.py
Contains basic IR metrics:
- precision_at_k
- recall_at_k
- MRR
- DCG
- nDCG

This is the foundation of the evaluation layer.

### evaluate_dense.py
Evaluates dense retrieval quality.

It compares retrieved chunks against relevance labels.

### evaluate_bm25.py
Evaluates BM25 retrieval quality.

### evaluate_hybrid.py
Measures hybrid retrieval quality combining dense + BM25.

This is typically a key experiment and usually the project’s “best method” story.

### evaluate_reranker.py
Measures the impact of reranking after hybrid retrieval.

This is important because reranking often boosts final answer quality.

### evaluate_grading_thresholds.py
Looks at score thresholds for relevance grading.

This helps tune when a document is considered “good enough”.

### evaluate_reranker_thresholds.py
Tuning script for reranker thresholds.

Useful during evaluation and system optimization.

### find_eval_candidates.py / find_eval_candidates_hybrid.py
Help identify candidate queries or documents for evaluation.

These are support scripts for deeper research and retrieval tuning.

### compare_retrieval_results.py
Compares different retrieval strategies and writes a CSV summary.

This is useful for ablation studies:
- dense
- BM25
- hybrid
- hybrid + reranker

### verify_qdrant.py
Checks whether the Qdrant collection is valid and populated as expected.

---

## 7) data/ folder

This folder holds all raw and processed project data.

### data/raw/
Contains downloaded papers and metadata.

Typical files:
- papers/*.pdf
- metadata.json

### data/processed/
Contains parsed PDFs as JSON documents, page by page.

### data/processed_clean/
Contains cleaned document text and generated chunk files.

Important file:
- chunks.json

This is the main dataset fed into search indexes.

### evaluation/
Contains evaluation queries and result files.

Important file:
- queries.json

This is where the system’s retrieval quality is measured against labeled relevance.

---

## 8) tests/

This folder contains validation tests for the project.

### test_api_validation.py
Tests the FastAPI behavior:
- health endpoint returns ok
- valid query returns 200
- empty queries fail with 422
- backend failures return HTTP 503

This is a solid test suite for API reliability.

### test_query_router.py
Tests routing logic:
- short queries route to rewrite
- normal questions route to retrieval

This validates the first decision step in the graph.

---

## 9) airflow/dags/

This is where orchestration jobs or scheduled tasks can be placed.

The project appears to use Airflow as an optional workflow layer, but the core RAG logic is mostly implemented directly in Python scripts and LangGraph.

---

## 10) How the full pipeline fits together

A typical project run looks like this:

1. Download data
   - download_papers.py

2. Parse PDFs
   - parse_pdfs.py

3. Clean text
   - clean_text.py

4. Create chunks
   - section_chunker.py

5. Embed chunks
   - embed_chunks.py

6. Create Qdrant index
   - setup_qdrant.py

7. Load vectors into Qdrant
   - load_qdrant.py

8. Create OpenSearch index
   - setup_opensearch.py

9. Query the system
   - app/api/main.py → app/rag/graph.py

10. Retrieval and response
   - query_router → rewrite/retrieve → rerank → generate_answer

At a high level, this project demonstrates:
- document ingestion
- vector search
- lexical search
- hybrid retrieval
- reranking
- graph-based orchestration
- API layer
- evaluation workflow

---

## 11) Interview talking points

If you are asked, “What did you build?”, you can say:

> I built a research-paper RAG system that ingests arXiv papers, parses and chunks the text, indexes content in both Qdrant and OpenSearch, uses hybrid retrieval and reranking, and answers questions with evidence-backed citations using a LangGraph pipeline and FastAPI API.

If asked, “What are the most important technical decisions?” you can explain:

- Dense retrieval for semantic matching
- BM25 for lexical precision
- Hybrid retrieval to combine both
- Reranker to improve evidence quality
- Query rewrite and intent routing for ambiguity handling
- Citation-aware answer generation to ground responses
- Validation and evaluation metrics to measure retrieval quality

If asked, “What is the architecture?” you can say:

> The system is a pipeline architecture with four layers: ingestion, retrieval, orchestration, and serving. The ingestion layer turns PDFs into chunked JSON data, the retrieval layer stores embeddings and BM25 indexes, the orchestration layer uses LangGraph state transitions to route and rewrite queries, and the API layer serves the final answer.

If asked, “What challenges did you face?” you can mention:

- noisy PDF extraction
- section detection and chunking
- reference-only content polluting results
- balancing recall vs precision
- deciding whether a query is answerable
- grounding the answer to retrieved evidence only

---

## 12) Short project summary

This project is a strong end-to-end AI engineering portfolio project because it covers:

- data ingestion
- data cleaning
- chunking
- embeddings
- vector DB
- keyword indexing
- retrieval evaluation
- LLM orchestration
- API development
- testing
- production-style system design

This is exactly the kind of project that demonstrates real-world RAG engineering beyond a simple prompt demo.

---

## 13) One-line interview answer

> This project is a research-paper RAG system that downloads arXiv content, extracts and chunks scientific text, stores semantic and keyword indexes, retrieves and reranks relevant evidence, and generates grounded, citation-aware answers through a LangGraph-based pipeline exposed via FastAPI.

If you want, I can also turn this into:
- a 2-minute interview answer
- a 5-minute technical explanation
- a “project architecture” explainer that sounds more polished and recruiter-friendly
- a shorter version specifically for GitHub README / portfolio use
