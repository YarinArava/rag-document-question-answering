# Retrieval-Augmented Generation (RAG) – Document Question Answering

A Retrieval-Augmented Generation (RAG) project that combines semantic retrieval with large language models to answer questions using only information retrieved from a given knowledge base.

The project was initially developed on *Winnie-the-Pooh* and later extended to an external food recipes dataset to test the same RAG pipeline on a different domain.

---

## Project Overview

The goal of this project was to build a complete RAG pipeline from scratch and explore how different preprocessing, chunking, embedding, retrieval, and generation choices affect the quality of the final answers.

The pipeline consists of:

1. Text preprocessing
2. Document chunking
3. Embedding generation
4. Semantic retrieval
5. Context construction
6. LLM-based answer generation
7. Evaluation of different RAG configurations

The generated answers are constrained to the retrieved context.  
If the answer cannot be found in the provided context, the model is instructed to respond with:

> "I don't know."

This helps reduce hallucinations and keeps the generated answers grounded in the source material.

---

## Knowledge Base

The first experiment uses the Project Gutenberg version of:

**Winnie-the-Pooh – A. A. Milne**

The original text is cleaned before being used as the RAG knowledge base.

### Text Preprocessing

The preprocessing pipeline includes:

- Removal of Project Gutenberg headers and license sections
- Removal of unnecessary formatting characters
- Whitespace normalization
- Removal of excessive blank lines
- Preservation of paragraph structure
- Cleaning without changing the semantic content of the story

---

## Chunking Strategy

Several chunk sizes were evaluated:

- 250 words
- 200 words
- 150 words
- 100 words

Both overlapping and non-overlapping strategies were tested.

The most balanced configuration was:

- **Target chunk size:** 100 words
- **Overlap:** 20 words

Larger chunks preserved more context but often introduced unrelated information, while smaller chunks provided more focused retrieval.

Adding overlap helped preserve information located near chunk boundaries.

---

## Embedding Models

Several embedding approaches were compared.

### Sentence Transformers

- `sentence-transformers/all-MiniLM-L6-v2`
  - 384-dimensional embeddings

- `sentence-transformers/all-mpnet-base-v2`
  - 768-dimensional embeddings

### Cohere

- `embed-v4.0`
  - 1536-dimensional embeddings
  - Used for semantic search and retrieval

The experiments demonstrated that similarity scores alone are not sufficient for evaluating retrieval quality. The relevance of the retrieved context and the quality of the final generated answer are more important.

---

## Semantic Retrieval

Each document chunk is converted into a dense vector representation.

For a user query:

1. The query is embedded using the same embedding model.
2. Similarity is calculated between the query and all document chunks.
3. The most relevant chunks are ranked.
4. The top-K chunks are passed to the language model as context.

Different retrieval depths were evaluated:

- `K = 1`
- `K = 3`
- `K = 5`

In many experiments, **K=1 or K=3 produced more focused answers**, while retrieving too many chunks sometimes introduced irrelevant information.

---

## Answer Generation

Two generation approaches were explored.

### Hugging Face

- `TinyLlama/TinyLlama-1.1B-Chat-v1.0`

TinyLlama was used to test a fully local RAG pipeline using Sentence Transformer embeddings.

### Cohere

- **Command A+ (05-2026)**

Cohere was used for the final RAG experiments together with its `embed-v4.0` embedding model.

The prompt explicitly instructs the model to:

- Use only the retrieved context
- Avoid inventing information
- Respond with `"I don't know"` when the answer is unavailable

---

## RAG Pipeline

```text
Document
   ↓
Text Cleaning
   ↓
Chunking + Overlap
   ↓
Embedding Generation
   ↓
Vector Representations
   ↓
User Question
   ↓
Query Embedding
   ↓
Similarity Search
   ↓
Top-K Relevant Chunks
   ↓
Context Construction
   ↓
LLM
   ↓
Grounded Answer
