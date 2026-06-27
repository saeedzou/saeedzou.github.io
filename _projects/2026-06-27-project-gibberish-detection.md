---
title: "Gibberish Speech Detection via Phonetic vs Acoustic Representations"
date: 2026-06-27
tags: [speech-processing, zipa, wav2vec2, gibberish-detection]
---

## Overview

This project studies **gibberish speech detection** using two representation families:

- phoneme-based representations (ZIPA CR-CTC model)
- self-supervised acoustic representations (wav2vec2)

The goal is to evaluate whether gibberish detection is better modeled in phonetic space or in learned acoustic embedding space.

We use the EARS dataset and the EARS-WHAM Gibberish Test set.

---

## Dataset

We use:

- **EARS dataset** (clean speech, multi-speaker recordings) [2]

- **EARS-WHAM Gibberish Test set** [3], introduced in:  
  *Are These Even Words? (2025)* [1]

### Splitting strategy

- One speaker is held out entirely as test set  (p103)
- Remaining data split into:
  - 90% training
  - 10% validation  

---

## Methods

We evaluate two feature extraction pipelines:

### ZIPA [5] features
- phoneme posterior sequences from ZIPA CR-CTC model [4]
- sequence models applied on top:
  - mean pooling
  - attention pooling

### wav2vec2 features
- self-supervised contextual embeddings from wav2vec2 [6]
- same sequence models applied:
  - mean pooling
  - attention pooling

### Classifier architecture (both settings)

- linear projection layer
- bi-directional GRU
- pooling (mean or attention)
- MLP classification head
- binary cross entropy loss
- evaluation metric: ROC-AUC


### PR + LLM features

- phoneme sequences extracted using ZIPA CR-CTC model [4]
- sequences treated as text input
- perplexity computed using Qwen3.5-4B [7]

This setup follows ASR + LLM-style scoring from prior work [1], but operates at the phoneme level instead of word/subword transcripts.

---

## Results

The following results report:

- first value = validation AUC  
- second value = test AUC (computed using best validation checkpoint)

### ZIPA features

- mean pooling: **0.874 / 0.827**
- attention pooling: **0.945 / 0.890**

### wav2vec2 features

- mean pooling: **1.000 / 1.000**
- attention pooling: **1.000 / 1.000**

### PR + LLM features

Unlike ASR + LLM results reported in [1], PR + LLM shows weaker separability between gibberish and clean speech.

![PR + LLM](files/pr+llm.png)
![ASR + LLM](files/asr+llm.png)
---

## Discussion

The results show a clear difference between phoneme-based and acoustic self-supervised representations:

- ZIPA features provide strong but imperfect separability
- wav2vec2 features achieve perfect separation under both pooling strategies

This suggests that:

- phoneme-level representations still retain useful signal for distinguishing gibberish vs clean speech
- however, self-supervised acoustic embeddings capture stronger discriminative structure for this task

A key observation is that attention pooling improves ZIPA performance significantly, but does not change the overall ranking between feature types.

PR + LLM further supports this trend: phoneme-level transcription is less suitable for LLM-based scoring compared to word/subword ASR pipelines in [1].

---

## Relation to Prior Work

[1] argues that combining ASR outputs with language models improves gibberish detection.

https://arxiv.org/pdf/2510.21317

In contrast, this work finds:

- phoneme-level modeling (ZIPA) is sufficient for strong but not perfect separation
- acoustic SSL embeddings (wav2vec2) yield near-perfect separability in this dataset setup

This does not contradict prior work, but suggests:

- word/subword-level linguistic structure (used in ASR+LLM pipelines) may be more informative than phoneme-only representations
- SSL embeddings may also capture non-linguistic artifacts correlated with the dataset labels

---

## References

[1] Are These Even Words? Quantifying the Gibberishness of Generative Speech Models (2025). https://arxiv.org/pdf/2510.21317

[2] EARS Dataset. https://sp-uhh.github.io/ears_dataset/

[3] EARS-WHAM Gibberish Test set. https://www.inf.uni-hamburg.de/en/inst/ab/sp/publications/icassp2026-gibberish

[4] anyspeech/zipa-small-crctc-500k.https://huggingface.co/anyspeech/zipa-large-crctc-500k

[5] ZIPA: A family of efficient models for multilingual phone recognition. https://arxiv.org/abs/2505.23170

[6] wav2vec 2.0: A Framework for Self-Supervised Learning of Speech Representations. https://arxiv.org/abs/2006.11477