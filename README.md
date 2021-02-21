# FuzzyString_Annoy

This is an experimental project to implement some form of Fuzzy String matching using Spotify's Annoy library by embedding strings into a high-dimensional space.

## Goal

Given a large corpus of strings, how do we find fuzzy close neighbors substantially faster than naive O(N) levenshtein distance scoring?

## (Broad) Approach

Extract english_alphabets from strings -> specify some transform into a vector representation -> use Annoy library to build index trees -> support fast search, slow indexing :)

## Dataset

Open-sourced Singapore NEA 'Licensed Eating Establishments' dataset from: https://data.gov.sg/dataset/list-of-nea-licensed-eating-establishments-with-grades-demerit-points-and-suspension-history. We are primarily interested in the 'Licensee Name' field.

# Some Initial exploration and results

1. Using Annoy index-tree approach to the k-NN problem once we **already have good vector embeddings** is an at least ~300x speedup over a naive O(N) cycle through the list of vectors.
2. This however, begs the question of what is a 'good vector embedding' for this approach to work.
3. Control test with Levenshtein distance did not fare too well due to query string having different length from search corpus.
4. Point (3) is somewhat mitigated when we control for this by cycling (wrapping around) strings to fit exactly `NAME_MAXLEN` size. (so 'abc' is repeated to 'abcabcabc...' until we reach the desired string length).
5. Experimented with naive approaches to the embedding by:
    - Transforming string into sequence of 3 n-grams, 2,4 skip-grams and summing the 'value' of each character ('a' is 0, 'b' is 1 etc...).
    - Using the [`chars2vec`](https://hackernoon.com/chars2vec-character-based-language-model-for-handling-real-world-texts-with-spelling-errors-and-a3e4053a147d) library which has pre-trained models for embedding english words into 50,100,150...-dimensional space
        - Using this approach, we appear to have similar issues with controlling for string length and similar-sized strings as our search query appear higher up in the list of similarity results.
6. The original use-case of Annoy is to recommend music on Spotify. As such, we don't expect un-indexed vectors (we would run our indexing once we have a batch of new songs). So rather than searching to fix mis-spellings in our search query, it gives pretty good similarity results to other items already within our corpus. (See cell 18 results when we search for an item existing within our list).

## Recommendations

- Focus on the problem of **vector embedding** by exploring training a new model using the existing `chars2vec` architecture (described in the blog post linked).
- Since our search scales cheaply (single query through Annoy tree against size of items in the tree), we can index substrings in addition to the full strings in our corpus?
- An orthogonal exploration vector would be to look into the literature on Protein DNA matching.