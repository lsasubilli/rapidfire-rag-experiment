## Dataset Structure (NFCorpus – Canonical Format)

This experiment uses **canonicalized NFCorpus files** with **integer IDs**, ensuring consistent joins across queries, corpus, and relevance judgments.

All files are produced by the notebook’s normalization step and are guaranteed to be ID-consistent.

---

### 1) Queries (`queries.final.jsonl`)

Each line is one JSON object with:

- `query_id` (integer)
- `query` (string)

```jsonl
{"query_id": 7, "query": "What causes muscle fatigue during exercise?"}
{"query_id": 9, "query": "How does vitamin D affect bone health?"}
```

---

### 2) Corpus (`corpus.final.jsonl`)

Each line is one JSON object with:

- `_id` (integer document ID)
- `text` (string document content)

```jsonl
{"_id": 256, "text": "Vitamin D supports calcium absorption and bone metabolism. Deficiency can lead to weakened bones."}
{"_id": 128, "text": "Muscle fatigue can result from metabolite buildup, reduced glycogen availability, and impaired neuromuscular signaling."}
```

These documents are **chunked and indexed offline** before retrieval.

---

### 3) Relevance Judgments (`qrels.final.tsv`)

Tab-separated rows in the format:

```
query_id<TAB>corpus_id<TAB>relevance
```

Example:

```tsv
9	256	1
7	128	1
```

---

### How This Maps to the Experiment

- **Retrieval output (per query):**
```
retrieved_documents = [corpus_id, ...]
```

- **Ground truth (per query):**
```
ground_truth_documents = [corpus_id, ...]
```

These mappings are used directly to compute:

- Precision
- Recall
- F1
- NDCG@5
- MRR

No generation output is used for evaluation — this is a **retrieval-first RAG experiment**.
