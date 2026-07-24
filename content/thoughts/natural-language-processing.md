---
title: "Natural Language Processing"
date: 2026-07-24
tags:
  - nlp
  - machine-learning
  - learning
publish: false
---

# Natural Language Processing

Translate natural human language into a form a computer can understand.

## Ambiguity

### Lexical

- A word having multiple meanings

### Syntactic

- "I saw a man with a binocular" has two interpretations: seeing a man using binoculars, or seeing a man who has binoculars

### Semantic

- "Everyone should read a book"

### Pragmatic

- "Can you open the door" — ability vs request

## Word-level analysis

### Tokenization

Breaking a stream of text into smaller units called **tokens** (words, characters, or sub-words).

### Stemming

A heuristic, rule-based process that chops off word endings to reach a common stem. Does not always yield a valid dictionary word. Removes suffixes/prefixes such as `-ing`.

### Lemmatization

Uses vocabulary and morphological analysis to return the base/dictionary form of a word (the lemma). Dictionary- and grammar-based.

## Bag of Words

Converts text into numerical vectors by treating a document as a "bag" of words — ignores grammar, order, and context; focuses on word frequency.

## TF-IDF

**Term Frequency–Inverse Document Frequency** — a statistical measure of how important a word is to a document in a corpus.

## Word embedding

Maps words to dense vectors in a multi-dimensional space so semantic meaning and relationships are captured from context, unlike isolated ID-based representations.

## Part-of-speech (POS) tagging

Assigns a grammatical category to each word (noun, verb, adjective, etc.). Critical preprocessing for syntactic analysis and semantic understanding.

### Rule-based tagging

Hand-written linguistic rules (e.g. "if a word follows 'the', it is likely a noun"). Precise in simple settings; hard to maintain for complex language.

### Stochastic (probabilistic) tagging

Uses frequency and probability for the most likely tag.

- **Hidden Markov Models (HMM):** Probability of a tag sequence given observed words — transition probability (tag→tag) and emission probability (word→tag).

### Deep learning tagging

Neural architectures such as **RNNs**, **LSTMs**, and **Transformers (BERT, RoBERTa)**. Treat POS tagging as sequence labeling and capture dependencies across the sentence.

## Related

- [[machine-learning]]
