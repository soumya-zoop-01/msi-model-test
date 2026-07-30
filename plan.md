# Owner Name Entity Resolution Pipeline

## Execution Plan

---

# Goal

Cluster similar owner names into a single canonical owner while minimizing false positives.

Dataset Size

- ~349,000 owner names

Embedding Model

- Qwen Embedding Model

Search Engine

- FAISS

Clustering

- Union Find (Disjoint Set)

---

# Overall Architecture

Raw Owner Name
│
▼
Data Cleaning
│
▼
Canonical Name Generation
│
▼
Embedding Generation
│
▼
FAISS Candidate Search
│
▼
Feature Engineering
│
▼
Rule Engine
│
▼
Union Find
│
▼
Cluster Creation
│
▼
Canonical Owner Name Selection

---

# Phase 1 : Data Cleaning

Goal

Standardize owner names before embedding.

Input

Original Owner Name

Example

D P JAIN & COMPANY INFRASTRUCTURE PRIVATE LIMITED

Cleaning Steps

□ Convert to uppercase

□ Remove extra spaces

□ Replace multiple spaces with single space

□ Replace punctuation

&
→
AND

,

.

-

/

()

etc.

□ Unicode normalization

Example

É

→

E

□ Trim leading/trailing spaces

Output

D P JAIN AND COMPANY INFRASTRUCTURE PRIVATE LIMITED

Validation

Randomly inspect 100 records.

Cleaning should never remove meaningful words.

---

# Phase 2 : Typo Normalization

Goal

Correct common business-word OCR mistakes.

Examples

TRAVLES
→
TRAVELS

LIMITD
→
LIMITED

COMPNY
→
COMPANY

COOPERATIV
→
COOPERATIVE

INFRASTRUOTURE
→
INFRASTRUCTURE

Implementation

Build

ocr_dictionary

Apply token-level replacement.

Do NOT normalize

Person names

Company names

City names

Validation

Print

Before

↓

After

for

200 corrected names.

Reject incorrect corrections.

---

# Phase 3 : Canonical Name Generation

Goal

Remove legal suffixes.

Keep industry descriptors.

Remove

PRIVATE

LIMITED

PVT

LTD

LLP

AND

COMPANY

CO

Example

Original

ABC TOURS AND TRAVELS PRIVATE LIMITED

Canonical

ABC TOURS TRAVELS

Example

Original

D P JAIN AND COMPANY INFRASTRUCTURE PRIVATE LIMITED

Canonical

D P JAIN INFRASTRUCTURE

Store

original_name

clean_name

canonical_name

Validation

Compare

Original

↓

Canonical

for

200 random rows.

---

# Phase 4 : Embedding Generation

Input

canonical_name

Model

Qwen Embedding

Parameters

normalize_embeddings=True

Store

embeddings.npy

Validation

Check

shape

Example

(349413,4096)

Verify

L2 Norm == 1

---

# Phase 5 : FAISS Index

Create

IndexFlatIP

dimension = embeddings.shape[1]

Add embeddings

Search

Top 100 neighbors

Store

scores

neighbors

Validation

Randomly inspect

50 neighbors

Ensure

Self similarity

≈1

---

# Phase 6 : Candidate Pair Generation

Generate candidate pairs

Ignore

self-match

Ignore

duplicate edges

Output

candidate_pairs

Example

Owner A

↓

Top100

Owner B

Owner C

Owner D

Validation

Count

Candidate pairs

Check

No duplicate

(i,j)

and

(j,i)

---

# Phase 7 : Feature Engineering

For every candidate pair calculate

1.

Cosine Similarity

2.

RapidFuzz Ratio

3.

Token Sort Ratio

4.

Token Set Ratio

5.

Levenshtein Distance

6.

Jaccard Similarity

7.

Common Token Count

8.

Prefix Match

9.

Business Keyword Match

Store

feature table

Example

| owner1 | owner2 | cosine | token_set | rapidfuzz | levenshtein | jaccard |

Validation

Inspect

500 pairs manually.

---

# Phase 8 : Rule Engine

DO NOT merge using cosine alone.

Example rules

Rule 1

IF

Cosine >= 0.97

AND

Token Set >=95

AND

RapidFuzz >=90

Merge

Rule 2

IF

Levenshtein <=2

AND

Cosine >=0.94

Merge

Rule 3

Reject

If only business words match.

Example

SAI DHAM TOURS

SHREE SAI TOURS

Should NOT merge.

Validation

Randomly inspect

1000 accepted

1000 rejected

pairs.

Tune thresholds.

---

# Phase 9 : Union Find

Merge only

approved pairs.

Output

cluster_id

Validation

Cluster size distribution

Largest clusters

Singleton count

Average cluster size

Inspect clusters

with

> 20 members.

---

# Phase 10 : Canonical Name Selection

For every cluster

Choose canonical owner

Options

Longest Name

Most Frequent Name

Official Registration Name

Store

cluster_id

canonical_owner

cluster_size

Validation

Inspect

largest

100 clusters.

---

# Phase 11 : Evaluation

Create

Ground Truth Dataset

1000 manually labeled pairs

same

different

Metrics

Precision

Recall

F1 Score

False Positive Rate

False Negative Rate

Goal

Precision

> 98%

Recall

> 95%

---

# Phase 12 : Incremental Pipeline

For every new owner

Cleaning

↓

Canonical

↓

Embedding

↓

Search Existing FAISS

↓

Candidate Features

↓

Rule Engine

↓

Existing Cluster

OR

New Cluster

No need to regenerate all embeddings.

Just create a notebook with each step in each cell with comment, also if step is huge then you can use multiple cells with comment like step 1.1 , step 1.2 for complete new step step 2 and step name.
