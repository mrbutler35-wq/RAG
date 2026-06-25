# Enterprise Dual-Gated RAG Knowledge Assistant

## Project Overview
This repository contains a production-oriented Retrieval-Augmented Generation (RAG) knowledge assistant designed for high-trust question answering over domain documentation.

The system uses semantic retrieval to locate relevant context, extractive question answering to produce grounded spans, and a dual-gated confidence policy to prevent unsupported responses. Instead of forcing a direct answer for every query, the assistant reliably transitions to a fallback response when retrieval quality or extractive confidence is insufficient.

## Core Features
- Semantic text retrieval over chunked domain documents using dense vector search.
- Manual token-span logit extraction from a Hugging Face QA model, bypassing high-level pipeline constraints for tighter control over start/end span selection.
- Dual-gated confidence reliability guardrail:
  - Gate 1: Retrieval confidence derived from vector distance alignment.
  - Gate 2: Generation confidence derived from extractive span probability.
  - Final decision: Return grounded answer only when fused confidence exceeds threshold; otherwise return low-confidence fallback.
- Source attribution for every response to support traceability and reviewability.
- Structured evaluation dashboard for category-level QA performance analysis.

## Tech Stack
| Technology | Role in System |
|---|---|
| ChromaDB | Vector database for chunk indexing and semantic nearest-neighbor retrieval |
| Hugging Face Transformers | Extractive QA model and tokenizer for token-span scoring |
| PyTorch | Runtime for model inference and logit/probability computation |
| Sentence-Transformers | Embedding generation for document and query vectorization |

## System Architecture
The end-to-end pipeline follows this sequence:

Document Chunking -> Embedding Generation -> ChromaDB Vector Storage -> Query Matching -> Extractive Span Scoring -> Confidence Threshold Gating

### 1) Document Chunking
Domain documents are segmented into overlapping chunks to preserve local context while improving retrieval precision.

### 2) Embedding Generation
Each chunk is converted into a dense vector representation using a sentence-transformer embedding model.

### 3) ChromaDB Vector Storage
Embeddings and metadata are stored in ChromaDB for fast semantic lookup and source traceability.

### 4) Query Matching
A user question is embedded and matched against stored vectors to retrieve top-k context candidates.

### 5) Extractive Span Scoring
A QA model computes start/end logits across retrieved context. The assistant manually selects span boundaries and decodes the most probable answer span.

### 6) Confidence Threshold Gating
Two confidence signals are fused:
- Retrieval confidence from vector distances.
- Extractive confidence from token-span probabilities.

If fused confidence is below threshold, the system returns a controlled fallback response rather than an ungrounded answer.

## Installation and Usage
### 1) Clone the repository
```bash
git clone https://github.com/<your-username>/<your-repo>.git
cd <your-repo>
```

### 2) Install dependencies
```bash
python -m venv .venv
# Windows PowerShell
.\.venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 3) Run a sample script execution
```bash
python run_rag_demo.py
```

### 4) Optional notebook execution
```bash
jupyter notebook RAG_Project_Larry.ipynb.ipynb
```

## Performance Evaluation Suite
The project evaluates behavior across four categories:
- Simple Facts
- Detailed Explanations
- Complex Comparison
- Out-of-Bounds Edge Cases

### Why this evaluation design matters
- Simple Facts and Detailed Explanations validate direct factual extraction from retrieved context.
- Complex Comparison tests the limits of single-span extractive reasoning across multi-fact prompts.
- Out-of-Bounds Edge Cases verify hallucination resistance by ensuring fallback activation when confidence is low.

### Fallback behavior and confidence handling
Confidence scoring is intentionally conservative. When retrieval alignment is weak, span quality is poor, or extracted content is degenerate, the assistant suppresses direct answering and emits a low-confidence fallback. This makes failure modes explicit and substantially reduces hallucinated output in unanswerable-query scenarios.

## Repository Notes
- Primary implementation is in the notebook: `RAG_Project_Larry.ipynb.ipynb`.
- For production deployment, move core components into Python modules (ingestion, retrieval, QA, evaluation) and wire them into a CLI or API service.
