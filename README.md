# AI Engineer Intern Assignment 2: LLM Integration & Data Pipeline

This repository contains a modular Python pipeline that ingests unstructured text from local `.txt` / `.pdf` files and URLs, cleans and chunks the text, sends each chunk to an LLM for structured extraction, and writes JSON, CSV, and plain-text summary outputs.

The implementation uses direct API calls through the OpenAI SDK. It does not use LangChain, LlamaIndex, or similar orchestration frameworks.

## Why OpenAI

I used OpenAI because its chat completions API supports strict JSON response formatting, has mature Python SDK support, and provides clear error classes for production retry handling. The code can also run in `mock` mode to generate deterministic sample outputs without an API key.

## Features

- Accepts local `.txt` and `.pdf` files plus multiple URLs in one run.
- Cleans common encoding artifacts, page boilerplate, repeated whitespace, and noisy web text.
- Chunks long documents using token-aware chunking with overlap.
- Calls an LLM directly and requests structured JSON.
- Parses JSON robustly, including recovery from fenced or malformed model output.
- Retries transient failures, rate limits, and timeouts with exponential backoff using `tenacity`.
- Logs bad inputs and failed chunks while continuing the run.
- Writes:
  - `results.json`
  - `results.csv`
  - `summary_report.txt`

## Repository Layout

```text
.
├── data/
│   └── samples/
│       └── sample_article.txt
├── outputs/
│   └── sample_run/
│       ├── results.csv
│       ├── results.json
│       └── summary_report.txt
├── src/
│   └── llm_pipeline/
│       ├── cli.py
│       ├── config.py
│       ├── ingest.py
│       ├── llm_client.py
│       ├── models.py
│       ├── pipeline.py
│       ├── preprocess.py
│       ├── reporting.py
│       └── writers.py
├── requirements.txt
└── urls.sample.txt
```

## Setup

Create and activate a virtual environment:

```bash
python -m venv .venv
.venv\Scripts\activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Set your API key for real LLM calls:

```bash
set OPENAI_API_KEY=your_api_key_here
```

PowerShell users can run:

```powershell
$env:OPENAI_API_KEY="your_api_key_here"
```

## Run With Mock LLM

Mock mode is useful for local testing and for the included sample outputs:

```bash
python -m src.llm_pipeline.cli --files data/samples/sample_article.txt --urls-file urls.sample.txt --output-dir outputs/sample_run --mock
```

## Run With OpenAI

```bash
python -m src.llm_pipeline.cli --files data/samples/sample_article.txt path/to/document.pdf --urls https://example.com/article --output-dir outputs/run_openai
```

Optional model override:

```bash
python -m src.llm_pipeline.cli --model gpt-4o-mini --files data/samples/sample_article.txt --urls https://example.com --output-dir outputs/run_openai
```

## CLI Options

```text
--files        One or more .txt or .pdf files.
--urls         One or more URLs.
--urls-file    Text file containing one URL per line.
--output-dir   Directory for JSON, CSV, report, and logs.
--model        OpenAI model name. Defaults to gpt-4o-mini.
--mock         Use deterministic mock extraction instead of calling an API.
--max-tokens   Approximate maximum tokens per chunk.
```

## Design Decisions

- The pipeline treats every input as a `Document`, then splits it into `Chunk` objects. This makes local files and URLs flow through the same downstream logic.
- Preprocessing is deliberately simple and auditable: normalize text, remove obvious web boilerplate, collapse whitespace, and chunk with overlap.
- The LLM client owns retry logic and JSON parsing. Transient API errors are retried; malformed JSON is repaired when possible and logged when not.
- Per-input and per-chunk errors are logged explicitly. The pipeline continues after a failed URL, unreadable PDF, timeout, or malformed chunk response.
- Outputs use plain JSON and CSV so downstream systems can consume them without custom tooling.

## Tested Inputs

The included sample run used:

- `data/samples/sample_article.txt`
- URLs listed in `urls.sample.txt`

The committed sample outputs were generated in `--mock` mode so reviewers can inspect the output format without needing an API key.

## Known Limitations

- PDF extraction depends on embedded text. Scanned image-only PDFs require OCR, which is outside this assignment scope.
- Boilerplate removal uses practical heuristics, not site-specific extraction rules.
- Entity extraction quality depends on the selected LLM.
- Mock mode is deterministic and useful for tests, but it is not a substitute for real LLM quality evaluation.

## Suggested Video Walkthrough

For the required short video:

1. Show the repository layout and `README.md`.
2. Run the mock command to demonstrate end-to-end output generation.
3. Open `results.json`, `results.csv`, and `summary_report.txt`.
4. Briefly show the retry and malformed JSON handling in `src/llm_pipeline/llm_client.py`.
5. Explain that real OpenAI calls require `OPENAI_API_KEY` and use direct SDK calls.
