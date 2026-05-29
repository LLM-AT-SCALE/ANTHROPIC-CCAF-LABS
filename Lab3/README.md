# AI Medical Report Analyzer — Colab Notebook

A self-contained Google Colab notebook that turns a medical-report PDF into a
**validated structured JSON document** plus a **human-readable summary**, using
the Anthropic SDK's **forced tool use** plus a **typed Pydantic validation
retry loop** (max 3 attempts). The notebook writes every module
(`schema.py`, `extractor.py`, `validator.py`, `retry_handler.py`, `logger.py`,
`main.py`, `pdf_reader.py`) on the fly with `%%writefile`, uploads your PDF,
and runs `!python main.py` to print the structured JSON and summary inline.

## Architecture

```
  PDF -> pdfplumber -> text
            |
            v
    +----------------------------------+
    |   retry loop (max 3 attempts)    |
    |                                  |
    |   Claude  ->  tool:              |
    |     extract_medical_report       |
    |                                  |
    |   Pydantic validator             |
    |     patient_name : str           |
    |     age          : 0 < int < 120 |
    |     diagnosis    : List[str]     |
    |     + 13 optional fields         |
    |                                  |
    |   on failure -> tag pattern,     |
    |              -> feed back to LLM |
    +----------------------------------+
            |
            v
    Structured JSON  +  Human summary
    (printed inline in the cell output)
```

Forcing the tool guarantees a JSON-shaped response. Pydantic catches the
remaining failures (wrong type, out-of-range age, missing list). The loop
labels each failure with one of `invalid_age`, `missing_diagnosis`,
`unknown_validation_failure`, `missing_tool_output` and feeds the typed
signal back to the model for the next attempt.

## How to run

1. **Open the notebook in Google Colab.** Upload
   `ai_medical_report_analyzer_colab.ipynb` to Colab, or open it directly
   from GitHub / Drive.
2. **Get an Anthropic API key.** Visit
   <https://console.anthropic.com/settings/keys>, create a new key (it
   starts with `sk-ant-`), and keep it ready.
3. **Run the install cell** (cell 1). It installs `anthropic`, `pydantic`,
   `pymupdf`, `pdfplumber` into the Colab runtime.
4. **Run the `%%writefile` cells** (cells 2 through 11). Each writes one
   Python module of the analyzer to the working directory.
5. **Upload your PDF** (cell 12). The `files.upload()` widget appears;
   pick your medical report PDF. Then run **cell 13** — the API-key cell
   — which prompts you with `getpass.getpass()` for your key. The key
   is stored in the parent kernel's `os.environ` only; nothing is written
   to disk.
6. **Run `!python main.py`** (cell 14). The retry loop runs, prints the
   per-attempt trace, then prints the full structured JSON and the
   human summary inline.

> **Important — cell order matters.** Cell 13 (API-key) **must** run before
> cell 14 (`!python main.py`). The subprocess that `main.py` runs inside
> cannot read stdin from the Colab UI, so the API key must already be in
> the parent kernel's `os.environ` before the subprocess starts.

## Customization

| Want to…                              | Do this                                                                                                                                                |
|---------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| Add a new field (e.g. `insurance_id`) | Edit cell 4 (`schema.py`) — add the field to `MedicalReport` with a type and default. Then edit cell 6 (`extractor.py`) — add the matching key to the `properties` dict in `extract_tool["input_schema"]`. Re-run both cells. No other change needed. |
| Swap the model                        | In cell 8 (`retry_handler.py`), change the `model=` argument in `client.messages.create(...)`. Options: `claude-opus-4-20250514`, `claude-sonnet-4-6`, `claude-haiku-4-5-20251001`. |
| Change the retry count                | In cell 8 (`retry_handler.py`), change the `MAX_RETRIES = 3` constant. Higher = more chances for the model to self-correct, more tokens consumed.       |
| Tighten the age constraint            | In cell 4 (`schema.py`), edit `Field(gt=0, lt=120)` on `age` to your desired bounds. The validator will reject anything outside automatically.          |
| Re-extract a different PDF            | Re-run cell 12 (upload) with a new PDF, then re-run cell 14. Schema and retry stack are reused — no rebuild needed.                                     |
