---
layout: post
title:  Word embeddings compared
date:   2020-05-25 00:00:00 +0800
categories:
tags:
---

Comparisons of a few state-of-the-art word embedding models.

# GloVe

The main difference between GloVe and [word2vec](/2020/05/22/word2vec.html) is that, word2vec is designed to
predict a target word given its neighbouring words (skip-gram) or vice versa (cbow), 
whereas GloVe is designed to predict the possibilities that 2 words appear together
directly, by optimizing say `vec(A) • vec(B) = log(no_of_co_occurance_times)`.

Looking from another angle, word2vec (skip-gram for example) attemps to capture 
co-occurance information window by window with a selected window size. GloVe 
creates a co-occurance matrix for the entire corpus in the beginning, and 
optimize the overall statistics base on how oftern they appears.

# fastText

fastText is a extension to the skip-gram model in word2vec. It uses **subword**
embedding by using n-grams corpus. For example, a word "airplane" generates the
following corpus with `n = 3`:

```
"<ai", "air", "irp", "rpl", "pla", "lan", "ane", "ne>", "<airplane>"
```

fastText works well for rare words. Due to its nature understanding prefixes and 
suffixes, fastText also works for words that are not seen during the training. 
This is a major advantage as compared to word2vec or GloVe as both models can 
only deal with words appeared in the training.

# BERT (Bidirectional Encoder Representations from Transformers)

Embeddings like GloVe and word2vec use one vector for each word, making them
context-independent. BERT is context-aware in a way that it generates different
word embeddings for a word based on its **position** in a sentence. For example,
“He went to the prison **cell** with his **cell** phone to extract blood **cell** 
samples from inmates”, BERT generates different vectors for the same word "cell".

BERT is trained with 2 tasks combined:

1. BERT masks out 15% of the words in the input, runs the entire sequence through 
a bidirectional Transformer encoder, and predicts the masked words. Example:

```
Input: the man went to the [MASK1] . he bought a [MASK2] of milk.
Labels: [MASK1] = store; [MASK2] = gallon
```

  Since Transformer reads the entire sequence at once as compared to 
  left-to-right or right-to-left sequential reading, it is actually more of a 
  "non-directional" model.

2. BERT also models relationships between sentences. BERT is pre-trained with 
another task of finding whether 2 sentences are next to each other. Example:

  ```
  Sentence A: the man went to the store.
  Sentence B: he bought a gallon of milk.
  Label: IsNextSentence

  Sentence A: the man went to the store.
  Sentence B: penguins are flightless.
  Label: NotNextSentence
  ```

The tasks of predicting the masked words and the next sentence are trained together
with a goal of minimizing the combined loss function.

# XLM

XLM is built on top of BERT with a goal of learning the relations between words
in different languages.

XLM differs from BERT in the following aspects:

1. it uses [Byte-Pair Encoding](https://en.wikipedia.org/wiki/Byte_pair_encoding) 
to split words into subwords, thereby increasing shared vocabulary in different 
languages.

2. each training sample contains the same sentence from 2 different languages. 
The context of one language is used to predict masked words from the other 
langauge.

3. The language IDs and positional encoding of each language are fed into the 
model, so that the model can learn the relationships of tokens in different 
languages.

The complete model trains both a "vanilla" BERT and a modified BERT (#2 and #3) 
by alternating between 2 of the choices [(reference)](https://towardsdatascience.com/xlm-enhancing-bert-for-cross-lingual-language-model-5aeed9e6f14b).
