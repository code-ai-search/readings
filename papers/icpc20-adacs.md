# Adaptive Deep Code Search (AdaCS) — Ling et al., ICPC 2020

- Title: Adaptive Deep Code Search
- Authors: Chunyang Ling, Zeqi Lin, Yanzhen Zou, Bing Xie
- Venue: ICPC 2020
- PDF: ICPC20-AdaCS-perprint.pdf (in this repo)

## TL;DR
AdaCS separates domain-specific lexical learning (unsupervised fastText embeddings per project) from cross-project syntactic matching (supervised RNN on lexical interaction matrices). This design makes a single trained matching model transferable to new codebases by only re-training per-project embeddings — improving robustness to OOV domain identifiers.

## What they propose
1. Learn per-project word embeddings (fastText) from project corpus (code, comments, docs).
2. For each (query q, code c) produce an interaction / lexical matching matrix of cosine similarities between q- and c-word embeddings.
3. Convert each column (code token) into a vector comprised of the column (padded to fixed length) + IDF(token).
4. Feed the sequence of code-token vectors into a 2-layer LSTM and map final state to a matching score f(q,c).
5. Train supervisedly on many GitHub text–code pairs (pairwise hinge loss with negative sampling).
6. At transfer time: only re-compute per-project fastText embeddings and matching matrices — reuse the pretrained matching model.

## Key experimental setup
- Training data: 77,920 GitHub text–code positive pairs (first javadoc sentence as query); 20 negative samples per positive (→ triples).
- Test (adaptive): 1,606 pairs from projects excluded from training: Apache Lucene, Apache POI, JFreeChart.
- Model: 2-layer LSTM (hidden dim 64), dropout 5%; fastText for embeddings.
- Training: ADAM lr 0.005, batch size 64; negative samples per positive = 20.
- Efficiency: per (q,c) prediction ≈ 1.29 ms (CPU+GPU environment used by authors).

## Main results (adaptive setting — model not trained on target projects)
- AdaCS MRR = 0.621; Hit@5 = 0.772
- Best baseline CodeHow: Hit@5 = 0.567, MRR ≈ 0.479
- DeepCS / BVAE degrade heavily under domain shift (OOV).

## Ablation / variants
- AdaCS-IND (indicator exact-match instead of embedding cosine): lower performance → unsupervised embeddings matter.
- AdaCS-noIDF: lower performance → IDF dimension helps.

## Strengths
- Designed for cross-project transfer; addresses OOV domain identifiers.
- Requires no labeled target data — only project corpus for fastText.
- Uses interaction matrix approach -> learns matching patterns (like “for each” patterns) that generalize across domains.
- Good empirical gains on adaptive code search; competitive in in-domain setting.

## Limitations
- Supervised training is still computationally heavy; training iterations ~30 min each in their setup.
- Requires a reasonable project corpus for reliable fastText embeddings.
- Choice of LSTM: sequential LSTMs may be slower than more-parallel encoders (authors mention exploring Transformers as future work).
- Evaluation scope: tested on Java only; target projects are popular open-source projects (not necessarily large closed-enterprise systems).
- The interaction matrix relies on tokenization decisions and a query-length cap (N).

## Reproducibility checklist (to replicate)
- Data: extract Java methods and first javadoc sentence as query (exclude methods without javadoc). Filter as paper describes.
- Embeddings: train fastText per project on code+comments+docs (use default fastText params or tune).
- Preprocessing: tokenization (split identifiers, lowercasing, filter punctuation), IDF compute per corpus.
- Model: 2-layer LSTM (hidden=64), dropout=0.05, ADAM lr=0.005, batch=64.
- Training: pairwise hinge loss; negative samples per positive = 20; early stop on validation.
- Hardware: they used GTX1080Ti; training timing will vary by hardware.
- Code: authors indicated code and experiments are on GitHub; check authors’ repo or contact authors if needed.

## Suggested discussion / next-experiment questions
- What are the best practices for building the project-specific corpus when comments are scarce? (e.g., incorporate issues, PR descriptions, design docs, StackOverflow.)
- Would a Transformer-based encoder on the interaction matrix (or on sequences of match features) improve accuracy and speed?
- How sensitive are results to the query-length cap N, embedding dimensions, and negative-sampling size?
- How well does AdaCS perform on other languages (Python, JavaScript, C#) or mixed-language repositories?
- Can domain adaptation be further improved by unsupervised adversarial alignment between GitHub embedding space and target-project embedding space (rather than re-training fastText)?
- Could you combine AdaCS with a neural reranker that uses AST/structural features (not just token similarity) to further improve precision?

## Suggested citation
Chunyang Ling, Zeqi Lin, Yanzhen Zou, Bing Xie. Adaptive Deep Code Search. ICPC 2020.
