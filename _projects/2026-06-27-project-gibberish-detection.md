---
title: "Gibberish Speech Detection via Phonetic vs Acoustic Representations"
date: 2026-06-27
tags: [speech-processing, gibberish-detection, ssl, asr, wav2vec2, zipa, research]
---

## Overview

This project investigates the problem of **gibberish speech detection** using multiple speech representation paradigms. The central question is whether gibberish is primarily detectable through:

- phonetic structure (ASR / phoneme models), or  
- higher-level acoustic or linguistic representations (self-supervised learning models, LLM scoring).

We evaluate this using the EARS dataset family and the EARS-WHAM Gibberish Test set, combined with modern speech and language models.

---

## Dataset

### Sources

- **EARS dataset** (clean speech, multi-speaker recordings)  
  

- **EARS-WHAM Gibberish Test set**, introduced in:  
  *Are These Even Words? Quantifying the Gibberishness of Generative Speech Models* (2025)

### Construction Strategy

To avoid speaker leakage:

- One speaker is held out entirely as **test set**
- Remaining speakers are split into:
  - 90% training
  - 10% validation

Additionally, a subset of speakers appears in both clean and gibberish conditions, reducing the risk that models rely purely on speaker identity.

---

## Methods

### 1. ZIPA Phoneme-Based Representation

We use the **ZIPA CR-CTC phoneme recognition model** to convert speech into phoneme distributions.

- Input: log-mel filterbanks
- Output: phoneme posterior sequences

Two modeling approaches were tested:
- handcrafted statistical features over phoneme posteriors
- GRU-based sequence modeling

---

### 2. wav2vec2 Self-Supervised Features

We extract frame-level embeddings from wav2vec2:

- Input: raw waveform
- Output: contextual acoustic embeddings
- Model trained via GRU + attention pooling classifier

---

### 3. Sequence Classifier Architecture

Across experiments, a shared architecture is used:

- Linear projection layer
- Bi-directional GRU
- Attention pooling (replacing mean pooling in later experiments)
- MLP classification head

Loss: binary cross-entropy  
Metric: ROC-AUC

---

### 4. Phoneme-Language Model Scoring

We also evaluate a hybrid pipeline:

1. Decode speech using ZIPA
2. Feed phoneme sequences into **Qwen3.5-4B**
3. Compute perplexity over phoneme sequences
4. Compare distributions between clean vs gibberish speech

---

## Results

### 1. Logistic Regression on Handcrafted ZIPA Features

- Cross-validation AUC: ~0.60 ± 0.08  
- Best observed AUC: **0.83**

---

### 2. GRU on ZIPA Features

| Model Variant | Best Validation AUC |
|--------------|---------------------|
| Mean pooling | ~0.81–0.83 |
| Attention pooling | **0.869** |

---

### 3. GRU on wav2vec2 Features

| Model | Validation AUC | Test AUC |
|------|----------------|----------|
| wav2vec2 + GRU + attention | **1.000** | **1.000** |

---

### 4. ZIPA vs wav2vec2 Comparison

| Representation | Best AUC | Behavior |
|----------------|----------|----------|
| ZIPA phonemes | ~0.87 | Moderate separability |
| wav2vec2 embeddings | **1.00** | Perfect separability |

---

### 5. Phoneme-LM Perplexity (Qwen3.5-4B)

- Clean and gibberish distributions show partial overlap
- Weak separability compared to ASR+LLM results in:
  *Are These Even Words?* (2025)

---

## Discussion

### Phonetic vs Acoustic Signal

The experiments suggest:

- phoneme-level representations (ZIPA) provide limited but useful signal
- wav2vec2 embeddings provide very strong separability

This indicates that:
> gibberish detection is not purely a phonetic modeling problem.

However, the perfect performance of wav2vec2-based classifiers also suggests:
- possible dataset-specific acoustic artifacts
- or that SSL embeddings capture non-linguistic cues correlated with the labels

---

## Comparison to Prior Work

The paper *Are These Even Words?* (2025) argues that:

- ASR + LLM pipelines provide strong sensitivity to gibberish speech
- linguistic modeling improves detection performance

### Relation to this work

- This work evaluates **phoneme-level decoding (ZIPA)** instead of word-level ASR output
- We evaluate **phoneme perplexity using an LLM (Qwen3.5-4B)**

### Key observation

- Phoneme-level LLM scoring shows weak separability
- This differs from the stronger separability reported in ASR+LLM pipelines

### Interpretation

This does not strictly contradict prior work, but suggests:

- word/subword structure may be crucial for LLM-based separation
- phoneme-level abstraction may remove critical linguistic information used by LLMs

---

## Conclusion

This project evaluates gibberish detection across phonetic and acoustic representations.

Key findings:

- wav2vec2 embeddings achieve near-perfect classification performance
- ZIPA phoneme representations provide moderate separability
- phoneme-level LLM scoring does not reproduce strong ASR+LLM separability
- attention-based sequence modeling improves performance but does not close the gap

### Main Insight

> Gibberish detection appears to depend more on higher-level linguistic structure than phoneme-level representations, while acoustic SSL embeddings may exploit additional non-linguistic cues.

---

## Future Work

- Control for dataset artifacts leading to near-perfect wav2vec2 performance
- Direct comparison with word-level ASR + LLM pipelines
- Hierarchical phoneme-to-word modeling
- Cross-dataset generalization tests