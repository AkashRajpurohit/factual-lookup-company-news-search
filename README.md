# Factual Lookup: Company News Search

Harness, vendor runners, and judges for the
[Company News Search Benchmark](https://openbenchmarks.com/company-news).

Published by **[OpenBenchmarks Labs](https://openbenchmarks.com)**.

This repo is **open code only**. It does not include run dumps, raw vendor
HTTP, or leaderboard snapshots. The public 100-question set (question +
ground truth answer) is on Hugging Face:
[`openbenchmarks/OB-News-Websearch`](https://huggingface.co/datasets/openbenchmarks/OB-News-Websearch).

Live board: https://openbenchmarks.com/company-news · hub: https://openbenchmarks.com/web-search

## What is here

| path | purpose |
|---|---|
| `scripts/run_websearch_benchmark.py` | Vendor sweep orchestrator |
| `scripts/websearch/run_official_news_probe.py` | HTTP adapters, redaction, resume/attach |
| `scripts/websearch/predictleads_news.py` | PredictLeads news_events adapter |
| `scripts/websearch/rejudge_official_extract_and_gold.py` | Terra extract from stored hits |
| `scripts/websearch/rejudge_official_gold_opus.py` | Opus accuracy judge (Bedrock converse) |
| `scripts/websearch/rejudge_official_ar_opus.py` | Opus AR@K judge on snippets |
| `scripts/websearch/score_official_search_metrics.py` | Recompute accuracy / AR@K / latency |
| `scripts/websearch/bedrock_judge.py` | Bedrock converse helper |

Each question is sent **as-is**: one request, max 10 results, title + snippet
only, no query rewrite and no page fetch. Extracts use `gpt-5.6-terra`
(medium reasoning). Accuracy and AR@K are judged by `claude-opus-5` on
Amazon Bedrock.

## Run

```bash
python3 -m venv .venv && source .venv/bin/activate
python3 -m pip install -r requirements.txt
cp .env.example .env

PYTHONPATH=scripts python scripts/run_websearch_benchmark.py --limit 1 --endpoints exa_instant
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
