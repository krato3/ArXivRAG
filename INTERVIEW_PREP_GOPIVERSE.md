# 🎯 Final Round Interview Prep — Gopiverse Universal IT Solutions

> **You are the only candidate selected for the final round.** You scored highest in both previous rounds.
> This document is your complete 10-hour preparation guide.

---

## 📋 Table of Contents

1. [Your Position & Context](#1-your-position--context)
2. [10-Hour Roadmap](#2-10-hour-roadmap)
3. [Hour 1 — Own Your Story](#3-hour-1--own-your-story)
4. [Hour 2 — FastAPI & Your Assignment](#4-hour-2--fastapi--your-assignment)
5. [Hour 3 — Python Fundamentals](#5-hour-3--python-fundamentals)
6. [Hour 4 — Django & SQL](#6-hour-4--django--sql)
7. [Hour 5 — AI Integration (Core Differentiator)](#7-hour-5--ai-integration-core-differentiator)
8. [Hour 6 — Pandas & Scikit-learn](#8-hour-6--pandas--scikit-learn)
9. [Hour 7 — System Design for AI APIs](#9-hour-7--system-design-for-ai-apis)
10. [Hour 8 — Docker & Deployment](#10-hour-8--docker--deployment)
11. [Hour 9 — Mock Interview (Spoken Practice)](#11-hour-9--mock-interview-spoken-practice)
12. [Hour 10 — HR Round & Questions to Ask](#12-hour-10--hr-round--questions-to-ask)
13. [Complete Question Bank](#13-complete-question-bank)
14. [Your Project Quick-Reference](#14-your-project-quick-reference)
15. [Day-Of Checklist](#15-day-of-checklist)

---

## 1. Your Position & Context

| Item | Detail |
|------|--------|
| **Company** | Gopiverse Universal IT Solutions |
| **Role** | Python Developer / AI Integration Engineer (Fresher) |
| **Round 1** | Written answers — Python, Django, API, SQL, Pandas, Scikit-learn |
| **Round 2** | Assignment — Build a REST API (FastAPI, duplicate transaction detector) |
| **Your Status** | Highest scorer both rounds. **Only finalist.** |
| **What they want** | Someone who can build REST APIs AND integrate AI into systems |

### What Makes You the Right Candidate

- You've built **production-grade AI systems**, not just notebooks
- LexiRedact is **published on PyPI** — a real open-source library
- ArXivRAG is a full production RAG pipeline with evaluation metrics
- You've **deployed on AWS EC2** with Docker, Nginx, Gunicorn
- Your FastAPI assignment was delivered successfully and is exactly what they need

---

## 2. 10-Hour Roadmap

```
Hour 1  │ Own Your Story              │ Narrative, resume walk-through, 3-min intro
Hour 2  │ FastAPI & Assignment        │ Your submitted code + general FastAPI depth
Hour 3  │ Python Fundamentals         │ OOP, async, decorators, generators
Hour 4  │ Django & SQL                │ ORM, migrations, joins, window functions
Hour 5  │ AI Integration              │ RAG, LexiRedact, ArXivRAG, LLM APIs
Hour 6  │ Pandas & Scikit-learn       │ Data manipulation, ML pipelines, metrics
Hour 7  │ System Design               │ AI API design, scaling, trade-offs
Hour 8  │ Docker & Deployment         │ Dockerfile, docker-compose, Nginx, EC2
Hour 9  │ Mock Interview              │ Speak answers out loud — most important hour
Hour 10 │ HR Round + Questions        │ Behavioral prep, questions to ask them
```

> **Priority if short on time:** Hour 2 → Hour 5 → Hour 9

---

## 3. Hour 1 — Own Your Story

### Your 3-Minute Introduction (Write & Rehearse This)

```
"I'm Hussain, a Computer Engineering student (Honours in Data Science) at NMIET Pune,
graduating June 2026 with a CGPA of 8.36.

I build production-ready AI systems. Most of my work focuses on RAG pipelines,
NLP, and FastAPI backends.

Some highlights:
- LexiRedact — a privacy-preserving RAG middleware I published on PyPI
- ArXivRAG — a full hybrid retrieval RAG system with evaluation (1.000 Recall@10)
- SkimLit — deployed on AWS EC2 with Docker, 88% validation accuracy
- Prepway internship — deployed a full Django app on AWS EC2 with Nginx + Gunicorn

The reason I'm excited about this role is that your focus on REST API + AI integration
is exactly what I've been building. I'd love to apply this directly."
```

### Why They Should Hire You (3 points)
1. You already delivered the assignment successfully — less onboarding needed
2. You have deployed, production AI experience that most freshers don't
3. You understand the full stack: API → AI model → deployment → cloud

---

## 4. Hour 2 — FastAPI & Your Assignment

### Your Submitted Code — Know It Cold

```python
# Key logic to explain fluently:

for t in sorted(data, key=lambda x: x["time"]):        # Why sort first?
    key = (t["accountNumber"], t["amount"], t["transactionType"])
    prev = last_seen.get(key)

    if prev and datetime.fromisoformat(t["time"]) - \
       datetime.fromisoformat(prev["time"]) <= timedelta(minutes=5):
        duplicates.append(...)   # Only the SECOND occurrence is added

    last_seen[key] = t           # Always update to latest seen
```

**Be ready to explain:**
- Why sort before iterating → ensures chronological comparison
- Why `last_seen[key] = t` updates every time → sliding window behaviour
- Time complexity: O(n log n) for sort + O(n) for scan = **O(n log n)**
- Space complexity: O(k) where k = unique (account, amount, type) combos

### Likely Questions About Your Assignment

| Question | Your Answer |
|----------|-------------|
| Walk me through the logic | Sort → iterate → check 5-min window → track last seen |
| What if 3 duplicates in a row? | It catches all — trace: t1 saved, t2 flags t1→saved, t3 flags t2→saved |
| How to scale to 10M transactions? | Redis sorted sets for sliding window, Kafka for ingestion |
| How to add a database? | SQLAlchemy + PostgreSQL, async sessions with `databases` lib |
| How to add authentication? | JWT with `python-jose`, `Depends()` for protected routes |
| How to test this endpoint? | `pytest` + `httpx.AsyncClient` with `TestClient` from fastapi.testclient |

### FastAPI Concepts to Know

```python
# Pydantic model for request validation
from pydantic import BaseModel

class Transaction(BaseModel):
    accountNumber: str
    amount: float
    transactionType: str
    time: datetime

# Dependency Injection
from fastapi import Depends

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

@app.get("/transaction")
def transaction(db: Session = Depends(get_db)):
    ...

# Background Tasks
from fastapi import BackgroundTasks

@app.post("/ingest")
def ingest(data: dict, background_tasks: BackgroundTasks):
    background_tasks.add_task(process_async, data)
    return {"status": "queued"}
```

### FastAPI Quick Reference

| Concept | Know This |
|---------|-----------|
| Path params | `@app.get("/items/{item_id}")` |
| Query params | `def read(skip: int = 0, limit: int = 10)` |
| Request body | `BaseModel` class via Pydantic |
| Status codes | `from fastapi import HTTPException; raise HTTPException(404)` |
| Middleware | `@app.middleware("http")` |
| CORS | `CORSMiddleware` from `fastapi.middleware.cors` |
| Async | `async def` endpoint + `await` for I/O operations |
| Uvicorn vs Gunicorn | Uvicorn = ASGI server; Gunicorn = process manager (use both in prod) |

---

## 5. Hour 3 — Python Fundamentals

### OOP Essentials

```python
class BankAccount:
    _total_accounts = 0  # class variable

    def __init__(self, owner: str, balance: float = 0):
        self.owner = owner
        self._balance = balance          # "protected" convention
        BankAccount._total_accounts += 1

    @property
    def balance(self):
        return self._balance

    @balance.setter
    def balance(self, value):
        if value < 0:
            raise ValueError("Balance cannot be negative")
        self._balance = value

    @classmethod
    def get_total_accounts(cls):          # access class state
        return cls._total_accounts

    @staticmethod
    def validate_amount(amount):          # no access to class/instance
        return amount > 0

    def __repr__(self):
        return f"BankAccount(owner={self.owner}, balance={self._balance})"
```

### Decorators

```python
import functools, time

def timer(func):
    @functools.wraps(func)               # preserves __name__, __doc__
    def wrapper(*args, **kwargs):
        start = time.perf_counter()
        result = func(*args, **kwargs)
        print(f"{func.__name__} took {time.perf_counter() - start:.4f}s")
        return result
    return wrapper

@timer
def process_data(n):
    return sum(range(n))
```

### Generators

```python
# Memory-efficient — useful for large datasets
def chunked_reader(filepath, chunk_size=1000):
    with open(filepath) as f:
        chunk = []
        for line in f:
            chunk.append(line)
            if len(chunk) == chunk_size:
                yield chunk
                chunk = []
        if chunk:
            yield chunk
```

### Async / Await

```python
import asyncio, httpx

async def fetch_transactions(account_id: str):
    async with httpx.AsyncClient() as client:
        response = await client.get(f"/api/transactions/{account_id}")
        return response.json()

async def fetch_all(account_ids: list):
    tasks = [fetch_transactions(aid) for aid in account_ids]
    return await asyncio.gather(*tasks)   # concurrent, not sequential
```

### Key Python Gotchas

```python
# Mutable default argument — classic bug
def add_item(item, lst=[]):        # BAD — lst shared across calls
    lst.append(item)
    return lst

def add_item(item, lst=None):      # GOOD
    if lst is None:
        lst = []
    lst.append(item)
    return lst

# Shallow vs deep copy
import copy
a = [[1, 2], [3, 4]]
b = a.copy()           # shallow — inner lists still shared
c = copy.deepcopy(a)   # deep — fully independent
```

---

## 6. Hour 4 — Django & SQL

### Django ORM Quick Reference

```python
# Basic queries
Transaction.objects.all()
Transaction.objects.filter(amount__gte=500)
Transaction.objects.exclude(transactionType="CREDIT")
Transaction.objects.get(pk=1)                    # raises if not found

# Aggregations
from django.db.models import Sum, Count, Avg
Transaction.objects.values("accountNumber").annotate(total=Sum("amount"))

# Joins — avoids N+1 queries
Order.objects.select_related("user")             # ForeignKey — single JOIN
Order.objects.prefetch_related("items")          # ManyToMany — separate query

# Chaining
Transaction.objects
    .filter(transactionType="DEBIT")
    .values("accountNumber")
    .annotate(count=Count("id"))
    .order_by("-count")[:10]
```

### Your Prepway Deployment Stack

```
Browser → Nginx (reverse proxy, port 80/443)
       → Gunicorn (WSGI, manages worker processes)
       → Django app
       → Amazon RDS (PostgreSQL)
       → Amazon S3 (media files)
       → systemd (keeps Gunicorn alive on reboot)
```

**Be able to explain each component's role in one sentence.**

### SQL Must-Know Queries

```sql
-- Find duplicate transactions within 5 minutes (your assignment in SQL)
SELECT t1.accountNumber, t1.amount, t1.transactionType,
       t1.time AS time1, t2.time AS time2
FROM transactions t1
JOIN transactions t2
  ON  t1.accountNumber = t2.accountNumber
  AND t1.amount        = t2.amount
  AND t1.transactionType = t2.transactionType
  AND t2.time > t1.time
  AND t2.time <= t1.time + INTERVAL '5 minutes';

-- Window function — running total per account
SELECT accountNumber, amount, time,
       SUM(amount) OVER (
           PARTITION BY accountNumber
           ORDER BY time
           ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
       ) AS running_total
FROM transactions;

-- CTE — clean and readable
WITH debit_summary AS (
    SELECT accountNumber, COUNT(*) as debit_count, SUM(amount) as total
    FROM transactions
    WHERE transactionType = 'DEBIT'
    GROUP BY accountNumber
)
SELECT * FROM debit_summary WHERE debit_count > 5;
```

### SQL Concepts Table

| Concept | When to Use |
|---------|-------------|
| `WHERE` | Filter rows before grouping |
| `HAVING` | Filter groups after `GROUP BY` |
| `INNER JOIN` | Only matching rows on both sides |
| `LEFT JOIN` | All rows from left, NULL for no match on right |
| `ROW_NUMBER()` | Assign unique rank within partition |
| `LAG()` / `LEAD()` | Access previous/next row value |
| `INDEX` | Speed up `WHERE`, `JOIN`, `ORDER BY` on large tables |
| `CTE` | Complex multi-step logic, better readability than subquery |

---

## 7. Hour 5 — AI Integration (Core Differentiator)

> This section is what separates you from every other Python dev applying for this role.

### RAG in 60 Seconds (for non-technical interviewers)

> "RAG stands for Retrieval-Augmented Generation. Instead of asking an LLM to answer from
> memory — which causes hallucinations — you first retrieve relevant documents from a
> knowledge base, then give them to the LLM as context. It's like giving the model a cheat sheet
> before the exam."

### ArXivRAG — Know Your Architecture

```
PDF Ingestion (PyMuPDF)
    ↓
Text Cleaning + Section-aware Chunking (16,237 chunks from 279+ papers)
    ↓
Embeddings (BAAI/bge-small-en-v1.5)
    ↓
┌─────────────────┬──────────────────┐
│  Dense Retrieval │  Sparse (BM25)   │
│  (Qdrant)        │  (OpenSearch)    │
└────────┬─────────┴────────┬─────────┘
         └────────┬──────────┘
              Hybrid RRF
                  ↓
         BGE Cross-encoder Reranking
                  ↓
         LangGraph Agentic Workflow
         (routing → rewriting → retrieval → grading → iterative retrieval)
                  ↓
         LLM Answer Generation (Ollama)
                  ↓
         FastAPI REST API + Redis Cache + Docker
```

**Key Result:** Hybrid RRF → **1.000 Recall@10**, **0.933 MRR**

### LexiRedact — Know Your Architecture

```
Document Ingestion
    ↓
┌──────────────────────────────────┐
│        DUAL-PATH (parallel)       │
│                                  │
│  PII Detection (Presidio NER)    │
│         ↓                        │
│  Sanitized Text ──────────────→  │
│                                  │
│  Embedding Generation (FastEmbed)│
│         ↓                        │
│  Embeddings ──────────────────→  │
└──────────────────────────────────┘
              ↓
    ChromaDB (sanitized vectors only)
    Redis (session cache)
              ↓
    Retrieval — same quality, no PII stored
```

**Key Results:**
- 82.5% PII detection F1-score
- 78.9% reduction in sensitive data storage
- nDCG@5: 0.309 (matches unprotected baseline)
- 16.9ms lower median ingestion latency

**Published on PyPI** ← always mention this

### How to Integrate an LLM into a REST API

```python
# Example: Document Q&A endpoint
@app.post("/ask")
async def ask_question(query: str, document_id: str):
    # 1. Retrieve relevant chunks
    chunks = vector_db.similarity_search(query, k=5)

    # 2. Build prompt with context
    context = "\n".join([c.page_content for c in chunks])
    prompt = f"Context:\n{context}\n\nQuestion: {query}\nAnswer:"

    # 3. Call LLM
    response = ollama.chat(model="llama3", messages=[
        {"role": "user", "content": prompt}
    ])

    return {
        "answer": response["message"]["content"],
        "sources": [c.metadata for c in chunks]
    }
```

### Common AI Integration Questions

| Question | Key Points |
|----------|------------|
| RAG vs Fine-tuning? | RAG = updatable knowledge, cheaper; Fine-tuning = baked-in behaviour, expensive |
| How to prevent hallucination? | Ground with retrieved context, cite sources, set temperature low, add confidence checks |
| What is an embedding? | Dense vector representation of text; semantically similar texts → close vectors |
| Vector DB vs SQL? | Vector DB = similarity search on embeddings; SQL = exact/relational queries |
| What is chunking strategy? | How you split documents — affects retrieval quality; sentence/paragraph/section-aware |
| What is RRF? | Reciprocal Rank Fusion — combines ranked lists from multiple retrievers without score normalization |
| What is nDCG? | Normalized Discounted Cumulative Gain — measures retrieval quality accounting for position |

---

## 8. Hour 6 — Pandas & Scikit-learn

### Pandas Core Operations

```python
import pandas as pd

df = pd.read_csv("transactions.csv")

# Inspection
df.info()
df.describe()
df.isnull().sum()

# Filtering
df[df["amount"] > 500]
df[(df["amount"] > 500) & (df["transactionType"] == "DEBIT")]

# GroupBy
df.groupby("accountNumber").agg(
    total=("amount", "sum"),
    count=("amount", "count"),
    avg=("amount", "mean")
).reset_index()

# Apply
df["amount_category"] = df["amount"].apply(
    lambda x: "high" if x > 1000 else "low"
)

# Merge (like SQL JOIN)
merged = pd.merge(df_transactions, df_accounts,
                  on="accountNumber", how="left")

# Pivot
df.pivot_table(values="amount", index="accountNumber",
               columns="transactionType", aggfunc="sum", fill_value=0)
```

### Scikit-learn Pipeline

```python
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.ensemble import RandomForestClassifier
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import classification_report

# Preprocessing
preprocessor = ColumnTransformer([
    ("num", StandardScaler(), ["amount", "hour"]),
    ("cat", OneHotEncoder(), ["transactionType"])
])

# Pipeline
pipeline = Pipeline([
    ("preprocessor", preprocessor),
    ("classifier", RandomForestClassifier(n_estimators=100))
])

# Train / Evaluate
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)
pipeline.fit(X_train, y_train)
print(classification_report(y_test, pipeline.predict(X_test)))
```

### ML Metrics Cheat Sheet

| Metric | Formula | When to Use |
|--------|---------|-------------|
| Accuracy | TP+TN / Total | Balanced classes |
| Precision | TP / (TP+FP) | Cost of false positives is high |
| Recall | TP / (TP+FN) | Cost of false negatives is high (fraud detection!) |
| F1-Score | 2 × P×R / (P+R) | Imbalanced classes |
| ROC-AUC | Area under curve | Ranking/probability calibration |

> **For fraud detection (your assignment domain): prioritize Recall** — missing a fraud (FN) is worse than a false alarm (FP)

---

## 9. Hour 7 — System Design for AI APIs

### Framework for Any System Design Question

```
1. Clarify scope     → Who uses it? What scale? Latency budget? Real-time or batch?
2. Core components   → API → Queue → Processing → Storage → Cache → Response
3. Data flow         → Walk through a single request end-to-end
4. Bottlenecks       → Where does it break at scale? How do you fix it?
5. Trade-offs        → Consistency vs availability, cost vs latency
```

### Design: Duplicate Transaction Detector at Scale

```
Request
  ↓
FastAPI (load balanced, multiple instances)
  ↓
Redis Sorted Set (sliding window, key = account+amount+type)
  → Check if same transaction exists within last 5 minutes
  → If yes → flag as duplicate → push to alert queue
  ↓
PostgreSQL (persist all transactions for audit)
  ↓
Kafka (async processing of flagged duplicates)
  ↓
Alert Service (notify fraud team)
```

### Design: Document Q&A API with AI

```
User Query
  ↓
FastAPI endpoint
  ↓
Redis Cache (check if query was answered recently)
  ↓
Vector DB (Qdrant / ChromaDB) — semantic search
  ↓
BM25 (OpenSearch) — keyword search
  ↓
RRF Fusion + Reranking
  ↓
LLM (Ollama / OpenAI) — answer generation with context
  ↓
Response with source citations
  ↓
Log to MLflow / Langfuse for observability
```

### Key Trade-offs to Know

| Decision | Option A | Option B | When to pick A |
|----------|----------|----------|----------------|
| Storage | PostgreSQL | Vector DB | Relational/exact queries |
| Cache | Redis | In-memory | Multi-instance deployments |
| LLM | Local (Ollama) | API (OpenAI) | Cost/privacy constraints |
| Retrieval | Dense only | Hybrid | When keyword matching matters |
| Deployment | Docker single | K8s | Need horizontal scaling |

---

## 10. Hour 8 — Docker & Deployment

### Dockerfile for Your FastAPI App

```dockerfile
FROM python:3.11-slim

WORKDIR /app

# Install dependencies first (layer caching)
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

EXPOSE 8000

CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### Docker Compose (FastAPI + Redis + PostgreSQL)

```yaml
version: "3.9"
services:
  api:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - redis
      - db
    environment:
      - DATABASE_URL=postgresql://user:pass@db:5432/transactions

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

  db:
    image: postgres:15
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: transactions
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

### Your Prepway Deployment — Explain This Fluently

```
[EC2 Instance]
    systemd
      └─ gunicorn (4 worker processes)
            └─ Django WSGI app
                  └─ Amazon RDS (PostgreSQL) for DB
                  └─ Amazon S3 for media

    nginx (port 80/443)
      └─ reverse proxy → gunicorn (port 8000)
      └─ serves static files directly (no Django overhead)
```

**Why Nginx in front of Gunicorn?**
- Handles slow clients (buffers requests)
- Serves static files 10x faster
- SSL termination
- Rate limiting, logging

---

## 11. Hour 9 — Mock Interview (Spoken Practice)

> **This is the most important hour. Do not skip it.**
> Pick 10 questions, set a timer, and answer OUT LOUD as if you're in the interview room.

### 10 Questions to Speak Through

1. "Walk me through your duplicate transaction detection code."
2. "What is RAG and how did you use it in ArXivRAG?"
3. "Tell me about LexiRedact — what problem does it solve?"
4. "What's the difference between `select_related` and `prefetch_related` in Django?"
5. "Write a SQL query to find duplicate transactions within 5 minutes."
6. "How would you scale your FastAPI app to handle 1 million requests per day?"
7. "Explain precision vs recall. For fraud detection, which matters more?"
8. "How did you deploy your Prepway app? Walk me through the stack."
9. "What is your biggest project and what would you do differently?"
10. "Why do you want to work at Gopiverse specifically?"

### Rules for Mock Practice
- No reading from notes while answering
- If you blank out, pause 3 seconds, then start with "So the core idea is..."
- Time each answer — aim for 2-3 min max per technical question
- Record yourself if possible

---

## 12. Hour 10 — HR Round & Questions to Ask

### Behavioral Questions with STAR Answers

**"Tell me about a challenge in a project"**
> **S:** In LexiRedact, the PII detection and embedding pipeline were running sequentially.
> **T:** The ingestion latency was too high compared to a baseline without redaction.
> **A:** I redesigned the pipeline to run PII detection and embedding generation in parallel using dual threads.
> **R:** Reduced median ingestion latency by 16.9ms, matching the unprotected baseline's retrieval performance.

**"What's something you built that didn't work the way you expected?"**
> Talk about the Enron dataset benchmark project where you identified that circular query generation was a conceptual flaw — and how you pivoted.

**"Why should we hire you over others?"**
> "I'm the only candidate who made it this far, which tells you I've already demonstrated what you're looking for. I've built and deployed the exact combination you need — REST APIs and AI integration — not as tutorials but as real systems. LexiRedact is on PyPI, ArXivRAG has evaluation benchmarks, and your assignment was delivered successfully."

### HR Questions & Suggested Answers

| Question | Key Points |
|----------|------------|
| Why Gopiverse? | Research their product tonight. Tie it to API + AI work. |
| Where in 2 years? | Building production AI systems, growing into AI engineering. |
| Strength? | Production mindset — I build things that work in the real world, not just notebooks. |
| Weakness? | "I tend to over-engineer for scale early — I'm learning to balance that with shipping fast." |
| Salary expectations? | Research Pune fresher Python dev salaries. Be confident, give a range. |

### Questions YOU Ask Them (Pick 2)

1. "What does the AI integration roadmap look like for your products?"
2. "What does the tech stack look like for the team I'd be joining?"
3. "What does success look like in the first 90 days?"
4. "Will I have the opportunity to work on both the backend API and the AI components?"

---

## 13. Complete Question Bank

### FastAPI / REST API

1. What is the difference between `PUT` and `PATCH`?
2. How do you handle request validation in FastAPI?
3. What is a Pydantic model and why is it useful?
4. How would you add rate limiting to a FastAPI endpoint?
5. Explain `Depends()` with an example.
6. How would you handle errors and return proper HTTP status codes?
7. What's the difference between synchronous and async endpoints?
8. Walk me through your duplicate transaction detection code.
9. How would you add database persistence to your assignment?
10. How would you write a unit test for your `/transaction` endpoint?
11. What is the difference between `uvicorn` and `gunicorn`?
12. How would you add JWT authentication to your API?
13. What is CORS and how do you enable it in FastAPI?
14. How would you implement pagination in an API endpoint?

### Python

15. What is a decorator? Write one that logs execution time.
16. What's the difference between `@staticmethod` and `@classmethod`?
17. Explain generators — give a real use case.
18. What is the GIL and when does it matter?
19. How does `async`/`await` work in Python?
20. What's the difference between `deepcopy` and `copy`?
21. How would you handle a file too large to load into memory?
22. What is `functools.wraps` and why use it?
23. Explain `*args` and `**kwargs`.
24. What is a context manager? Write a custom one.

### Django

25. Explain Django's ORM — how is it different from raw SQL?
26. What is `select_related` vs `prefetch_related`? When to use each?
27. How did you deploy your Prepway app? Walk through the full stack.
28. What is Django middleware? Give a use case.
29. How do Django migrations work?
30. What is the difference between Django and FastAPI?
31. What is `annotate()` in Django ORM?

### SQL

32. Write a query to find duplicate transactions within 5 minutes for the same account.
33. What's the difference between `WHERE` and `HAVING`?
34. When would you use a CTE over a subquery?
35. Explain window functions — give an example using transaction data.
36. What is an index and when would you add one?
37. What is the difference between `INNER JOIN` and `LEFT JOIN`?
38. Write a query to find the top 3 accounts by total debit amount.

### AI / ML / RAG

39. What is RAG and when would you use it vs fine-tuning?
40. Explain how you built ArXivRAG — what was the hardest part?
41. What is Reciprocal Rank Fusion and why did it outperform dense-only retrieval?
42. What is LexiRedact solving and how does the dual-path architecture work?
43. What is an embedding? How are they used in retrieval?
44. How would you integrate an LLM into a REST API?
45. What is hallucination in LLMs and how do you mitigate it?
46. Explain precision, recall, and F1. For fraud detection, which matters more?
47. What does `nDCG` measure and why did you use it?
48. What is overfitting and how do you fix it?
49. What's the difference between a SQL database and a vector database?
50. How does chunking strategy affect RAG quality?

### System Design

51. How would you scale your transaction duplicate detector to 1M requests/day?
52. Design a simple document Q&A API using AI.
53. How would you add caching to your FastAPI service?
54. What is a message queue and when would you use one?
55. How would you monitor an AI API in production?

### Behavioral

56. Tell me about a time you debugged a difficult problem.
57. Which project are you most proud of and why?
58. Why do you want to work here specifically?
59. What's something you built that didn't work as expected?
60. How do you stay updated with AI/ML developments?

---

## 14. Your Project Quick-Reference

### ArXivRAG

| Item | Detail |
|------|--------|
| Type | Production RAG system |
| Data | 279+ arXiv papers, 16,237 chunks |
| Embeddings | BAAI/bge-small-en-v1.5 |
| Dense retrieval | Qdrant |
| Sparse retrieval | OpenSearch (BM25) |
| Fusion | Reciprocal Rank Fusion (RRF) |
| Reranking | BGE cross-encoder |
| Orchestration | LangGraph |
| LLM | Ollama (local) |
| API | FastAPI + Redis + Docker |
| Best result | Hybrid RRF: 1.000 Recall@10, 0.933 MRR |

### LexiRedact

| Item | Detail |
|------|--------|
| Type | Privacy-preserving RAG middleware |
| Problem | PII leaking into vector databases |
| PII detection | Microsoft Presidio (NER) |
| Embedding | FastEmbed |
| Vector DB | ChromaDB |
| Cache | Redis |
| Dataset | Enron email dataset |
| PII F1-score | 82.5% |
| Storage reduction | 78.9% |
| Latency improvement | -16.9ms median ingestion |
| Published | PyPI ✅ |

### SkimLit

| Item | Detail |
|------|--------|
| Type | Sequential sentence classification |
| Task | Classify abstract sentences: BACKGROUND / OBJECTIVE / METHODS / RESULTS / CONCLUSIONS |
| Dataset | PubMed 20k RCT |
| Architecture | Tribrid: MiniLM + Character BiLSTM + Positional Embeddings |
| Accuracy | 88.0% validation, 87.6% test |
| Deployment | AWS EC2 via Docker Hub |
| Interface | Streamlit |

### Prepway Internship

| Item | Detail |
|------|--------|
| Role | Python Developer Intern |
| Duration | Nov 2024 – Feb 2025 |
| Project | Full-stack food ordering web app |
| Framework | Django |
| Deployment | AWS EC2, Gunicorn, Nginx, systemd |
| Database | Amazon RDS (PostgreSQL) |
| Storage | Amazon S3 |
| Status | Live |

---

## 15. Day-Of Checklist

```
Before leaving:
  [ ] Resume printed (2 copies)
  [ ] GitHub open on phone — ArXivRAG, LexiRedact, SkimLit
  [ ] LexiRedact PyPI page bookmarked
  [ ] Your assignment code reviewed one last time
  [ ] 3-minute intro rehearsed

Mental state:
  [ ] You are the ONLY candidate selected
  [ ] You scored highest in BOTH rounds
  [ ] You already delivered their assignment successfully
  [ ] Your projects are exactly what they're looking for

In the room:
  [ ] Pause before answering — it's a sign of thoughtfulness, not weakness
  [ ] If you don't know something: "I haven't worked with that specifically,
      but here's how I'd approach it..."
  [ ] End with questions — shows genuine interest
```

---

> **Final reminder:** The job is yours to lose. Walk in confident. You've earned this spot.

---

*Prepared for: Baihela Abid Hussain | Final Round — Gopiverse Universal IT Solutions*
