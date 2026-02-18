RETRIEVAL AUGMENTED GENERATION (RAG) SYSTEM:

python -m venv venv
venv\Scripts\activate

pip install langchain langchain-community langchain_text_splitters langchain_openai langchain_chroma python_dotenv langchain_experimental langchain_core

| Model                         | Speed     | Quality   | Cost |
| ----------------------------- | --------- | --------- | ---- |
| OpenAI text-embedding-3-small | Fast      | Excellent | Paid |
| all-MiniLM-L6-v2              | Very fast | Good      | Free |
| all-mpnet-base-v2             | Medium    | Very Good | Free |

Delete the virtual environment and recreate it.

deactivate
rm -r venv
rm -r db/chroma_db

python -m venv venv
venv\Scripts\activate

pip install langchain==1.2.10
pip install langchain-community==0.4.1
pip install langchain-chroma==1.1.0
pip install langchain-text-splitters==1.1.0
pip install chromadb==1.1.0
pip install sentence-transformers==2.7.0

---

1_ingestion_pipeline.py

Source docs (10M tokens) --> Chunking (5K tokens) --> further split the chunks (2000 chunks of each) --> Embedding (Using Open AI) --> output as 2000 vector embeddings --> Vector DB

# Check if docs directory exists

# Load all .txt files from the docs directory

# Create ChromaDB vector store

# Define paths

# Check if vector store already exists

# Step 1: Load documents

# Step 2: Split into chunks

# Step 3: Create vector store

High-Level Architecture

Raw Documents (.txt)
↓
Load into LangChain Documents
↓
Split into Chunks
↓
Generate Embeddings (OpenAI)
↓
Store in Chroma Vector DB (Persistent)

---

2_retrieval_pipeline.py [User query will get embedded and the retriever will goes off and fetches the chunks]

User query --> converts to Embedding (Using Open AI) --> Retriever --> Chunks --> Output context

# Load embeddings and vector store

# Search for relevant documents

# Display results

used COSINE algorithm to search and compare the data in the DB. FORMULA: cosine_similarity = (A . B) / (||A|| \* ||B||)
-- (A . B) dot product of vectors A and B
-- ||A|| magnitude(length) of vector A
-- ||B|| magnitude(length) of vector B

High-Level Architecture

User Query
↓
Embed Query
↓
Vector Similarity Search (Chroma / HNSW / Cosine)
↓
Top-K Relevant Chunks

---

3_answer_generation.py

# Load embeddings and vector store

# Search for relevant documents

# Combine the query and the relevant document contents

# Create a ChatOpenAI model

# Define the messages for the model

# Invoke the model with the combined input

# Display the full result and content only

High-Level Architecture

User Query
↓
Vector Retrieval (Chroma + HuggingFace Embeddings)
↓
Top-K Relevant Chunks
↓
Prompt Construction
↓
LLM Ollama (llama2)
↓
Final Answer

from langchain_ollama import ChatOllama

model = ChatOllama(
model="llama2-7b", # e.g. l2, l2-7b, etc
temperature=0.0,
max_length=512
)

---

4_history_aware_generation.py

# Load environment variables

# Connect to your document database

# Set up AI model

# Store our conversation as messages

# Step 1: Make the question clear using conversation history

# Ask AI to make the question standalone

# Step 2: Find relevant documents

# Show first 2 lines of each document

# Step 3: Create final prompt

# Step 4: Get the answer

# Step 5: Remember this conversation

# Simple chat loop

User Question
↓
Conversation Rewriting (if history exists)
↓
Vector Retrieval (Chroma + HuggingFace Embeddings)
↓
Context Injection
↓
Local LLM (Ollama - llama2)
↓
Answer
↓
Stored in chat history

---

RAG-CHUNKING STRATEGIES

RAG Ingestion Pipeline:

- Document Loading (PDF, DOCX, TXT → text)
- Text Chunking (long text → smaller pieces)
  Embedding (text chunks → vectors)
- Storage (vectors → vector database)

Chunking is the critical second step - it determines how your content gets divided for retrieval.
Your RAG system doesn't search entire documents. It searches chunks. So the final answer generation quality depends
on those chunks.
Bad chunking breaks everything downstream - even perfect embeddings can't fix poorly split content.

THERE ARE PROBLEMS WITH characterTextSplitter -- it just cuts text at fixed character counts. So the context gets lost across chunks and it ends up with Poor retrieval quality.

The 5 Chunking Strategies

1. CharacterTextSplitter (Beyond basic chunk_size)
   • Custom separators (split on specific patterns)
   • Still useful for simple, uniform documents or when speed matters most

2. RecursiveCharacter TextSplitter (Upgrade from CharacterTextSplitter)
   Tries to split at natural boundaries (paragraphs, sentences, words)
   • Falls back gracefully if chunks too big
   • Preserves more context than basic splitting

3. Document-Specific Splitting (Respects document structure)
   • PDF: Splits by pages, sections, headers
   • Markdown: Splits by headers, code blocks, lists
   • Each document type gets appropriate treatment

4. Semantic Splitting (Content-aware boundaries)
   • Uses embeddings to detect topic shifts
   • Keeps related concepts together
   Splits when meaning changes, not just by size
   • More intelligent but computationally expensive

5. Agentic Splitting (AI-powered chunking)
   • LLM analyzes content and decides optimal splits
   • Can understand complex relationships
   • Adapts to content type automatically
   Most sophisticated but slowest/most expensive

---

1> CharacterTextSplitter:

CharacterTextSplitter doesn't just cut at character limits. It follows a split-first, merge-second approach.

1. Split: Break text at separators (default: \n\n)
2. Merge: Combine pieces until hitting chunk_size limit

splitter1 = CharacterTextSplitter(
chunk_size=100,
chunk_overlap=0,
separator="\n\n" # ["\n\n", "\n", ". ", " ", ""] -- we can pass only one separator here
)

---

2> RecursiveCharacterTextSplitter:

recursive splitter = Recursive CharacterTextSplitter
separators=["\n\n", "\n", ". ", "", ""], # Multiple separators can be passed here
chunk_size=100,
chunk_overlap=0

---

3> SemanticChunker

Semantic chunking breaks up long documents into meaningful pieces by finding where topics naturally change.
Instead of cutting at random word counts, it uses AI embeddings to understand the semantic meaning of
sentences.
If one sentence talks about a certain topic and the next sentence talks about an entire different topic, then it means we
need to chunk

3-Step Process:

1. Encode: Convert each sentence into embeddings (numerical vectors)
2. Compare: Calculate similarity score between nearby sentences
3. Split: Create boundaries where similarity drops significantly

---

4> Agentic chunking

# Initialize the LLM

# Create the prompt

# Get AI response

# Clean up the chunks (remove extra whitespace)

# Show results

---

# Stage 1 — Indexing (Done Once)

Step 1: Load Documents
Raw .txt files → LangChain Document objects.

Step 2: Split into Chunks
Because embeddings work best on smaller semantic units.

Step 3: Generate Embeddings
Each chunk is converted into a vector.

Step 4: Store in Chroma
Chroma stores: Vector + Original text + Metadata

# Stage 2 — Retrieval (Per User Query)

Now user asks:
"How much did Microsoft pay to acquire GitHub?"

Step 1 — Conversation Rewriting (Optional)
If history exists, then rewrite to improves search accuracy

Step 2 — Query Embedding
The rewritten query is embedded as Query → 384-dim vector using the SAME HuggingFace model.

Step 3 — Similarity Search
Chroma computes cosine similarity between Query vector VS All stored chunk vectors and returns top-k most similar chunks.

# Stage 3 — Why We Need an LLM (Ollama)

We have User Question + Relevant Document Chunks But the user expects: "$7.5 billion" Not A 900-character paragraph.

The LLM Does 3 Critical Things: Reading Comprehension, Information Extraction, Answer Synthesis

              (Indexing Phase — one time)

Documents
↓
Split into chunks
↓
Generate embeddings (MiniLM)
↓
Store vectors in Chroma

---

              (Runtime Phase — per query)

User Question
↓
Conversation Rewriting (LLM)
↓
Embed Query
↓
Cosine Similarity Search (Chroma)
↓
Top-k Relevant Chunks
↓
LLM (Ollama)
↓
Final Answer
↓
Store in chat history
