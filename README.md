# PDF Question Answering Automation with n8n

A two-stage document question-answering system built with **n8n**, **Supabase pgvector**, **Hugging Face embeddings**, and a **Groq-hosted Qwen model**.

The system converts a PDF into searchable semantic chunks and allows users to ask questions in plain English. Answers are generated from the retrieved document context rather than from the model's general knowledge.

## Overview

This project contains two independent n8n workflows:

1. **Document Ingestion**
   - Reads a PDF from disk
   - Extracts and cleans the text
   - Splits the document into overlapping chunks
   - Generates vector embeddings
   - Stores the chunks, metadata, and embeddings in Supabase

2. **Conversational Retrieval**
   - Receives a question through the n8n chat interface
   - Searches Supabase for the most relevant document chunks
   - Passes the retrieved context to a Groq-hosted Qwen model
   - Generates a concise, document-grounded answer
   - Maintains conversation history for follow-up questions

## Architecture

```text
Flow 1: Document Ingestion

PDF
  ↓
Read File from Disk
  ↓
Extract Text from PDF
  ↓
Clean and Chunk Text with JavaScript
  ↓
Generate Hugging Face Embeddings
  ↓
Store Content + Metadata + Embeddings in Supabase pgvector
```

```text
Flow 2: Conversational Retrieval

User Question
  ↓
n8n Chat Trigger
  ↓
AI Agent
  ↓
Question Embedding
  ↓
Supabase Vector Similarity Search
  ↓
Top Relevant Chunks
  ↓
Groq-hosted Qwen Model
  ↓
Grounded Answer + Conversation Memory
```

## Key Features

- Two-stage RAG architecture
- PDF text extraction and cleaning
- Custom JavaScript preprocessing
- 1,000-character chunks with 200-character overlap
- 768-dimensional Hugging Face embeddings
- Semantic search using Supabase and pgvector
- Top-4 relevant chunk retrieval
- Groq-hosted Qwen model for answer generation
- Conversation memory for contextual follow-up questions
- Grounded responses based on the indexed document

## Tech Stack

- **Automation:** n8n
- **Vector Database:** Supabase with pgvector
- **Embeddings:** Hugging Face Inference API
- **Embedding Model:** `sentence-transformers/distilbert-base-nli-mean-tokens`
- **LLM:** Qwen hosted through Groq
- **Custom Logic:** JavaScript
- **Runtime:** Docker

## Workflow 1: Document Ingestion

The ingestion workflow prepares the PDF for semantic retrieval.

### Steps

1. A manual trigger starts the workflow.
2. The PDF is read from a folder mounted into the n8n Docker container.
3. The PDF extraction node converts the binary file into raw text.
4. A JavaScript node:
   - Removes PDF metadata and formatting noise
   - Validates that usable text was extracted
   - Splits the text into 1,000-character chunks
   - Adds a 200-character overlap between neighboring chunks
   - Adds metadata such as the source name and chunk index
5. The Hugging Face model converts each chunk into a 768-dimensional embedding.
6. The Supabase Vector Store inserts the following for every chunk:
   - Original text
   - Metadata
   - Embedding vector

At the end of this flow, the PDF has been transformed into a searchable semantic index.

## Workflow 2: Conversational Retrieval

The retrieval workflow runs whenever a user asks a question.

### Steps

1. The n8n Chat Trigger receives the user's message.
2. The AI Agent decides when to use the Supabase retrieval tool.
3. The user's question is converted into an embedding using the same Hugging Face model.
4. Supabase compares the question embedding with the stored document embeddings.
5. The four most relevant chunks are returned.
6. The retrieved chunks and the original question are passed to the Groq-hosted Qwen model.
7. The model generates a concise answer grounded in the retrieved context.
8. Simple Memory stores recent chat history so the user can ask follow-up questions.

## Example

### Input

```json
{
  "question": "Why does the Transformer use positional encoding?"
}
```

### Example Output

```json
{
  "answer": "The Transformer uses positional encoding because it has no recurrence or convolution, so it needs an explicit way to represent the order of tokens in the sequence."
}
```
