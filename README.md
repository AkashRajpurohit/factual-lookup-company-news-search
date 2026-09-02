# Factual Lookup: Company News Search

Harness, vendor runners, and judges for the
[Company News Search Benchmark](https://openbenchmarks.com/company-news).

Published by **[OpenBenchmarks Labs](https://openbenchmarks.com)**.

This repo is **open code only**. It does not include run dumps, raw vendor
HTTP, or leaderboard snapshots.

**Datasets.** The live board is scored on a locked **300-question** private
set. A **100-question** public set (question + ground truth; source URLs
omitted) is on Hugging Face as
[`openbenchmarks/OB-News-Websearch`](https://huggingface.co/datasets/openbenchmarks/OB-News-Websearch).
Use the public rows to inspect the format and run this harness; scores on
those rows are not comparable to the board.

Live board: https://openbenchmarks.com/company-news · hub: https://openbenchmarks.com/web-search

## What is here

| path | purpose |
|---|---|
| `scripts/run_websearch_benchmark.py` | Vendor sweep orchestrator |
| `scripts/websearch/run_official_news_probe.py` | HTTP adapters, redaction, resume/attach |
| `scripts/websearch/rejudge_official_extract_and_gold.py` | Terra extract from stored hits |
| `scripts/websearch/rejudge_official_gold_opus.py` | Opus accuracy judge (Bedrock converse) |
| `scripts/websearch/rejudge_official_ar_opus.py` | Opus AR@K judge on snippets |
| `scripts/websearch/score_official_search_metrics.py` | Recompute accuracy / AR@K / latency |
| `scripts/websearch/bedrock_judge.py` | Bedrock converse helper |

Each question is sent **as-is**: one request, max 10 results, title + snippet
only, no query rewrite and no page fetch. Extracts use `gpt-5.6-terra`
(medium reasoning). Accuracy and AR@K are judged by `claude-opus-5` on
Amazon Bedrock.

## Public endpoints

These are the general web-search arms on the live factual-lookup board:

| CLI id | Call |
|---|---|
| `tinyfish` | GET api.search.tinyfish.ai |
| `parallel_fast` | POST /v1/search mode=fast |
| `parallel_turbo` | POST /v1/search mode=turbo |
| `serp` | GET google-search74.p.rapidapi.com limit=10 |
| `perplexity_low` | POST /search search_context_size=low |
| `linkup_fast` | POST /v1/search depth=fast |
| `firecrawl` | POST /v2/search |
| `brave_llm` | POST /res/v1/llm/context |
| `you` | POST /v1/search |
| `parallel_basic` | POST /v1/search mode=basic |
| `brave` | GET /res/v1/web/search |
| `linkup_standard` | POST /v1/search depth=standard |
| `you_highlights` | POST /v1/search extraction_mode=highlights |
| `exa_fast` | POST /search type=fast |
| `exa_instant` | POST /search type=instant |
| `tavily_ultrafast` | POST /search search_depth=ultra-fast |

`--backend` / `--endpoints` default to this list. TinyFish Search is free
with a 30 req/min cap.

The live page also bands dedicated news indexes (Seltz news, Autobound,
PredictLeads, Datahyena) on the same 300 questions. Those adapters are not
in this runner; this repo covers the web-search band only.

## Run

```bash
python3 -m venv .venv && source .venv/bin/activate
python3 -m pip install -r requirements.txt
cp .env.example .env

PYTHONPATH=scripts python scripts/run_websearch_benchmark.py --limit 1 --endpoints tinyfish
```

A full live sweep spends vendor credits. After a sweep, re-extract and re-judge:

```bash
PYTHONPATH=scripts python scripts/websearch/rejudge_official_extract_and_gold.py --run --apply
PYTHONPATH=scripts python scripts/websearch/rejudge_official_gold_opus.py --run --apply
PYTHONPATH=scripts python scripts/websearch/rejudge_official_ar_opus.py --run --apply
PYTHONPATH=scripts python scripts/websearch/score_official_search_metrics.py
```

## License

MIT.
