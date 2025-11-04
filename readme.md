Dendrum is a high-performance, local-first RAG (Retrieval-Augmented Generation) system. It allows you to ingest documents into a local vector database and use a web interface to search and chat with your data.

# Features
⚡ High-Performance Backend: The darkice API is built in Rust using the axum framework for high throughput and low memory usage.

⚡ Modern Frontend: A responsive and easy-to-use web interface built with Nuxt.

⚡ Local-First Database: Built on DuckDB for fast, local vector search and analytics, using an HNSW index for embeddings.

⚡ Flexible RAG Pipeline: Natively supports connecting to remote LLMs (like OpenAI) or local models (via Ollama) to power your chat.

⚡ Document Ingestion: Upload files (PDFs, text, etc.) to be processed, chunked, embedded, and added to your knowledge base.

⚡ Asynchronous Data Pipeline: Features a background merger service that processes ingested data from Parquet files into the main database. This ensures the API remains fast and responsive during heavy ingestion.

⚡ System Dashboard: A statistics page to monitor the number of documents and data chunks in your database.

# 🏁 Quick Start: Running with Docker Compose
This project is distributed as a set of Docker containers. The easiest way to run the entire stack is with docker-compose.

## Prerequisites
Docker
docker-compose 

## Step 1: Create Your Project Files
Create a new directory for your Dendrum server, (e.g., my-dendrum-server), and cd into it.


```
mkdir my-dendrum-server
cd my-dendrum-server
```