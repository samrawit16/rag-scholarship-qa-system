# RAG Scholarship Q&A System

A Retrieval-Augmented Generation (RAG) system that answers questions about
international scholarships — **eligibility requirements, application procedures,
and deadlines** — using only official program websites, with a citation for
every claim.

## The Problem

Large language models answer from memory and can **hallucinate** — inventing
deadlines or misstating eligibility rules with total confidence. For scholarship
information, where one wrong date can cost a student an entire year, that is
unacceptable. Meanwhile, the real information is scattered across dozens of
official websites in different countries.

## The Solution

This system **retrieves** the most relevant passages from a corpus of official
scholarship pages *before* answering, then builds the answer **grounded only in
those passages** — with citations the user can verify. When a question falls
outside the corpus, the system **refuses to answer** rather than guess.

## Pipeline

| Stage | What it does | Output |
|---|---|---|
| 1. Collection | Hybrid crawler: curated official URLs + breadth-first same-domain discovery, politeness delay, provenance headers | 100 raw documents (source URL, license, crawl date) |
| 2. Cleaning | Encoding fixes (mojibake), Unicode normalisation, boilerplate removal, whitespace cleanup, MD5 fingerprint deduplication | Clean corpus (~80 documents) |
| 3. Indexing | 200-word overlapping chunks (50-word overlap) → MiniLM-L6-v2 embeddings (384-dim) → ChromaDB (cosine) | Vector database of chunks with source metadata |
| 4. Retrieval | Question embedded with the same model; top-5 chunks by cosine similarity | Grounded evidence per question |
| 5. Generation | Extractive answer builder with two refusal gates (distance threshold + keyword coverage); citation on every sentence | Answer with sources — or honest refusal |
| 6. Evaluation | 10-question golden set: 8 in-scope questions + 2 out-of-corpus traps | Hit rate (top-5), MRR, abstain correctness |

## Key Design Decisions

- **200-word chunks with 50-word overlap** — focused units for precise semantic
  matching; the overlap prevents facts from being split at chunk boundaries.
- **MiniLM-L6-v2 embeddings** — small (80 MB), fast on CPU, proven retrieval
  quality; the right choice for a laptop without a GPU.
- **ChromaDB over FAISS** — FAISS is a pure search index, while Chroma stores
  vectors, text, and metadata (source URLs) together, making citations free.
- **Refusal gates instead of blind answering** — a distance threshold and a
  keyword-coverage check make the system say "I could not find this in my
  sources" on out-of-scope questions instead of hallucinating.
- **Provenance from day one** — every document carries its source URL, license
  line, and crawl date, so every answer is traceable to an official page.

## Repository Structure

```text
Ylsak_Samrawit_RAG_Assignment.ipynb   # the full pipeline, task by task
data/raw/                             # crawled documents with provenance headers
data/clean/                           # cleaned, deduplicated corpus
