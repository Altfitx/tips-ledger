# AltFitX tips ledger

Append-only public record of prediction tips for [AltFitX Predictions](https://api.predictions.altfitx.com/record/).

This repository exists so anyone can verify that tips were **published before kickoff** and were **not silently edited** after the fact.

Organization: [github.com/Altfitx](https://github.com/Altfitx)

## Trust rules

1. **Tips are append-only.** New tips are added as new lines / new daily files. Do not rewrite history.
2. **Force-push to `main` is forbidden** (enable branch protection). A force-push would invalidate trust.
3. **A tip is only “proven” if** `published_at` (and/or the git commit that introduced it) is **before** `kickoff_at`.
4. **Settlements are separate append-only records** that reference a tip `id` + `content_hash`. They do not change the original tip payload.
5. Historical rows imported after the match may be marked `"backfill": true` — those are **not** pre-match proofs. The honest ledger starts from the day continuous export was enabled.

## Layout (target)

```text
tips/YYYY-MM-DD.jsonl          # one tip per line at publish time
settlements/YYYY-MM-DD.jsonl   # one settlement per line after the match
snapshots/daily/YYYY-MM-DD.json # optional end-of-day ROI summary
```

### Tip line (example)

```json
{
  "id": 479,
  "published_at": "2026-09-08T10:22:00Z",
  "kickoff_at": "2026-09-15T19:00:00Z",
  "league": "LLA",
  "home": "Elche",
  "away": "Real Madrid",
  "market": "ou25",
  "selection": "over",
  "book_odds": "2.550",
  "model_prob": 0.553,
  "edge": 0.161,
  "stake_units": "1.00",
  "model_run": "dc-20260908-085057",
  "content_hash": "sha256:…"
}
```

`content_hash` covers the immutable tip fields (not status / PnL).

### Settlement line (example)

```json
{
  "tip_id": 479,
  "content_hash": "sha256:…",
  "result": "won",
  "pnl_units": "1.550",
  "score": "4-1",
  "settled_at": "2026-09-15T21:05:00Z"
}
```

## How to verify

1. Clone this repo.
2. For each tip: check `published_at < kickoff_at` (and/or that the git commit that introduced the tip predates kickoff).
3. Recompute `content_hash` and match the stored hash.
4. Ensure tip `id`s are unique; settlements only append.
5. Sum `pnl_units` from settlements and compare with the public record:
   - https://api.predictions.altfitx.com/record/
   - https://api.predictions.altfitx.com/api/performance/

## Disclaimer

Entertainment / research only. Not financial advice. Past ROI does not guarantee future results.

## Status

Bootstrap commit. Automated export from production will populate `tips/` and `settlements/` next.

## Automation

Production VPS runs `export_ledger` + `cron_ledger.sh` hourly (append-only JSONL commits).
