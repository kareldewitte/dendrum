Dendrum is a high-performance, local-first RAG (Retrieval-Augmented Generation) system. It allows you to ingest documents into a local vector database and use a web interface to search and chat with your data.

# Features
⚡ High-Performance Backend: The darkice API is built in Rust using the axum framework for high throughput and low memory usage. Fully accessible via API. Build your own ingestion runners and send data to backend.

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

Inside this directory, create the following three files.

1. docker-compose.yml
This file defines and links all the services: the frontend, the backend API, and the merger service.

```YAML
version: '3.8'

services:
  # The main API backend (Rust)
  darkice-backend:
    image: DOCKER_IMAGE_NAME/dendrum-server:latest # 
    platform: linux/amd64
    hostname: dendrum-server
    restart: unless-stopped
    ports:
      - "8001:8001"
    volumes:
      # Links the shared data volume
      - dendrum-server:/data
      # Links your local config file into the container
      - ./processing_config.toml:/app/processing_config.toml:ro
    environment:
      - OPENAI_TOKEN=${OPENAI_TOKEN}
      - BIND_ADRESS=0.0.0.0:8001
      - SERVICE_MODE=node
      - RUST_LOG=info

  # The background merger service (Rust)
  darkice-merger:
    image: DOCKER_IMAGE_NAME/dendrum-server:latest # <-- TODO: Replace with your image name
    platform: linux/amd64
    restart: unless-stopped
    volumes:
      # Needs access to the same data volume
      - dendrum-server:/data
      # Needs access to the same config file
      - ./processing_config.toml:/app/processing_config.toml:ro
    environment:
      - OPENAI_TOKEN=${OPENAI_TOKEN}
      - SERVICE_MODE=merger
      - RUST_LOG=info
    depends_on:
      - dendrum-server

  # The web frontend (Nuxt)
  darkfront-frontend:
    image: DOCKER_IMAGE_NAME/dendrum-front:latest # <-- TODO: Replace with your image name
    platform: linux/amd64
    restart: unless-stopped
    ports:
      - "3000:3000"
    environment:
      - PORT=3000
      - NODE_ENV=production
    depends_on:
      - dendrum-server
```

# Define the named volume for persistent data
```
volumes:
  darkice_data:
```

2. processing_config.toml
This file configures the database paths and LLM providers. It must use the container paths (e.g., /data/...).

```Ini, TOML

[defaults.system]
# This path points *inside* the Docker volume
db_path = "/data/dendrum-server.db"

[defaults.planner_llm]
# Provider can be "Ollama" or "OpenAI"
provider = "Ollama"
model = "mistral"

[defaults.generator_llm]
# Provider can be "Ollama" or "OpenAI"
provider = "Ollama"
model = "mistral"
```

3. .env
This file stores your secret keys.

```bash
# Your OpenAI API key
OPENAI_TOKEN=sk-YourSecretKeyGoesHere
```

Step 2: Create Data Directories
You must manually create the local database directory structure to avoid file permission issues and ensure the database's WAL file is correctly initialized.

```Bash
# This creates the directories inside the Docker-managed volume
# You only need to run this once
docker-compose run --rm dendrum-server mkdir -p /data/dbs /data/pq
```

Step 3: Run the Services
Now, start the entire stack.

```Bash
docker-compose up --build -d
--build: Builds your images (you can skip this after the first run).

-d: Runs the containers in detached (background) mode.
```

Step 4: Access Your Application
Web Interface: Open your browser to http://localhost:3000
Backend API: The API is accessible at http://localhost:8001

⚙️ Configuration
LLM Provider
You can change your LLM provider by editing the processing_config.toml file and restarting the services.

To use OpenAI:

```Ini, TOML

[defaults.planner_llm]
provider = "OpenAI"
model = "gpt-4-turbo"

[defaults.generator_llm]
provider = "OpenAI"
model = "gpt-4-turbo"
```

To use a local Ollama model:

```Ini, TOML

[defaults.planner_llm]
provider = "Ollama"
model = "llama3"

[defaults.generator_llm]
provider = "Ollama"
model = "llama3"
```

(Note: For Ollama to work from inside a Docker container, you may need to set the OLLAMA_HOST environment variable in your docker-compose.yml to point to your host machine's IP, e.g., http://host.docker.internal:11434)

Persistent Data
Your DuckDB database, Parquet files, and WAL file are all stored in the darkice_data Docker volume. This means your data is safe even if you stop and remove the containers.

To "reset" your database, you can stop the services and remove the volume:

```Bash
docker-compose down
docker volume rm my-darkice-server_darkice_data
```