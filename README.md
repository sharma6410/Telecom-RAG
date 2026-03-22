# 📞 Telecom RAG Pipeline

> Turn raw telecom customer support call transcripts into a queryable, intelligent Q&A system using hybrid retrieval and LLM-powered answers.

---

## 🧩 Problem

A telecom company had hundreds of raw customer support call transcripts stored in Excel — unstructured, unsearchable, and impossible to analyse at scale. Analysts had to manually read through transcripts to find patterns in complaints, cancellations, or unresolved issues.

**This pipeline solves that.** Ask any natural language question about the calls and get a grounded, bullet-pointed answer in seconds.

---

## 🏗️ Architecture & Workflow

```
Excel Transcripts
      ↓
GPT-4.1 → Canonical JSON (customer, agent, issues, summary)
      ↓
LangChain Documents with Metadata
      ↓
Structure-aware Recursive Chunking (chunk_size=600, overlap=100)
      ↓
ChromaDB Vector Store  +  BM25 Index
      ↓
Hybrid Search (Vector + BM25, k=25 each)
      ↓
Merge & Deduplicate Candidates
      ↓
CrossEncoder Reranking (top 5)
      ↓
GPT-4.1 Context Compression → Bullet Points
      ↓
GPT-4.1 Answer Generation (grounded, facts-only)
      ↓
RAGAS Evaluation (Faithfulness + Answer Relevancy)
```

---

## ✨ Key Features

- **Canonical JSON extraction** — GPT-4.1 strips greetings and filler, preserving only structured call data: customer, agent, issues, actions taken, resolution status, and overall summary
- **Structure-aware chunking** — splits first by numbered issues (`1.`, `2.`, …), then applies recursive chunking inside each section (chunk size 600, overlap 100) to preserve context boundaries
- **Hybrid retrieval** — combines dense vector search (ChromaDB + OpenAI embeddings) with sparse BM25 keyword search, then merges and deduplicates results
- **CrossEncoder reranking** — uses `cross-encoder/ms-marco-MiniLM-L-6-v2` to score query-document pairs and select the top 5 most relevant chunks
- **Context compression** — GPT-4.1 reads the top 5 chunks and extracts only the bullet points relevant to the question before final answer generation
- **RAGAS evaluation** — measures `faithfulness` and `answer_relevancy` against ground-truth summaries using the `ragas` library

---

## 📁 Project Structure

```
Telecom_RAG/
├── Telecom_RAG.ipynb        # Main notebook — full pipeline end to end
├── Extended Call Tagging 1.xlsx   # Input data (call transcripts)
├── chroma_db/               # Persisted ChromaDB vector store
└── README.md
```

---

## 🛠️ Tech Stack

| Component | Tool |
|---|---|
| LLM | GPT-4.1 (OpenAI) |
| Embeddings | OpenAI Embeddings |
| Vector Store | ChromaDB |
| Sparse Retrieval | BM25 (rank-bm25) |
| Reranking | CrossEncoder (sentence-transformers) |
| Orchestration | LangChain |
| Evaluation | RAGAS, HuggingFace Datasets |
| Data | Pandas, OpenPyXL |
| Environment | Google Colab / Jupyter |

---

## ⚙️ Setup & Installation

### 1. Clone the repo
```bash
git clone https://github.com/YOUR_USERNAME/Telecom-RAG.git
cd Telecom-RAG
```

### 2. Install dependencies
```bash
pip install langchain langchain-community langchain-openai chromadb openai \
            sentence-transformers rank-bm25 ragas datasets pandas openpyxl
```

### 3. Add your OpenAI API key

Replace the `api_key` value in the notebook cells with your own key, or set it as an environment variable:

```python
import os
os.environ["OPENAI_API_KEY"] = "your-key-here"
```

> ⚠️ Never commit API keys to GitHub. Use environment variables or a `.env` file with `python-dotenv`.

### 4. Add your data

Place your Excel file at the path referenced in the notebook:
```
/content/Extended Call Tagging 1.xlsx
```
Ensure it has a sheet named `Transcriptions` with a `Transcription` column.

### 5. Run the notebook

Open `Telecom_RAG.ipynb` in Google Colab or Jupyter and run cells top to bottom.

---

## 💬 Example Queries

```python
answer("Which issues cause anger that leads to cancellations?")
answer("Which fixes actually improve customer sentiment?")
answer("Why are customers frustrated?")
answer("Which root causes affect both revenue and reputation?")
answer("Which customer issues were not resolved?")
```

Each returns a bullet-pointed, grounded answer drawn only from the call transcripts.

---

## 📊 Evaluation

The pipeline is evaluated using **RAGAS** with two metrics:

| Metric | Description |
|---|---|
| `faithfulness` | Are the answers factually grounded in the retrieved context? |
| `answer_relevancy` | Does the answer actually address the question asked? |

Ground-truth summaries are hand-crafted for 5 representative queries and used as reference during evaluation.

---

## 🔮 Potential Improvements

- Add Neo4j knowledge graph layer (GraphRAG) for entity-level reasoning across calls
- Stream answers progressively using LangChain streaming
- Build a Streamlit / Gradio front-end for non-technical analysts
- Add multi-turn conversation memory for follow-up questions
- Extend evaluation with `context_precision` and `context_recall` RAGAS metrics

---

## 👩‍💻 Author

**Aditi Sharma** — B.Tech CSE (Data Science), Class of 2026  
📧 aditias0101@gmail.com · [LinkedIn](https://linkedin.com/in/aditi-sharma-5a1a31289)
