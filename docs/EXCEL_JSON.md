# Excel -> JSON (and optional Notion upload)

Simple utility to convert an Excel file to JSON and optionally upload rows to a Notion database.

Quick start

1. Install dependencies:

```bash
pip install -r requirements.txt
```

2. Convert Excel to JSON:

```bash
python convert_excel.py /Users/cm/Library/CloudStorage/OneDrive-UCIrvine/Jobs.xlsx -o Jobs.json --pretty
```

3. Upload rows to Notion (will create pages; ensure database schema accepts text fields):

```bash
export NOTION_TOKEN="secret_..."
export NOTION_DATABASE_ID="your-db-id"
python convert_excel.py /Users/cm/Library/CloudStorage/OneDrive-UCIrvine/Jobs.xlsx --notion
```

Notes
- The Notion uploader attempts to map each column to a `title` (first column) or `rich_text` property. If your database expects different property types, use the API or adjust the database schema.
- For multi-sheet workbooks the script will choose the first sheet when uploading to Notion; JSON output will contain a mapping of sheet -> rows.
