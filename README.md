# 📥 Email → Notion Sync

Pull a OneDrive-hosted `Jobs.xlsx`, classify new rows with an LLM, and sync structured results to Notion—automatically, safely, and fast.

---

## Program features

1) Clear, consistent labels
- The LLM assigns stage, priority, next action, and an importance score with timestamps. Outputs stay within allowed values, so you don’t get surprises.
- Example: stage set to `interview_scheduled`, priority updated to `high`, next action determined as `schedule`, importance score calculated as `0.82`, summary generated as `"Confirm 2pm interview with Acme"`.

2) Notion stays in sync
- Missing properties are created for you, pages are updated or created using saved page ids, and duplicate body text is avoided to keep timelines readable.

3) Protect work in progress
- Local edits stay in control during merge, NEW rows are protected unless you force-refresh, and terminal stages cannot be rolled back.

4) Easy to review in Excel
- Excel artifacts are cleaned, table styling is preserved when possible, and the sheet remains easy to scan by humans.

---

## 🚀 Quickstart
1) Install deps: `pip install -r requirements.txt`
2) Set env vars (or put them in `.env`): `OPENAI_API_KEY`, optional `OPENAI_MODEL`, `NOTION_TOKEN`, `NOTION_DATABASE_ID`
3) Have access to the OneDrive `Jobs.xlsx` (default path is in [main.py](main.py))
4) Run it all:
	 ```bash
	 python main.py
	 ```

### Common variants
- Force refresh (ignore local NEW rows):
	```bash
	python -c "from main import run_full; run_full(force_refresh=True)"
	```
- Point to another OneDrive path:
	```bash
	python -c "from main import run_full; run_full(onedrive_path='PATH/TO/Jobs.xlsx')"
	```
- Work only on the local copy:
	```bash
	python -c "from local_copy_manager import copy_and_merge_to_local; copy_and_merge_to_local('/path/to/Jobs.xlsx')"
	```

After processing, copy the local `Jobs.xlsx` back to OneDrive via Finder or `cp` if you need to manually sync upstream.

---

## 🛠️ How it works
1. [main.py](main.py) loads configuration and calls `run_full`.
2. [local_copy_manager.py](local_copy_manager.py) copies from OneDrive and smart-merges with any local file (local `message_id` rows stay authoritative unless `force_refresh=True`).
3. [LLM.py](LLM.py) selects blank/NEW rows, cleans bodies, builds prompts, calls the OpenAI Responses API, validates enums/ranges, stamps timestamps, and clears errors.
4. [notion_sync/runner.py](notion_sync/runner.py) walks DONE/ERROR rows, ensures Notion property types, and creates/updates pages via [notion_sync/notion_client.py](notion_sync/notion_client.py).
5. Forward-only stage logic and duplicate-body suppression live in [notion_sync/idempotency.py](notion_sync/idempotency.py) and [notion_sync/page_template.py](notion_sync/page_template.py).
6. The updated dataframe is written back to the local Excel file through [notion_sync/excel_io.py](notion_sync/excel_io.py).

![Screenshot placeholder: Notion sync](docs/notion_properties.png)
![Screenshot placeholder: end-to-end flow](docs/flow.png)

---

## 🧭 File map
- Orchestration: [main.py](main.py)
- Copy/merge: [local_copy_manager.py](local_copy_manager.py)
- LLM prompt + schema: [LLM.py](LLM.py)
- Notion sync: [notion_sync/runner.py](notion_sync/runner.py) plus helpers in [notion_sync](notion_sync)
- Docs: [docs/PRODUCT_FEATURES.md](docs/PRODUCT_FEATURES.md), [docs/LOCAL_COPY_WORKFLOW.md](docs/LOCAL_COPY_WORKFLOW.md)

![Screenshot placeholder: LLM results](docs/notion_example.png)
