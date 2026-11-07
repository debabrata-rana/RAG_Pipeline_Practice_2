# RAG Pipeline Practice

A small, notebook-based retrieval-augmented generation (RAG) pipeline for asking
questions about the India travel guide PDF in `Data/Travel_guide_for_India.pdf`.
It uses OpenAI embeddings, Qdrant for vector search, and Anthropic Claude to
generate an answer from retrieved text.

## How it works

1. `loading_vectors.ipynb` extracts text from each PDF in `Data/` and saves the
   documents to `data.json`.
2. The notebook generates an OpenAI `text-embedding-3-small` vector for each
   document and stores the vectors and text payloads in a local Qdrant collection
   named `Travel_guide_for_India`.
3. `running_rag_pipeline.ipynb` embeds a user question, searches Qdrant for
   matching text, and sends the question and retrieved context to Claude.

## Requirements

- Python and Jupyter Notebook (or JupyterLab)
- Docker Desktop or Docker Engine
- An OpenAI API key for embeddings
- An Anthropic API key and a Claude model available to your account

Install the Python packages in the environment used by your notebook kernel:

```bash
pip install anthropic openai pypdf python-dotenv qdrant-client
```

## Configuration

Create a `.env` file in the project root:

```dotenv
OPENAI_API_KEY=your-openai-api-key
ANTHROPIC_API_KEY=your-anthropic-api-key
CLAUDE_MODEL=your-available-claude-model
```

The notebooks load these values with `python-dotenv`. Keep `.env` private; do
not commit API keys or notebook output containing credentials.

## Run the pipeline

1. Start Qdrant locally. The notebooks connect to `http://localhost:6333`.

   ```bash
   docker pull qdrant/qdrant
   docker run --rm -p 6333:6333 qdrant/qdrant
   ```

   Leave this command running while using the notebooks.

2. Open `loading_vectors.ipynb` from the project root and run its cells in order.
   It reads PDFs from `Data/`, writes `data.json`, creates embeddings, and loads
   them into Qdrant.
3. Open `running_rag_pipeline.ipynb` and run its cells in order. Edit
   `user_query` in the final cell to ask a question about the guide.

## Current behavior and notes

- The ingestion notebook creates the Qdrant collection and uploads points only
  when the collection does not already exist. To reload changed PDFs, remove the
  existing `Travel_guide_for_India` collection in Qdrant before running
  ingestion again.
- The query notebook requests two matching points but currently uses only the
  first point's text as context for Claude.
- The Qdrant container command above does not mount persistent storage. If the
  container is removed, its collection is lost and ingestion must be run again.
- `data.json` contains extracted PDF text and, after ingestion, embedding
  vectors. Treat it as generated data and avoid publishing it if your documents
  are private.
