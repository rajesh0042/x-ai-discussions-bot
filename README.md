# X AI Discussions Bot

A daily bot that searches X for notable AI discussions, prioritizes influential
authors and highly engaged posts, summarizes the results with a Microsoft
Foundry agent, and writes a Markdown report.

## What it does

- Runs each configured X recent-search query with up to 100 results.
- Excludes authors with fewer than 50,000 followers.
- Scores posts from 0–100 using followers (30%), reposts (30%), likes (20%),
  and replies (20%).
- Sends the ten highest-scoring posts to a hosted Microsoft Foundry agent for
  concise summaries and key points.
- Writes dated reports to `reports/`.
- Runs at 09:00 UTC every day through GitHub Actions, with manual dispatch
  available.

## Prerequisites

- Python 3.12 or later
- An X API bearer token with recent-search access
- A Microsoft Foundry project and hosted agent

## Setup

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
```

Fill in `.env`:

| Variable | Description |
| --- | --- |
| `X_API_BEARER_TOKEN` | X API v2 bearer token |
| `FOUNDRY_API_KEY` | Microsoft Foundry project API key |
| `FOUNDRY_AGENT_ID` | Name/ID of the hosted Foundry agent |
| `FOUNDRY_PROJECT_ENDPOINT` | Foundry project endpoint, such as `https://RESOURCE.services.ai.azure.com/api/projects/PROJECT` |

The default queries, thresholds, score weights, and metric caps are in
`config.json`. Metric caps define the value at which a metric receives its
full weighted score. Scores are calculated as:

```text
sum(min(metric / cap, 1) * weight * 100)
```

## Run

```bash
python main.py
```

Use a different configuration or output location when needed:

```bash
python main.py --config path/to/config.json --output-dir path/to/reports
```

The command exits without creating a report if no qualifying posts are found.
If one Foundry summary fails, that post receives a short extractive fallback so
the rest of the report can still be generated.

## GitHub Actions

Add these repository secrets:

- `X_API_BEARER_TOKEN`
- `FOUNDRY_API_KEY`
- `FOUNDRY_AGENT_ID`
- `FOUNDRY_PROJECT_ENDPOINT`

The workflow in `.github/workflows/daily-scan.yml` runs daily and commits a new
report when one is generated. Repository Actions must have read/write workflow
permissions.

## Testing

```bash
python -m unittest discover -s tests -v
```
