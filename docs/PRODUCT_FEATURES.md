# Email Ingestion Product Brief

## Product features
- Local-first processing flow that copies OneDrive Jobs.xlsx into a working copy, preserves in-progress rows (`llm_status` blank/NEW), and smart-merges by `message_id` to avoid overwriting local updates.
- Automated LLM triage that classifies each new email into `stage`, `priority`, `next_action`, and `importance_score`, returns an action-oriented `summary`, and normalizes company naming.
- Safety guardrails on LLM output: fixed enums for stage/priority/next_action, bounded `importance_score` in [0,1], rejection of invalid `next_action`, and structured JSON contract enforced via pydantic schema.
- Notion CRM sync that maps DataFrame columns to Notion properties, auto-creates missing properties, and keeps a `notion_page_id` reference for idempotent updates.
- Stage progression protection when updating existing Notion pages (prevents regressions after terminal states) and duplicate-body suppression to avoid noisy appends.
- Body sanitization for Excel-encoded line breaks and preservation of existing Excel table formatting when writing results back.
- Schema tooling that casts key text columns to nullable strings, normalizes `importance_score` to float, and supports JSON Schema/SQL DDL generation for downstream storage (per prior scripts).

## How does it work
- On start, the system loads environment config, copies the OneDrive spreadsheet into the project directory as a working copy, and performs a smart merge where local rows take precedence when `message_id` matches.
- The working DataFrame is lightly typed (string/float casts) and queued rows are determined by `llm_status` being blank or NEW.
- For each pending row, the prompt builder cleans body text, assembles metadata, and calls the OpenAI Responses API; structured output is validated against enums and range checks before being written back with timestamps and cleared errors.
- After LLM completion, rows marked DONE or ERROR flow into the Notion sync layer: properties are mapped, the Notion database schema is ensured, and each row is either created or updated with thread-aware lookups by conversation/message id.
- Updates to existing Notion pages respect forward-only stage rules and mark a `Status Updated` flag; body content is appended only if not already present to reduce duplication.
- Final results are persisted into the working Excel file, maintaining table styles when possible and falling back to plain writes if needed.

## Technical prerequisites
- Python environment with project dependencies from requirements, including pandas, openpyxl, requests, python-dotenv, pydantic, and openai client libraries.
- Environment variables: `OPENAI_API_KEY` (and optional `OPENAI_MODEL`) for LLM calls; `NOTION_TOKEN` and `NOTION_DATABASE_ID` for Notion sync; optional `.env` file colocated with LLM module for key loading.
- Access to the OneDrive source file at `/Users/cm/Library/CloudStorage/OneDrive-UCIrvine/Jobs.xlsx` (or configured path) and permission to create a local working copy `Jobs.xlsx` in the project directory.
- Stable network connectivity to reach OpenAI and Notion APIs; macOS filesystem access to read/write Excel and preserve table metadata.
