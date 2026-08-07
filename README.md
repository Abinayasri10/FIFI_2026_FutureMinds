# FIFI @ FIRE 2026 - FutureMinds

<div align="center">

# Scientific Title Retrieval and Reconstruction using Dense Retrieval and Retrieval-Augmented Generation

### Official Repository for our submission to the **FIFI @ FIRE 2026 Shared Task**

**Team:** FutureMinds

Kongu Engineering College, Tamil Nadu, India

</div>

---

# Team Members

| Name | Email |
|------|------|
| Abinaya Sri J | abinayasrij.23cse@kongu.edu |
| Danush P | danushp.23cse@kongu.edu |
| Chandru K | chandruk.23cse@kongu.edu |

---

# Abstract

Scientific paper titles are frequently rewritten into different styles such as **technical**, **accessible**, and **catchy** to improve readability and audience engagement while preserving the underlying scientific meaning. Although semantically similar, these rewritten titles often differ substantially in wording, vocabulary, and structure, making it challenging to retrieve or reconstruct the original research title.

This repository presents our submission to the **FIFI @ FIRE 2026 Shared Task**, where we investigate dense semantic retrieval and sequence-to-sequence title reconstruction for scientific document understanding.

For **Task 1**, we formulate the problem as a large-scale semantic retrieval task and employ dense sentence embeddings generated using **BAAI/bge-base-en-v1.5**, indexed with **FAISS** for efficient nearest-neighbor retrieval. We further explore **semantic query expansion**, **CrossEncoder reranking**, and **retrieval fusion** to improve retrieval quality across different rewriting styles.

For **Task 2**, we investigate multiple reconstruction strategies, including **FLAN-T5 fine-tuning**, **Retrieval-Augmented Generation (RAG)**, and **retrieval-based reconstruction**, where the highest-ranked retrieved candidate is directly selected as the predicted original title.

---

# Shared Task Description

The FIFI @ FIRE 2026 Shared Task consists of two subtasks.

---

## Task 1 — Scientific Title Retrieval

### Objective

Given a rewritten scientific paper title, retrieve the corresponding original scientific title from a large candidate corpus.

Example

**Input**

```
How Do AI Language Models Learn Words?
```

**Output**

```
Word Acquisition in Neural Language Models
```

Evaluation Metric

```
MRR@10
```

---

## Task 2 — Scientific Title Reconstruction

### Objective

Given a rewritten title, generate or reconstruct the original scientific paper title.

Example

Input

```
From Snapshot to Sawdust:
Rebuilding Wooden Objects as Editable Designs
```

Output

```
Fabrication-Aware Reverse Engineering for Carpentry
```

Evaluation Metric

```
Token-Level F1
```

---

# Dataset

Official dataset provided by the organizers.

```
train.tsv
val.tsv
meta_train_titles.tsv
```

Dataset Statistics

| File | Description |
|-------|------------|
| train.tsv | Training dataset |
| val.tsv | Validation dataset |
| meta_train_titles.tsv | Additional candidate titles |

No external datasets were used.

---

# Repository Structure

```
FIFI_2026_FutureMinds/

│
├── data/
│      train.tsv
│      val.tsv
│      meta_train_titles.tsv
│
├── notebooks/
│      01_EDA.ipynb
│      02_BM25.ipynb
│      03_BGE_FAISS.ipynb
│      04_CrossEncoder.ipynb
│      05_Task1.ipynb
│      06_QueryExpansion.ipynb
│      07_Task1_Run3.ipynb
│      08_FLAN_T5.ipynb
│      09_RAG_Reconstruction.ipynb
|      10_Run3_Task1.ipynb
|      11_Task2_RUN3.ipynb
│
├── models/
│
├── submissions/
│
└── README.md
```

---

# Methodology

## Overall Pipeline

```
                 Rewritten Scientific Title
                           │
                           ▼
               Sentence Embedding (BGE)
                           │
                           ▼
                 Dense Retrieval (FAISS)
                           │
          ┌────────────────┴───────────────┐
          │                                │
          ▼                                ▼
   Task 1 Retrieval              Task 2 Reconstruction
                                          │
                                          ▼
                     FLAN-T5 / Retrieval-Based Prediction
```

---

# Task 1 Methodology

## Step 1 — Candidate Corpus Construction

The retrieval corpus was constructed by combining:

- Original titles from **train.tsv**
- Additional titles from **meta_train_titles.tsv**

Duplicate titles were removed before indexing.

---

## Step 2 — Dense Semantic Embedding

We used

```
BAAI/bge-base-en-v1.5
```

to encode every candidate title into a dense semantic embedding.

Unlike BM25, dense embeddings capture semantic similarity beyond lexical overlap.

---

## Step 3 — FAISS Indexing

Candidate embeddings were indexed using

```
FAISS IndexFlatIP
```

allowing efficient nearest-neighbor search over the complete corpus.

---

## Step 4 — Dense Retrieval

For every rewritten title

1. Generate embedding
2. Search FAISS
3. Retrieve Top-100 candidates

---

## Step 5 — Semantic Query Expansion

To improve retrieval robustness, rewritten titles were expanded using domain-specific scientific keywords.

Example

```
Image

↓

image vision segmentation detection cnn
```

This improves retrieval for heavily paraphrased titles.

---

## Step 6 — CrossEncoder Re-ranking

The Top retrieved candidates were re-ranked using

```
cross-encoder/ms-marco-MiniLM-L-6-v2
```

which evaluates semantic similarity for every query–candidate pair.

---

# Task 2 Methodology

We explored three reconstruction strategies.

---

## Approach 1 — FLAN-T5 Fine-Tuning

Model

```
google/flan-t5-base
```

Input Prompt

```
Recover the original scientific paper title.

Style:
technical

Rewritten Title:
...
```

Output

```
Original scientific paper title
```

---

## Approach 2 — Retrieval-Augmented Generation (RAG)

Top retrieved candidate titles were provided as additional context.

Prompt

```
Recover the ORIGINAL scientific paper title.

Style:
technical

Rewritten Title:
...

Top Candidate Original Titles

1. ...
2. ...
3. ...
4. ...
5. ...

Return ONLY the original paper title.
```

This enables FLAN-T5 to leverage retrieval evidence during generation.

---

## Approach 3 — Retrieval-Based Reconstruction

Instead of generating a new title, the highest-ranked retrieved title was directly selected as the predicted original title.

This approach achieved the strongest validation Token-level F1 among the approaches we evaluated.

---

# Models Used

| Component | Model |
|-----------|-------|
| Dense Retriever | BAAI/bge-base-en-v1.5 |
| Generator | google/flan-t5-base |
| Re-ranker | cross-encoder/ms-marco-MiniLM-L-6-v2 |
| ANN Search | FAISS |

---

# Experimental Pipeline

```
EDA
 │
 ▼
BM25 Baseline
 │
 ▼
BGE Dense Retrieval
 │
 ▼
FAISS Retrieval
 │
 ▼
Semantic Query Expansion
 │
 ▼
CrossEncoder Re-ranking
 │
 ▼
Task 1 Submission
 │
 ▼
FLAN-T5 Fine-Tuning
 │
 ▼
Retrieval-Augmented Generation
 │
 ▼
Retrieval-Based Reconstruction
 │
 ▼
Task 2 Submission
```

---

# Installation

```bash
pip install sentence-transformers
pip install transformers
pip install faiss-cpu
pip install datasets
pip install accelerate
pip install evaluate
pip install rank-bm25
```

---

# Running Task 1

```bash
python task1_dense_retrieval.py
```

Output

```
FutureMinds_task1_run1.tsv
```

---

# Running Task 2

```bash
python task2_reconstruction.py
```

Output

```
FutureMinds_task2_run1.tsv
```

---

# Evaluation Metrics

## Task 1

Mean Reciprocal Rank

```
MRR@10
```

---

## Task 2

Token-Level F1

```
Precision
Recall
F1
```

---

# Example Retrieval

**Rewritten Title**

```
How Do AI Language Models Learn Words?
```

↓

Retrieved Candidates

```
Word Acquisition in Neural Language Models
Learning Vocabulary in Neural Networks
...
```

↓

Predicted Original Title

```
Word Acquisition in Neural Language Models
```

---

# Example Reconstruction

Input

```
Reading faces zone by zone
```

↓

Predicted

```
Facial expression recognition based on local regions
```

---

# Future Improvements

Possible future directions include

- Hybrid Sparse + Dense Retrieval
- Fine-tuning BGE using contrastive learning
- Larger scientific embedding models
- Metadata-aware retrieval using abstracts and keywords
- Graph Neural Networks for citation-aware retrieval
- Retrieval-Augmented Large Language Models
- Multi-stage reranking with DeBERTa CrossEncoder
- End-to-End Neural Retrieval

---

# Reproducibility

Random seeds were fixed where applicable.

Experiments were conducted using

- Google Colab
- NVIDIA T4 GPU

Main Libraries

- Transformers
- SentenceTransformers
- FAISS
- PyTorch
- HuggingFace Datasets
- Evaluate

---

# Results

We evaluated multiple retrieval and reconstruction approaches during development, including dense retrieval, semantic query expansion, CrossEncoder reranking, FLAN-T5 fine-tuning, retrieval-augmented generation, and retrieval-based reconstruction. Performance was measured using **MRR@10** for Task 1 and **Token-Level F1** for Task 2 on the official validation set. These experiments helped us compare different retrieval and reconstruction strategies and guided the selection of our final submitted runs.

---

# Acknowledgements

We sincerely thank the organizers of **FIFI @ FIRE 2026** for organizing this challenging shared task and for providing the datasets, evaluation framework, and baseline resources that enabled this research.


---

# GitHub Repository

https://github.com/Abinayasri10/FIFI_2026_FutureMinds

---

# Contact

**FutureMinds**

Department of Computer Science and Engineering

Kongu Engineering College

Tamil Nadu, India
