Retrieval Augmented Generation (RAG) System

This project implements a complete Retrieval-Augmented Generation (RAG) pipeline using LangChain, ChromaDB, OpenAI/HuggingFace embeddings, and Ollama (LLaMA2).

The system is divided into three main stages:

🔹 Stage 1 — Indexing (One-Time Setup)

Transforms raw documents into searchable vector embeddings.

Pipeline Flow

Raw Documents (.txt)
↓
Load as LangChain Documents
↓
Split into Chunks
↓
Generate Embeddings
↓
Store in Chroma Vector Database

Steps

Load Documents

Reads .txt files from the docs/ directory.

Chunking

Splits large documents into smaller semantic chunks.

Chunking strategy directly impacts retrieval quality.

Embedding Generation

Converts text chunks into vector representations.

Supported models:

text-embedding-3-small (OpenAI – Paid)

all-MiniLM-L6-v2 (Free, Fast)

all-mpnet-base-v2 (Free, Higher quality)

Storage

Stores vectors, text, and metadata in ChromaDB (persistent).

🔹 Stage 2 — Retrieval (Per User Query)

Retrieves relevant document chunks using vector similarity search.

Flow

User Query
↓
Embed Query
↓
Cosine Similarity Search (Chroma / HNSW)
↓
Top-K Relevant Chunks

Similarity Formula
cosine_similarity = (A · B) / (||A|| * ||B||)


A · B → Dot product

||A|| → Vector magnitude

Used to compare query vector with stored chunk vectors.

Optional:

Conversation rewriting (history-aware search)

🔹 Stage 3 — Answer Generation (LLM)

The LLM transforms retrieved chunks into a precise, human-friendly answer.

Flow

User Query
↓
Vector Retrieval
↓
Context Injection
↓
LLM (Ollama – llama2)
↓
Final Answer
↓
Stored in Chat History

Why LLM Is Needed

Even after retrieval, users expect:

Precise answers

Extracted facts

Synthesized responses

The LLM performs:

Reading comprehension

Information extraction

Answer synthesis

🔹 History-Aware Generation

Enhances multi-turn conversations:

Rewrites follow-up questions into standalone queries.

Retrieves relevant documents.

Generates final response.

Stores conversation history.

Chunking Strategies (Critical for RAG Quality)

Chunking determines retrieval performance. Poor chunking leads to poor answers.

1️⃣ CharacterTextSplitter

Splits by fixed separators

Fast but may lose semantic context

2️⃣ RecursiveCharacterTextSplitter

Splits using multiple fallback separators

Preserves structure better

3️⃣ Document-Specific Splitting

Respects document structure (PDF, Markdown, etc.)

4️⃣ Semantic Chunking

Uses embeddings to detect topic shifts

Splits where meaning changes

More intelligent, higher compute cost

5️⃣ Agentic Chunking

LLM decides optimal chunk boundaries

Most advanced and expensive approach

System Architecture Overview
Indexing Phase (One-Time)

Documents
↓
Chunking
↓
Embedding Generation
↓
Chroma Storage

Runtime Phase (Per Query)

User Question
↓
Conversation Rewriting (Optional)
↓
Query Embedding
↓
Cosine Similarity Search
↓
Top-K Chunks
↓
LLM (Ollama)
↓
Final Answer
↓
Chat History Storage

Tech Stack

LangChain

ChromaDB

HuggingFace Sentence Transformers

OpenAI Embeddings

Ollama (LLaMA2)

Python

This RAG system enables efficient document retrieval combined with powerful LLM-based answer generation, supporting both single-turn and history-aware conversations.
