# SocialPilot AI Backend

The backend provides the API and application logic for SocialPilot AI's project-to-content workflow. It imports a public GitHub repository or pasted Markdown, analyzes its contents with a language model, generates LinkedIn drafts and case studies, checks generated claims against source evidence, persists project data in SQLite, and exports case studies.

This directory contains a deliberately small FastAPI application. It currently does **not** implement user accounts, authentication, per-user project ownership, a social network, or blockchain payments.

## Capabilities

- Import a public GitHub repository or pasted Markdown notes.
- Analyze imported sources into a structured project profile.
- Chunk and retrieve source text with BM25; optionally use Ollama embeddings with hybrid ranking.
- Generate LinkedIn drafts for technical lesson, engineering decision, and personal reflection angles.
- Generate structured case studies.
- Attach evidence references and statuses to claims (`supported`, `needs_review`, or `unsupported`).
- Persist projects, sources, generated artifacts, and edited artifacts in SQLite.
- Export a case study to Markdown, or to PDF using Typst.
- Expose an OpenAPI/Swagger interface through FastAPI.

## Architecture

```text
api/main.py
    HTTP routes, request models, error-to-status mapping
        |
core/pipeline.py
    coordinates import, analysis, generation, and saving
        |
    +--- core/ingestion.py       GitHub and pasted Markdown
    +--- core/rag/               chunking and retrieval
    +--- core/analysis.py        structured project analysis
    +--- core/generation.py      draft and case-study generation
    +--- core/evidence.py        claim evidence checks
    +--- core/llm.py             Ollama / OpenAI-compatible provider
    +--- core/storage.py         SQLite persistence
    +--- core/export.py          Markdown / Typst PDF exports
```

## Requirements

- Python 3.10 or newer.
- A language model service:
  - Ollama, or
  - an OpenAI-compatible chat completions endpoint.
- Typst is optional; only PDF export requires it.

## Local setup

Run these commands in PowerShell from the repository root:

```powershell
Set-Location Backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
Copy-Item .env.example .env
```

### Ollama (default)

The default configuration uses the Ollama model `llama3.1:8b`. Install Ollama, ensure its service is running, and pull the model:

```powershell
ollama pull llama3.1:8b
```

### OpenAI-compatible provider

Edit `Backend/.env` and configure:

```dotenv
LLM_PROVIDER=openai_compatible
LLM_MODEL=your-model-name
OPENAI_BASE_URL=https://api.openai.com/v1
OPENAI_API_KEY=your-api-key
```

The selected model/provider must return structured JSON compatible with the backend's Pydantic schemas.

### Start the API

From the `Backend` directory with the virtual environment active:

```powershell
uvicorn api.main:app --reload --host 127.0.0.1 --port 8000
```

Useful local URLs:

- Health: `http://localhost:8000/health`
- Swagger UI: `http://localhost:8000/docs`
- OpenAPI JSON: `http://localhost:8000/openapi.json`

## Configuration

Settings are read from `.env` in the backend working directory. Start the server from `Backend/` so the intended `.env` and relative database path are used.

| Variable | Default | Purpose |
| --- | --- | --- |
| `LLM_PROVIDER` | `ollama` | `ollama` or `openai_compatible`. |
| `LLM_MODEL` | `llama3.1:8b` | Model identifier sent to the provider. |
| `LLM_TIMEOUT_S` | `120` | Model request timeout. |
| `OLLAMA_HOST` | `http://localhost:11434` | Ollama service URL. |
| `OPENAI_BASE_URL` | `https://api.openai.com/v1` | OpenAI-compatible API base URL. |
| `OPENAI_API_KEY` | empty | Hosted provider credential. Keep it private. |
| `USE_EMBEDDINGS` | `0` | Set to `1` to include dense embedding retrieval. |
| `EMBED_MODEL` | `nomic-embed-text` | Ollama embedding model when dense retrieval is enabled. |
| `GITHUB_TOKEN` | empty | Optional GitHub API token for higher rate limits. |
| `DB_PATH` | `data/socialpilot.db` | SQLite database location; relative paths resolve from the backend working directory. |

See [`.env.example`](./.env.example) for the complete configuration template.

## API reference

All endpoint schemas and interactive examples are available at `/docs`.

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/health` | Return API health status. |
| `GET` | `/projects` | List persisted projects. |
| `POST` | `/projects/import` | Import a repository or pasted Markdown. |
| `GET` | `/projects/{pid}` | Return project data, source documents, profile, and artifacts. |
| `POST` | `/projects/{pid}/analyze` | Analyze the project's sources and save the profile. |
| `POST` | `/projects/{pid}/linkedin/{angle}` | Generate a LinkedIn draft. |
| `POST` | `/projects/{pid}/case-study` | Generate a case study. |
| `PUT` | `/projects/{pid}/artifacts/{artifact_type}` | Save an edited artifact version. |
| `POST` | `/export/markdown` | Render a case study as Markdown. |
| `POST` | `/export/pdf` | Compile a case study to PDF using Typst. |

### Import

Send one of `repo_url` or `pasted_readme` in the JSON body:

```json
{
  "repo_url": "https://github.com/owner/repository"
}
```

or:

```json
{
  "pasted_readme": "# Project notes\n\nDescribe the project and its implementation."
}
```

The response contains the created `project_id`. For generation requests, send an optional user context:

```json
{
  "user_context": "I implemented the retrieval layer and learned how to evaluate search quality."
}
```

Supported LinkedIn angle path values are `technical_lesson`, `engineering_decision`, and `personal_reflection`.

### Error responses

The API maps known errors to HTTP status codes. Invalid or unusable imports generally return `422`, model-provider failures `502`, missing projects `404`, and unavailable PDF export `503`. Unsupported LinkedIn angles return `400`.

## Ingestion and retrieval behavior

The GitHub importer:

- accepts public `github.com` repository URLs;
- reads repository metadata and file contents through GitHub and raw-content endpoints;
- considers selected documentation, source, and configuration file types;
- excludes common dependency/build directories and secret-like filenames;
- applies configured limits in `config.py`: at most 25 files, 60,000 bytes per file, 400,000 total bytes, and a 15-second HTTP timeout.

These filters reduce the imported surface but cannot guarantee that every sensitive file or value is excluded. Review repository contents and avoid sending confidential source material to a model provider without authorization.

Pasted content is truncated to the per-file ingestion limit. Local directories are not exposed as an HTTP import method.

The RAG implementation creates line-aware chunks, runs BM25 by default, and can combine BM25 and Ollama embeddings using Reciprocal Rank Fusion when `USE_EMBEDDINGS=1`. Evidence checks are heuristic and should be reviewed by a person.

## Persistence and generated artifacts

SQLite stores project metadata, source documents, project profiles, and artifacts. The default database path is `Backend/data/socialpilot.db` when Uvicorn is launched from this directory.

Generated and edited artifact versions are stored separately. Updating an artifact as an edit preserves the generated version; generating a replacement updates the generated content and clears an existing edit for that artifact.

The database contains imported source content. Back it up and restrict file access appropriately; do not commit production or private project databases.

## Tests

From `Backend/` with the virtual environment activated:

```powershell
pytest -q
```

The checked-in test suite covers ingestion, RAG, evidence validation, storage, and export.

## Security and deployment notes

- The current API has no authentication or project ownership enforcement. It is suitable for local development, not direct exposure to an untrusted network.
- Keep `.env`, LLM credentials, and GitHub tokens out of source control.
- Model prompts can include retrieved source text. Check the chosen provider's retention and privacy policies before processing sensitive projects.
- Use a trusted HTTPS reverse proxy and access controls for any deployment beyond localhost.
- The PDF exporter invokes the `typst` executable. Install it only from a trusted source and keep it updated.

## Troubleshooting

### Ollama connection or model errors

Verify Ollama is running, `OLLAMA_HOST` is reachable, and the model named in `LLM_MODEL` has been pulled.

### GitHub rate limit

Set `GITHUB_TOKEN` in `.env` and restart the API, or import the README as pasted Markdown.

### No usable repository files

The importer accepts only selected text/source formats and excludes generated directories. Try a repository with supported files or paste its project documentation.

### PDF export failure

Confirm `typst` is installed and available on the API process's `PATH`. Markdown export does not require Typst.

## License

The repository is licensed under Apache License 2.0. See the root [`LICENSE`](../LICENSE).
