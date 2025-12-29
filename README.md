# RAG Experiment Summary — NFCorpus (Retrieval-First)

> **TL;DR**  
> We designed a controlled retrieval-first RAG experiment on NFCorpus to study how chunk size and retrieval depth interact under strict context limits.  
> The best-performing and most deployable configuration uses **256-token chunks with k=8 retrieval**, achieving the highest ranking quality (NDCG@5) while avoiding prompt overflows.

---

## Links

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](
https://colab.research.google.com/github/lsasubilli/rapidfire-rag-experiment/blob/main/notebooks/nfcorpus_rag_experiment.ipynb
)

[![View Notebook](https://img.shields.io/badge/View-Notebook-blue)](
https://github.com/lsasubilli/rapidfire-rag-experiment/blob/main/notebooks/nfcorpus_rag_experiment.ipynb
)

[![GitHub Repo](https://img.shields.io/badge/GitHub-Repository-black)](
https://github.com/lsasubilli/rapidfire-rag-experiment
)

> Click **Open in Colab** to reproduce the experiment end-to-end.

---

## Screenshots

- **Results Table**  
  ![Results Table](https://raw.githubusercontent.com/lsasubilli/rapidfire-rag-experiment/main/screenshots/results_table.png)  

- **Metrics Trends**  
  ![Metrics Graphs](https://raw.githubusercontent.com/lsasubilli/rapidfire-rag-experiment/main/screenshots/metrics_graphs.png)

- **Experiment Progress (Interactive Control)**  
  ![Experiment Progress](https://raw.githubusercontent.com/lsasubilli/rapidfire-rag-experiment/main/screenshots/experiment_progress_ic.png)

- **Configuration Grid**  
  ![Experiment Configs](https://raw.githubusercontent.com/lsasubilli/rapidfire-rag-experiment/main/screenshots/experiment_set_configs.png)

---

## What We Built

This project implements a **retrieval-first RAG system for biomedical question answering** using the public **NFCorpus** dataset, evaluated with **RapidFire AI**.

The goal is to **maximize retrieval ranking quality (NDCG@5, MRR)** while remaining **stable under strict LLM context-length constraints**, avoiding prompt overflows and excessive latency.  
This mirrors real-world RAG systems where bounded context windows and predictable performance matter as much as raw recall.

NFCorpus is a realistic biomedical QA benchmark where **retrieval errors directly impact answer correctness**, making it well-suited for studying ranking quality and system stability.

---

## Setup

- **Corpus:** NFCorpus biomedical abstracts  
- **Queries / Eval set:** NFCorpus QA queries with human relevance judgments (qrels)  
- **Chunking:** Recursive text splitting with fixed overlap  
- **Embeddings:** `sentence-transformers/all-MiniLM-L6-v2`  
- **Retriever:** FAISS similarity search  
- **Retrieval depth (k):** 8 and 16  
- **Reranker:** `cross-encoder/ms-marco-MiniLM-L6-v2`  
- **Generator:** `Qwen/Qwen2.5-0.5B-Instruct` (3000-token context)  
- **Compute:** Google Colab (free tier, T4 GPU), ~15–20 min runtime  

---
## RAG Pipeline Overview

![RAG Pipeline Overview](https://raw.githubusercontent.com/lsasubilli/rapidfire-rag-experiment/main/screenshots/rag_final.drawio.png)
---

## Experiments (Controlled Changes)

We ran **controlled experiments**, varying **one RAG parameter at a time** while holding all others constant.

- **Baseline**  
  `chunk_size=128`, `chunk_overlap=32`, `k=8`, `top_n=2`

- **Variant A — Deeper Retrieval**  
  Increased retrieval depth: `k=16`

- **Variant B — Larger Chunks**  
  Increased chunk size: `chunk_size=256`

- **Variant C — Stress Test (Failure Case)**  
  `chunk_size=256` + `k=16`

---

## Results — Successful Runs

| Chunk Size | Retrieval k | Precision | Recall | F1 | NDCG@5 | MRR | Time |
|-----------:|------------:|----------:|-------:|---:|-------:|----:|-----:|
| **256** | **8** | **0.476** | 0.059 | 0.096 | **0.148** | 0.467 | **176s** |
| 128 | 8 | 0.471 | 0.045 | 0.078 | 0.124 | **0.492** | 87s |
| 128 | 16 | 0.422 | **0.071** | **0.110** | 0.115 | 0.421 | **74s** |

---

## Winning Configuration (Best Tradeoff)

- **Chunk Size:** 256  
- **Retrieval k:** 8  
- **Reranker top_n:** 2  

**Why this wins:**
- Achieves the **highest NDCG@5**, indicating better ranking quality
- Maintains strong precision with stable retrieval behavior
- Avoids prompt overflow failures seen in deeper retrieval + large chunks
- Latency remains acceptable given the ranking quality gains

While chunk_size=256, k=8 achieves the highest ranking quality (NDCG@5),
chunk_size=128, k=16 offers a faster and more robust configuration under strict latency constraints.

This configuration provides the **best balance between ranking quality and system stability**, which is critical for real-world RAG deployments.

---

## Failure Case Insight

The configuration `chunk_size=256, k=16` **failed deterministically** due to **decoder prompt length overflow**, even before generation.

This highlights a key retrieval-first RAG insight:

> Increasing chunk size and retrieval depth simultaneously can exceed context limits **during retrieval aggregation**, not generation.

RapidFire AI surfaced this failure clearly, allowing us to reason about **system constraints rather than silent metric degradation**.

---

## Consistency of Results

Across multiple runs, we observed **stable directional trends**:

- Larger chunks improve ranking quality **only up to a point**
- Increasing retrieval depth improves recall but can degrade ranking precision
- Optimal performance requires **joint tuning** of chunk size and retrieval depth

Even with small sample sizes, the **relative ordering of configurations remained consistent**, reinforcing the robustness of the observed tradeoffs.

---

## Key Takeaways

- Retrieval quality improves when chunk size and retrieval depth are tuned **together**
- Larger chunks alone do not guarantee better ranking
- Deeper retrieval increases recall but can hurt ranking quality and stability
- Prompt overflows are a **retrieval-stage failure mode**, not just a generation issue
- **RAG optimization is about tradeoffs**, not maximizing any single metric

---

## Why RapidFire AI Helped

RapidFire AI enabled fast, reproducible RAG experimentation by:

- Running multiple pipeline variants in parallel
- Making failure cases explicit (not silent)
- Separating retrieval configuration from evaluation logic
- Supporting interactive control (clone, stop, modify) mid-experiment

This workflow closely mirrors how real AI teams evaluate and harden RAG pipelines before deployment.


